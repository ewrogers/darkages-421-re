# `SMessage` (`0x0A`)

[Previous: SStatus](008-08-status.md) | [Server action index](../server-actions.md) | [Next: SMove](011-0b-move.md)

`SMessage` is the supplied later-client message name for server-direction action `0x0A`. It is not RTTI or a symbol recovered from the 4.21 executable.

**Direction:** server to client

**Encrypted:** Yes. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | `payload_length` | `unknown_00` | Payload bytes whose field boundaries are not yet mapped. |

## Handler functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x0041CCA0` | `sub_41CCA0` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x00444690` | `sub_444690` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x0045F780` | `ui_main_menu_handle_server_packet` | Accepts the action in the main-menu packet handler. |
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |
| `Darkages.exe:0x00491520` | `sub_491520` | Accepts the action in its pane's Event type 9 packet handler. |

## Handler notes

`Darkages.exe:0x0041CCA0` `sub_41CCA0`, `Darkages.exe:0x00444690` `sub_444690`, `Darkages.exe:0x0045F780` `ui_main_menu_handle_server_packet`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, `Darkages.exe:0x00491520` `sub_491520`.

## Schema status

The `SMessage` name is a later-client cross-version reference. Stone accepts action `0x0A` in the listed functions. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.
