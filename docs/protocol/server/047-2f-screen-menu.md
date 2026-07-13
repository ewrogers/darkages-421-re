# `SScreenMenu` (`0x2F`)

[Previous: SFieldMap](046-2e-field-map.md) | [Server action index](../server-actions.md) | [Next: SPursuitMenu](048-30-pursuit-menu.md)

`SScreenMenu` is server-to-client action `0x2F` in the 4.21 protocol.

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
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |
| `Darkages.exe:0x00479480` | `sub_479480` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x0047AFD0` | `sub_47AFD0` | Accepts the action in its pane's Event type 9 packet handler. |

## Handler notes

`ui_map_dispatch_server_packet` calls `Darkages.exe:0x00469474` `ui_map_handle_screen_menu`, which allocates a `0x104`-byte full-screen pane and calls `ui_screen_menu_pane_ctor`. The constructor adds it below `BackgroundPane`, registers it, and parses the original action `0x2F` packet. The handlers at `0x00479480` and `0x0047AFD0` can receive later menu packets while related panes remain registered.

## UI response path

Selections and text controls in this menu family send `CMenuCode` `0x39`. Every traced builder copies a server-supplied menu type, big-endian object identifier, and big-endian menu code before variant-specific data.

See [UI, Input, and Packet Flows](../../architecture/ui-network-flows.md#users-paper-and-server-menus).
