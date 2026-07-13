# Networking

The networking subsystem owns the active transport, turns a byte stream into packet bodies, applies the protocol's XOR transformations, and moves decoded server packets into the pane event hierarchy. Client packet builders use raw byte arrays and the same small big-endian field writers.

The directional action indexes are [Client actions](../protocol/client-actions.md) and [Server actions](../protocol/server-actions.md).

## High-level operation

### Connection and Windows notification

The normal game transport uses Winsock 1.1 and an IPv4 TCP stream. Connection setup creates an `AF_INET`, `SOCK_STREAM` socket, requests a `0x4000` byte Winsock receive buffer, resolves the configured host, and connects to the configured port. Once connected, `WSAAsyncSelect` associates the socket with the main window using message `0x0402` and event mask `0x21`, which is `FD_READ | FD_CLOSE`.

The main window procedure handles the low word of `lParam` for message `0x0402`:

| Winsock event | Value | Client action |
|---|---:|---|
| `FD_READ` | `0x0001` | Queue socket work code 4, which runs the receive framer. |
| `FD_CLOSE` | `0x0020` | Close the socket and serial handle, clear connection state, and offer the reconnect UI. |

The executable also retains serial and printable-text transports. They eventually feed the same logical packet event path, but the standard Stone server path is the binary TCP path described here.

### Binary frame

Every binary TCP frame begins with `0xAA`. The two-byte size is unsigned and big-endian. It counts the body beginning at the action byte, not the marker or the size itself.

| Offset | Width | Type | Meaning |
|---:|---:|---|---|
| `0x00` | 1 | `uint8_t` | Marker, always `0xAA`. |
| `0x01` | 2 | `uint16_t`, big-endian | Wire body length. |
| `0x03` | `body_length` | bytes | Action, transformation metadata where applicable, and payload. |

The total frame length is therefore `3 + body_length`. The receive code requires a positive body length and stores at most `0x10000` body bytes. Since the wire field is 16 bits, the largest representable wire value is `0xFFFF`.

The application-facing logical packet is:

```text
[uint8 action] [payload bytes...] [zero sentinel]
```

`net_queue_send` receives the action and payload length from a packet builder, copies those bytes, and appends the zero sentinel. The sentinel is an implementation guard, not a checksum. The decoder does not validate its encrypted value. After decoding it writes another zero immediately after the returned logical bytes.

### Direction defines the action

Action values are not globally unique. A client action and a server action with the same byte can have unrelated meanings. For example, both directions support `0x0D`, but the send builders and receive handlers are separate code paths. Protocol implementations must key dispatch by `(direction, action)`, never by action alone.

The executable contains 51 fixed client action values and 59 server action values. Thirty-five values occur in both directions. A server `0x4B` packet can also ask the client to forward an embedded logical client packet, so that path is not restricted to one statically embedded client action constant.

### Receive and event routing

The TCP reader refills a `0x18000` byte cache with `recv` and then supplies one byte at a time to the framer. This preserves a partial header or body across Windows notifications and permits multiple complete frames to be drained from one receive cache.

After a frame is complete, the client decodes it when required and posts a copy as internal work code `0x0E`. The event worker converts that work item into an Event with type 9, packet pointer at offset `+0x14`, and packet size at offset `+0x18`. The event manager recursively walks the pane hierarchy and calls virtual slot `+0x40` on each eligible pane. That virtual method is the pane's server-packet handler.

Dispatch is therefore distributed rather than centralized. `MapPane` owns the largest switch, while `MainMenuPane`, `ServerSelectDialogPane`, exchange panes, bulletin panes, and other UI objects handle actions relevant to their current state. The [server action index](../protocol/server-actions.md) combines those separate handlers.

### Send path

Packet builders assemble a logical packet in a local byte array and call `net_queue_send`. The Socket object copies the bytes, appends the sentinel, and queues work code 5. The socket worker then applies the client transformation rule, allocates `body_length + 3` bytes, writes the frame header, and calls Winsock `send` once.

The send path treats `SOCKET_ERROR` as failure. It does not retry when `send` succeeds with a byte count smaller than the requested frame length. A compatible implementation should use a full-write loop even though this client does not.

