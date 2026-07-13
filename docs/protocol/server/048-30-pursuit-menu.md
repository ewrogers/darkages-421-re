# `SPursuitMenu` (`0x30`)

[Previous: SScreenMenu](047-2f-screen-menu.md) | [Server action index](../server-actions.md) | [Next: SBulletin](049-31-bulletin.md)

`SPursuitMenu` is server-to-client action `0x30` in the 4.21 protocol.

**Direction:** server to client

**Encrypted:** Yes. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `dialog_subtype` | Selects one of seven dynamic message classes, values `0` through `6`. |
| `0x01` | variable | `dialog_data` | Subtype-specific message, face, identifiers, choices, and text. |

## Handler functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00461080` | `sub_461080` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |
| `Darkages.exe:0x00479480` | `sub_479480` | Accepts the action in its pane's Event type 9 packet handler. |

## Handler notes

`ui_map_dispatch_server_packet` calls `Darkages.exe:0x0046C004` `ui_map_handle_pursuit_menu`. Subtype `2` constructs `QuestionMessageDialog`; subtype `6` constructs `QuestionMessageFaceDialog`. The other five variants are dynamically allocated and registered, but their friendly source class names are not yet established. The handlers at `0x00461080` and `0x00479480` also observe pursuit traffic in their registered pane contexts.

## UI response path

The two confirmed question classes send selected answers through `CMessage` `0x3A`. Their answer builders retain the dialog type, object identifier, pursuit identifier, and step from the server packet, append response type `1` and the selected answer byte, and submit 12 logical bytes.

See [UI, Input, and Packet Flows](../../architecture/ui-network-flows.md#users-paper-and-server-menus).
