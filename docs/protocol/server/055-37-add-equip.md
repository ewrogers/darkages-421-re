# `SAddEquip` (`0x37`)

[Previous: SShowUsers](054-36-show-users.md) | [Server action index](../server-actions.md) | [Next: SRemoveEquip](056-38-remove-equip.md)

`SAddEquip` is server-to-client action `0x37` in the 4.21 protocol.

**Direction:** server to client

**Encrypted:** Yes. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `slot` | Equipment slot index. |
| `0x01` | 2 | `item_id` | Big-endian equipment icon or item identifier. |
| `0x03` | 1 | `name_length` | Number of following name bytes. |
| `0x04` | `name_length` | `name` | Equipment display name bytes. |

## Handler functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x0042EAF0` | `ui_equip_pane_handle_server_packet` | Accepts the action in its pane's Event type 9 packet handler. |

## Handler notes

`Darkages.exe:0x0042EAF0` `ui_equip_pane_handle_server_packet`.

## UI behavior

`ui_equip_pane_add_item` stores the identifier in the two-byte slot array beginning at pane `+0xB22` and the name in `0x80`-byte records beginning at `+0xB3E`, then redraws. It does not show the equipment pane.

See [UI, Input, and Packet Flows](../../architecture/ui-network-flows.md#equipment-and-self-look).