During a server transfer state, `net_queue_send` drops every client action except `0x10`, the client transfer-server action.

### Packet representation and C++ class evidence

The traced send and receive paths do not instantiate a class per packet or a common `ClientPacket` or `ServerPacket` object. Client builders write directly into byte arrays. The receiver copies decoded bytes into an Event, and pane handlers switch on `packet[0]` and call field readers on later offsets.

The code does establish these surrounding C++ types through constructors, virtual tables, source assertions, and diagnostics:

| Established type | Evidence in the packet path |
|---|---|
| `Socket` | `Socket.cpp` diagnostics, a constructor-installed virtual table, transport state fields, and virtual work-item processing. |
| `Event` | Socket Event type 9 owns the packet pointer and size and frees the packet after dispatch. |
| `MainMenuPane` | Named diagnostics and virtual table `Darkages.exe:0x005177C0`; handles initial version and key negotiation. |
| `MapPane` | `MapPane.cpp`, named method diagnostics, and virtual table `Darkages.exe:0x00519280`; owns the main in-game action switch. |
| `ServerSelectDialogPane` | Named constructor and deletion diagnostics and virtual table `Darkages.exe:0x00526140`; handles server action `0x56`. |

Names such as `kServerRequestPortrait` and `kClientTransferServer` are enum-style action names embedded in diagnostics, not evidence of packet classes. The safest compatible-client model for the mapped path is a direction-tagged byte buffer plus per-action serializers and handlers.

### Big-endian serialization

The packet builders and handlers share small integer helpers:

| Width | Writer | Reader | Byte order |
|---:|---|---|---|
| 1 byte | `net_write_u8` | `net_read_u8` | Not applicable |
| 2 bytes | `net_write_u16_be` | `net_read_u16_be` | Most significant byte first |
| 4 bytes | `net_write_u32_be` | `net_read_u32_be` | Most significant byte first |

The writers place a zero byte immediately after the value. Packet builders subsequently overwrite that byte when adding the next field, or leave it as the packet sentinel at the final logical offset.

### Sequence and XOR transformation

Ordinary bodies carry a sequence byte and three XOR passes. This is an obfuscating transformation. The code does not provide authentication, integrity checking, or a cryptographic security boundary, so this documentation calls it XOR rather than encryption.

The bypass lists differ by direction:

| Direction | Bodies sent without sequence or XOR |
|---|---|
| Client to server | `0x00`, `0x48`, `0x10` |
| Server to client | `0x00`, `0x40`, `0x03` |

For an ordinary client packet, let `A` be the action, `D` be `payload || 0x00`, `K` be the repeating key of length `k`, `T` be the generated byte table, and `S` be the current 8-bit send sequence. Encoding produces:

```text
[A] [S] [encoded D]
```

The client increments `S` modulo 256 after copying it. It transforms `D` in this order:

```c
for (i = 0; i < data_length; ++i)
    q[i] = D[i] ^ K[i % k];

for (block = 0; block * k < data_length; ++block) {
    uint8_t b = (uint8_t)block;
    if (b == S)
        continue;
    for (j = 0; j < k && block * k + j < data_length; ++j)
        q[block * k + j] ^= T[b + j];
}

for (i = 0; i < data_length; ++i)
    q[i] ^= T[S + i];
```

The table lookups use the shown additive indexes. The code does not apply an explicit modulo operation or bounds check after the addition. Decoding applies the sequence-table pass, the per-block pass, and the repeating-key pass in reverse order, removes the sequence byte, writes an additional zero after the returned bytes, and returns `wire_body_length - 1` logical bytes.

The initial key is the nine ASCII bytes `NexonInc.`. Work code 10 can replace it with a negotiated key of at most nine bytes. The implementation stores four consecutive copies for its unrolled loops. Work code 11 selects one of ten functions that fill the 256-entry table. With input `x` from 0 through 255, the generated byte is:

