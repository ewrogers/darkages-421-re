# `SLoginCheck` (`0x02`)

[Previous: SNewUserCheck](001-01-new-user-check.md) | [Server action index](../server-actions.md) | [Next: STransferServer](003-03-transfer-server.md)

`SLoginCheck` is server-to-client action `0x02` in the 4.21 protocol.

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
| `Darkages.exe:0x00461080` | `sub_461080` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x0045F780` | `ui_main_menu_handle_server_packet` | Accepts the action in the main-menu packet handler. |
| `Darkages.exe:0x00464300` | `sub_464300` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x00463A60` | `sub_463A60` | Accepts the action in its pane's Event type 9 packet handler. |

## Handler notes

`Darkages.exe:0x00461080` `sub_461080`, `Darkages.exe:0x0045F780` `ui_main_menu_handle_server_packet`, `Darkages.exe:0x00464300` `sub_464300`, `Darkages.exe:0x00463A60` `sub_463A60`. Used during login or lobby.

## Schema status

The 4.21 client accepts this action in the listed functions. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.
