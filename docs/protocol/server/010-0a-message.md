# `SMessage` (`0x0A`)

[Previous: SStatus](008-08-status.md) | [Server action index](../server-actions.md) | [Next: SMove](011-0b-move.md)

`SMessage` is server-to-client action `0x0A` in the 4.21 protocol.

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

In game, MapPane handles only message subtypes `8`, `9`, and `10` in this branch. It reads the big-endian text length at action-relative packet `+2`, changes tab bytes to carriage returns, allocates a dynamic message dialog, and registers it below `BackgroundPane`. Subtype `10` selects the alternate constructor mode; `8` and `9` use mode 0.

## UI behavior

These subtypes create a new dialog rather than updating a static root. Other `SMessage` subtypes are not consumed by this MapPane branch and must not be assumed to share the same dialog format.

See [UI, Input, and Packet Flows](../../architecture/ui-network-flows.md#server-actions-that-affect-ui).
