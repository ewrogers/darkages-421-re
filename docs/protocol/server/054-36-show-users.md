# `SShowUsers` (`0x36`)

[Previous: SShowPaper](053-35-show-paper.md) | [Server action index](../server-actions.md) | [Next: SAddEquip](055-37-add-equip.md)

`SShowUsers` is server-to-client action `0x36` in the 4.21 protocol.

**Direction:** server to client

**Encrypted:** Yes. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 2 | `unknown_00` | Big-endian list metadata retained by the pane. |
| `0x02` | 2 | `user_count` | Big-endian number of variable-length user records. |
| `0x04` | variable | `users` | Records containing flags, icon or status values, a name up to 48 bytes, another byte, and a secondary string up to 24 bytes. |

## Handler functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

## Handler notes

`ui_map_dispatch_server_packet` calls `Darkages.exe:0x0046C464` `ui_map_handle_show_users`. It obtains the static Users Dialog Pane, parses the list, then calls `ui_users_dialog_show`.

## UI flow

The pane is created lazily and hidden when the player first presses E or its window button. That input sends `CWho` `0x18`. `SShowUsers` is the step that populates row controls, sets the visible state, installs a 100 ms pane timer, and invalidates the pane region.

See [UI, Input, and Packet Flows](../../architecture/ui-network-flows.md#users-paper-and-server-menus).