| Function | Generated value |
|---:|---|
| 0 | `x` |
| 1 | `128 + (x even ? floor((x + 1) / 2) : -floor((x + 1) / 2))` |
| 2 | `255 - x` |
| 3 | `128 + (x even ? floor((255 - x) / 2) : -floor((255 - x) / 2))` |
| 4 | `floor(x / 16) * floor(x / 16)` |
| 5 | `(2 * x) & 0xFF` |
| 6 | `255 - ((2 * x) & 0xFF)` |
| 7 | `x <= 127 ? 255 - 2 * x : 2 * x - 256` |
| 8 | `x <= 127 ? 2 * x : 511 - 2 * x` |
| 9 | `255 - ((trunc((x - 128) / 8)^2) & 0xFF)` |

The image's initial table is function 0. Each byte value is stored as a dword whose four bytes are identical, which lets the optimized loops XOR four payload bytes at a time.

### Failure and bounds behavior

The standard receive path has these established edge behaviors:

- Outside a frame, the next consumed byte is asserted to be `0xAA`; the code does not scan ahead for a later marker.
- A body length of zero or greater than `0x10000` fails an internal assertion. The two-byte wire field cannot encode a value greater than `0xFFFF`.
- `recv == 0` closes the socket, sets it to `INVALID_SOCKET`, clears connection state, and stops reading.
- `recv == SOCKET_ERROR` clears the receive cache and returns no byte. This path does not distinguish individual `WSAGetLastError` values.
- Completed packet bytes are copied for the event worker. The Event owns the copy and frees it after pane dispatch.
- No checksum or message authentication value is validated by the mapped frame or XOR code.

## Code-level flow

### Startup and connection

1. `Darkages.exe:0x004A32F0` `net_socket_ctor` installs the Socket virtual table, initializes the transport buffers and state, and installs `NexonInc.` as the default repeating key.
2. `Darkages.exe:0x004A59F4` `net_connect_configured_server` starts Winsock 1.1, checks that version 1.1 was returned, resolves the configured host, creates and connects the TCP socket, and registers `WM_USER + 2` with `FD_READ | FD_CLOSE`.
3. `Darkages.exe:0x0045BDC0` `win_main_window_proc` receives message `0x0402`. `FD_READ` calls `net_queue_receive`; `FD_CLOSE` calls `net_handle_disconnect`.

### Receive, decode, and dispatch

```text
win_main_window_proc
  -> net_queue_receive                         work code 4
  -> net_process_work_item
      -> net_receive_frames
          -> net_read_transport_byte
              -> recv                          refill up to 0x18000 bytes
          -> net_decode_packet_body            ordinary server actions
          -> event_post_socket_bytes           work code 0x0E
              -> event_process_work_item
                  -> event_queue_socket_packet Event type 9
                      -> event_dispatch
                          -> event_dispatch_hierarchy
                              -> pane vtable +0x40
```

`net_receive_frames` maintains three important pieces of persistent state: whether a frame is open, the number of header or body bytes collected, and the current body length. It combines the first two body-header bytes as `(high << 8) | low`. Completion occurs after exactly `body_length` bytes have been placed in the packet buffer.

`event_dispatch_hierarchy` tests Event type 9 with `event_is_socket_packet` before calling pane virtual slot `+0x40`. The return value indicates whether the event was consumed. The event manager frees the packet pointer after dispatch when the Event is a socket packet.

### Build, encode, and send

```text
per-action packet builder
  -> net_write_u8 / net_write_u16_be / net_write_u32_be
  -> net_queue_send                            copy and append zero
      -> net_process_work_item                 work code 5
          -> net_send_packet_body
              -> net_encode_packet_body        ordinary client actions
              -> net_write_u8(0xAA)
              -> net_write_u16_be(body_length)
              -> send                          one call
```

`net_encode_packet_body` returns one byte more than its input size because it inserts the sequence after the action. `net_decode_packet_body` returns one byte less than its input size because it removes that sequence. Bypass actions keep their original body size.

### Representative server dispatch

`Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet` is the largest action switch. It accepts 31 action values and rejects the other values in its `0x03` through `0x4B` switch range. Important established branches include:

