# Data Map

This appendix tracks important globals, tables, structures, virtual tables, and persistent buffers.

| Address | Current IDA name | Type or size | Owner | Purpose | Detailed page | Notes |
|---:|---|---|---|---|---|---|
| `Darkages.exe:0x004E0F00` | `net_xor_table` | `uint32_t[256]` | Socket transformation | Generated XOR byte table. | [Networking](../subsystems/networking.md#sequence-and-xor-transformation) | Each dword repeats one byte four times. Initial contents implement function 0. |
| `Darkages.exe:0x004FD5A0` | `net_xor_key` | key storage, at least 13 bytes | Socket transformation | Runtime repeating XOR key. | [Networking](../subsystems/networking.md#sequence-and-xor-transformation) | Constructor default is ASCII `NexonInc.`. Negotiated updates require at most 9 bytes. |
| `Darkages.exe:0x004FD5AC` | `net_xor_key_length` | `int32_t` | Socket transformation | Number of bytes in the active key. | [Networking](../subsystems/networking.md#sequence-and-xor-transformation) | Default is 9. |
| `Darkages.exe:0x004FD5B0` | `net_xor_key_repeated` | `uint8_t[48]` | Socket transformation | Four adjacent copies of the key. | [Networking](../subsystems/networking.md#sequence-and-xor-transformation) | Supports optimized four-byte XOR loops. |
| `Darkages.exe:0x004FD5E0` | `net_c_send_sequence` | `uint8_t` | Socket transformation | Sequence inserted after ordinary transformed client actions. | [Networking](../subsystems/networking.md#sequence-and-xor-transformation) | Zero-initialized at process load and incremented modulo 256 only by the transformed client encoder; cleartext bypass actions do not consume it. |
| `Darkages.exe:0x005177C0` | `ui_main_menu_pane_vtable` | virtual table | `MainMenuPane` | Includes the server-packet virtual handler. | [Networking](../subsystems/networking.md#packet-representation-and-c-class-evidence) | Packet handler is `ui_main_menu_handle_server_packet`. |
| `Darkages.exe:0x00519280` | `ui_map_pane_vtable` | virtual table | `MapPane` | Includes the main in-game server action switch. | [Server actions](../protocol/server-actions.md#mappane-switch-coverage) | Socket Event virtual slot `+0x40` points to `ui_map_dispatch_server_packet`. |
| `Darkages.exe:0x00526140` | `ui_server_select_dialog_pane_vtable` | virtual table | `ServerSelectDialogPane` | Includes the action `0x56` server-list handler. | [Server actions](../protocol/server-actions.md) | Constructor and deletion diagnostics establish the class name. |
| `Darkages.exe:0x00526940` | `net_socket_vtable` | virtual table | `Socket` | Socket work-item interface. | [Networking](../subsystems/networking.md) | Installed by `net_socket_ctor`. |

## Socket fields used by binary TCP

Offsets below are from the Socket object passed as `this`. They are established by the constructor and the mapped receive and send paths. The complete class layout is not yet named.

| Offset | Width | Working field | Reads and writes |
|---:|---:|---|---|
| `+0x0068` | 4 | `event_sink` | Used as `this` for `event_post_socket_bytes`. |
| `+0x006C` | 4 | `receive_cache` | Points at the object's `0x18000` byte transport cache. |
| `+0x48074` | 4 | `receive_cursor` | Index of the next cached transport byte. |
| `+0x48078` | 4 | `receive_remaining` | Cached bytes remaining after the current byte. |
| `+0x4807C` | 4 | `current_body_length` | Big-endian length assembled from the two frame header bytes. |
| `+0x48084` | 4 | `serial_handle` | Set to `INVALID_HANDLE_VALUE` when unused. |
| `+0x480BC` | 1 | `socket_connected` | Cleared on `recv == 0` and disconnect cleanup. |
| `+0x480C0` | 4 | `socket` | Winsock `SOCKET`; `INVALID_SOCKET` is `0xFFFFFFFF`. |
| `+0x480C5` | `0x10000` | `wire_body_buffer` | Receives body bytes beginning with the server action. |
| `+0x580C8` | 4 | `frame_offset` | Counts length-header and body bytes while a frame is open. |
| `+0x680CC` | at least `0x10000` | `decoded_or_encoded_buffer` | Destination for packet-body XOR transformation. |
| `+0x780CC` | 1 | `transport_mode` | Value 5 selects the normal TCP byte path. |
| `+0x780CD` | 1 | `frame_open` | Set after consuming `0xAA`; cleared on body completion. |
| `+0x780CF` | 1 | `legacy_send_sequence` | Decimal digit sequence used by the printable legacy transport. |
| `+0x780D2` | 1 | `transfer_gate` | When set, only client action `0x10` may be queued. |

## Socket Event fields

The Event object created for a decoded packet has these fields:

| Offset | Width | Meaning | Lifetime |
|---:|---:|---|---|
| `+0x0C` | 1 | Event type, value 9 for a socket packet. | Set before queueing. |
| `+0x14` | 4 | Owned decoded-packet pointer. | Freed by the event manager after pane dispatch. |
| `+0x18` | 4 | Decoded packet size. | Positive when the pointer is present. |

Structures should use `unknown_XX` for an established field location whose meaning remains unknown.
