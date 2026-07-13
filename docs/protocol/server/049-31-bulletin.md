# `SBulletin` (`0x31`)

[Previous: SPursuitMenu](048-30-pursuit-menu.md) | [Server action index](../server-actions.md) | [Next: SStateObjectState](050-32-state-object-state.md)

`SBulletin` is server-to-client action `0x31` in the 4.21 protocol.

**Direction:** server to client

**Encrypted:** Yes. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `response_subtype` | Selects board list `1`, article list `2`, article `3`, mail list `4`, mail `5`, or acknowledgement `6`. |
| `0x01` | variable | `response_data` | Subtype-specific board, article, mail, or completion data. |

## Handler functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00413CE0` | `ui_bulletin_session_handle_server_packet` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x00414260` | `ui_bulletin_dialog_handle_server_packet` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

## Handler notes

`Darkages.exe:0x00413CE0` `ui_bulletin_session_handle_server_packet`, `Darkages.exe:0x00414260` `ui_bulletin_dialog_handle_server_packet`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`.

## UI behavior

An existing `BulletinSession` claims the packet through its Event type 9 handler and replaces its active child according to the subtype. Board, article-list, article, mail-list, and mail responses construct the correspondingly named dialog. Subtype `6` is accepted without constructing a child. Article-list and mail-list responses require the session request-wait flag.

When no session exists, `ui_map_handle_bulletin` can construct a server-opened session from this packet. Packet byte `+2` bit 0 suppresses that unsolicited creation path when set.

See [UI, Input, and Packet Flows](../../architecture/ui-network-flows.md#bulletin-and-mail-child-replacement).