| Server action | Callee or effect |
|---:|---|
| `0x03` | Transfer-server processing. |
| `0x0A` | Secondary subtype dispatch within `MapPane`. |
| `0x0E` | Delete an object from map state. |
| `0x32` | Inline tile or object updates. |
| `0x4B` | `net_forward_embedded_client_packet`, which reads a big-endian embedded length and sends the embedded logical client packet. |

The complete accepted set and all other pane handlers are in the [server action index](../protocol/server-actions.md).

## Function table

| Address | Current IDA name | Prototype | Purpose | Call relationships and notes |
|---:|---|---|---|---|
| `Darkages.exe:0x00431B84` | `event_dispatch` | `int __thiscall(void *this, void *event)` | Dispatch one internal Event. | Calls `event_dispatch_hierarchy`; participates in capture and consumed-event handling. |
| `Darkages.exe:0x00431D54` | `event_dispatch_hierarchy` | `int __thiscall(void *this, void *event, void *hierarchy)` | Recursively deliver an Event to panes. | Calls pane virtual slot `+0x40` for socket Event type 9. |
| `Darkages.exe:0x00432610` | `event_is_socket_packet` | `int __thiscall(void *event)` | Test for socket Event type 9. | Reads the Event type byte at `+0x0C`. |
| `Darkages.exe:0x00432E50` | `event_post_socket_bytes` | `void __thiscall(void *this, const uint8_t *packet, int length)` | Copy decoded packet bytes to the worker queue. | Posts work code `0x0E`. |
| `Darkages.exe:0x00433110` | `event_process_work_item` | `void __thiscall(void *this, int code, void *data, int value)` | Convert worker records to internal Events. | Code `0x0E` calls `event_queue_socket_packet`. |
| `Darkages.exe:0x00433DC4` | `event_queue_socket_packet` | `void __stdcall(uint8_t *packet, uint32_t size)` | Create and queue a socket Event. | Sets Event type 9, pointer `+0x14`, and size `+0x18`. |
| `Darkages.exe:0x0045BDC0` | `win_main_window_proc` | `LRESULT __stdcall(HWND, UINT, WPARAM, LPARAM)` | Main window procedure and Winsock notification bridge. | Message `0x0402` handles `FD_READ` and `FD_CLOSE`. |
| `Darkages.exe:0x004A32F0` | `net_socket_ctor` | `void *__thiscall(void *this, void *event_sink)` | Construct and initialize the Socket object. | Installs default key and transport buffers. |
| `Darkages.exe:0x004A3560` | `net_queue_receive` | `void __thiscall(void *this)` | Queue receive processing. | Posts Socket work code 4. |
| `Darkages.exe:0x004A3570` | `net_queue_send` | `void __thiscall(void *this, const uint8_t *packet, int16_t length)` | Copy and queue a logical client packet. | Adds the zero sentinel and posts work code 5. |
| `Darkages.exe:0x004A3770` | `net_write_u32_be` | `void __cdecl(uint32_t value, uint8_t *dest)` | Write a big-endian 32-bit field. | Writes a zero byte at `dest[4]`. |
| `Darkages.exe:0x004A37D0` | `net_read_u8` | `uint32_t __cdecl(const uint8_t *src)` | Read an unsigned byte. | Used by server action handlers. |
| `Darkages.exe:0x004A3810` | `net_read_u16_be` | `uint32_t __cdecl(const uint8_t *src)` | Read a big-endian 16-bit field. | Used by framing-related and packet handlers. |
| `Darkages.exe:0x004A38B0` | `net_read_u32_be` | `uint32_t __cdecl(const uint8_t *src)` | Read a big-endian 32-bit field. | Returns the combined 32-bit value. |
| `Darkages.exe:0x004A3910` | `net_handle_disconnect` | `void __thiscall(void *this)` | Close transports and handle connection loss. | Called for `FD_CLOSE`. |
| `Darkages.exe:0x004A39C0` | `net_poll_receive` | `void __thiscall(void *this)` | Poll transport readiness outside the async window path. | Uses `select` for TCP and calls `net_receive_frames`. |
| `Darkages.exe:0x004A3A74` | `net_receive_frames` | `void __thiscall(void *this)` | Extract, decode, and post complete packets. | Calls `net_read_transport_byte`, `net_decode_packet_body`, and `event_post_socket_bytes`. |
| `Darkages.exe:0x004A44D4` | `net_decode_packet_body` | `int __stdcall(const uint8_t *src, int src_size, uint8_t *dest)` | Reverse the ordinary server-body XOR transformation. | Returns `src_size - 1`. |
| `Darkages.exe:0x004A4D34` | `net_read_transport_byte` | `int __thiscall(void *this, uint8_t *out_byte)` | Supply one cached transport byte. | Refills with `recv` or `ReadFile`. |
| `Darkages.exe:0x004A54D0` | `net_process_work_item` | `void __thiscall(void *this, int code, void *data, int value)` | Execute queued Socket work. | Codes 4, 5, 10, and 11 drive receive, send, key, and table operations. |
| `Darkages.exe:0x004A59F4` | `net_connect_configured_server` | `void __thiscall(void *this)` | Establish the configured connection. | Uses Winsock startup, resolution, `connect`, and `WSAAsyncSelect`. |
| `Darkages.exe:0x004A72B4` | `net_send_packet_body` | `void __thiscall(void *this, const uint8_t *packet, int16_t length)` | Transform, frame, and send a client packet. | Calls `send` once and frees the temporary frame. |
| `Darkages.exe:0x004A7940` | `net_write_u8` | `void __cdecl(uint32_t value, uint8_t *dest)` | Write one byte. | Writes a zero byte at `dest[1]`. |
| `Darkages.exe:0x004A7990` | `net_write_u16_be` | `void __cdecl(uint32_t value, uint8_t *dest)` | Write a big-endian 16-bit field. | Used for the frame length and packet fields. |
| `Darkages.exe:0x004A79E4` | `net_encode_packet_body` | `int __stdcall(const uint8_t *src, int src_size, uint8_t *dest)` | Apply the ordinary client-body sequence and XOR transformation. | Returns `src_size + 1`. |
| `Darkages.exe:0x004A8414` | `net_set_xor_key` | `void __stdcall(int length, const uint8_t *key)` | Replace the runtime XOR key. | Requires length at most 9 and writes four repeated copies. |
| `Darkages.exe:0x004A8590` | `net_build_xor_table` | `void __stdcall(int function_index)` | Generate the 256-entry XOR table. | Accepts function indexes 0 through 9. |
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | `int __thiscall(void *this, void *event)` | Dispatch in-game server actions in `MapPane`. | Supports 31 fixed action values. |
| `Darkages.exe:0x0046CA54` | `net_forward_embedded_client_packet` | established `MapPane` action handler | Forward the client packet embedded in server action `0x4B`. | Reads a big-endian length and calls `net_queue_send`. |
| `Darkages.exe:0x0045F780` | `ui_main_menu_handle_server_packet` | `int __thiscall(void *this, void *event)` | Handle main-menu server actions. | Supports `0x00`, `0x02`, `0x03`, and `0x0A`. |
| `Darkages.exe:0x004A2ED0` | `ui_server_select_handle_server_packet` | `int __thiscall(void *this, void *event)` | Deserialize the server table from action `0x56`. | Owns and replaces the per-server entry allocation. |
| `Darkages.exe:0x0040B3A0` | `ui_handle_server_request_portrait` | `int __thiscall(void *this, void *event)` | Handle `kServerRequestPortrait`, action `0x49`. | Calls `net_build_portrait_response`. |
| `Darkages.exe:0x0040A664` | `net_build_portrait_response` | established `__thiscall` builder | Build client action `0x4F` from a local portrait. | Called after server action `0x49`. |
| `Darkages.exe:0x004C33B2` | `recv` | Winsock import | Refill the TCP receive cache. | Called with a maximum length of `0x18000`. |
| `Darkages.exe:0x004C33D6` | `send` | Winsock import | Send a complete allocated frame. | The caller does not handle a short successful write. |
| `Darkages.exe:0x004C33D0` | `WSAAsyncSelect` | Winsock import | Register socket notifications with the main window. | Uses message `0x0402` and mask `0x21`. |
