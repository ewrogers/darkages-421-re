# Function Map

This appendix is a cross-subsystem index of important functions, grouped by primary subsystem. Cross-subsystem functions appear under the subsystem that owns their main control flow. Detailed decisions belong in the relevant subsystem chapter. An address remains the durable identifier when a name improves later.

## Events

| Address | Current IDA name | Subsystem | Prototype | Purpose | Detailed page |
|---:|---|---|---|---|---|
| `Darkages.exe:0x00431B84` | `event_dispatch` | Events | `int __thiscall(void *this, void *event)` | Dispatch one internal Event. | [Networking](../subsystems/networking.md#receive-and-event-routing) |
| `Darkages.exe:0x00431D54` | `event_dispatch_hierarchy` | Events | `int __thiscall(void *this, void *event, void *hierarchy)` | Walk panes and call the type-specific virtual handler. | [Networking](../subsystems/networking.md#receive-and-event-routing) |
| `Darkages.exe:0x00432E50` | `event_post_socket_bytes` | Events and network | `void __thiscall(void *this, const uint8_t *packet, int length)` | Copy decoded packet bytes to work code `0x0E`. | [Networking](../subsystems/networking.md#receive-and-event-routing) |
| `Darkages.exe:0x00433110` | `event_process_work_item` | Events | `void __thiscall(void *this, int code, void *data, int value)` | Convert worker records to Event objects. | [Networking](../subsystems/networking.md#receive-and-event-routing) |
| `Darkages.exe:0x00433DC4` | `event_queue_socket_packet` | Events and network | `void __stdcall(uint8_t *packet, uint32_t size)` | Queue Event type 9 with packet ownership. | [Networking](../subsystems/networking.md#receive-and-event-routing) |

## Networking

| Address | Current IDA name | Subsystem | Prototype | Purpose | Detailed page |
|---:|---|---|---|---|---|
| `Darkages.exe:0x0040A664` | `net_c_build_portrait_response` | Network and UI | established `__thiscall` builder | Build client action `0x4F` from the local portrait. | [Networking](../subsystems/networking.md) |
| `Darkages.exe:0x0046CA54` | `net_forward_embedded_client_packet` | Network | established `MapPane` action handler | Forward the client packet embedded in server action `0x4B`. | [Client actions](../protocol/client-actions.md#server-directed-dynamic-send) |
| `Darkages.exe:0x004A32F0` | `net_socket_ctor` | Network | `void *__thiscall(void *this, void *event_sink)` | Construct and initialize the Socket object. | [Networking](../subsystems/networking.md#startup-and-connection) |
| `Darkages.exe:0x004A3570` | `net_c_queue_send` | Network | `void __thiscall(void *this, const uint8_t *packet, int16_t length)` | Copy, terminate, and queue a logical client packet. | [Networking](../subsystems/networking.md#send-path) |
| `Darkages.exe:0x004A3A74` | `net_s_receive_frames` | Network | `void __thiscall(void *this)` | Extract, decode, and post binary frames. | [Networking](../subsystems/networking.md#receive-decode-and-dispatch) |
| `Darkages.exe:0x004A44D4` | `net_s_decode_packet_body` | Network | `int __stdcall(const uint8_t *src, int src_size, uint8_t *dest)` | Reverse the ordinary server-body XOR transformation. | [Networking](../subsystems/networking.md#sequence-and-xor-transformation) |
| `Darkages.exe:0x004A4D34` | `net_read_transport_byte` | Network | `int __thiscall(void *this, uint8_t *out_byte)` | Read a cached byte, refilling with `recv` or `ReadFile`. | [Networking](../subsystems/networking.md#receive-and-event-routing) |
| `Darkages.exe:0x004A54D0` | `net_process_work_item` | Network | `void __thiscall(void *this, int code, void *data, int value)` | Execute queued Socket operations. | [Networking](../subsystems/networking.md#code-level-flow) |
| `Darkages.exe:0x004A59F4` | `net_connect_configured_server` | Network | `void __thiscall(void *this)` | Establish the configured socket and async notification. | [Networking](../subsystems/networking.md#startup-and-connection) |
| `Darkages.exe:0x004A72B4` | `net_c_send_packet_body` | Network | `void __thiscall(void *this, const uint8_t *packet, int16_t length)` | Transform, frame, and call `send`. | [Networking](../subsystems/networking.md#build-encode-and-send) |
| `Darkages.exe:0x004A79E4` | `net_c_encode_packet_body` | Network | `int __stdcall(const uint8_t *src, int src_size, uint8_t *dest)` | Add sequence and apply ordinary client-body XOR passes. | [Networking](../subsystems/networking.md#sequence-and-xor-transformation) |
| `Darkages.exe:0x004A8414` | `net_set_xor_key` | Network | `void __stdcall(int length, const uint8_t *key)` | Replace and repeat the runtime XOR key. | [Networking](../subsystems/networking.md#sequence-and-xor-transformation) |
| `Darkages.exe:0x004A8590` | `net_build_xor_table` | Network | `void __stdcall(int function_index)` | Generate the 256-entry XOR table. | [Networking](../subsystems/networking.md#sequence-and-xor-transformation) |

## UI protocol dispatch

| Address | Current IDA name | Subsystem | Prototype | Purpose | Detailed page |
|---:|---|---|---|---|---|
| `Darkages.exe:0x0040B3A0` | `ui_handle_server_request_portrait` | Network and UI | `int __thiscall(void *this, void *event)` | Handle server action `0x49`. | [Server actions](../protocol/server-actions.md) |
| `Darkages.exe:0x0045F780` | `ui_main_menu_handle_server_packet` | Network and UI | `int __thiscall(void *this, void *event)` | Handle initial main-menu server packets. | [Server actions](../protocol/server-actions.md) |
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Network and UI | `int __thiscall(void *this, void *event)` | Dispatch 31 in-game server actions. | [Server actions](../protocol/server-actions.md#mappane-switch-coverage) |
| `Darkages.exe:0x0046B574` | `ui_map_handle_object_direction` | Network and UI | `int __thiscall(void *this, const uint8_t *packet)` | Apply server action `0x11` to a map object's direction. | [SChangeDirection](../protocol/server-messages.md#0x11-schangedirection) |
| `Darkages.exe:0x004921F0` | `ui_exit_wait_handle_server_packet` | Network and UI | `int __thiscall(void *this, void *event)` | Complete the subtype-one safe-exit exchange for server action `0x4C`. | [SReconnect](../protocol/server-messages.md#0x4c-sreconnect) |
| `Darkages.exe:0x00492310` | `ui_exit_wait_pane_ctor` | UI and network | established `__thiscall` constructor | Create the exit-wait pane and send `CQuit` subtype `0x01`. | [SReconnect](../protocol/server-messages.md#0x4c-sreconnect) |
| `Darkages.exe:0x004A2ED0` | `ui_server_select_handle_server_packet` | Network and UI | `int __thiscall(void *this, void *event)` | Deserialize the action `0x56` server table. | [Server actions](../protocol/server-actions.md) |

## Windows integration

| Address | Current IDA name | Subsystem | Prototype | Purpose | Detailed page |
|---:|---|---|---|---|---|
| `Darkages.exe:0x0045BDC0` | `win_main_window_proc` | Win32 and network | `LRESULT __stdcall(HWND, UINT, WPARAM, LPARAM)` | Route `WM_USER + 2` Winsock notifications. | [Networking](../subsystems/networking.md#connection-and-windows-notification) |

## Entry requirements

Add a function when it is useful outside its immediate subsystem context. Record the calling convention and types established by the code, and link to the detailed subsystem explanation.
