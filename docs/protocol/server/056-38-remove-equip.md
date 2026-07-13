# `SRemoveEquip` (`0x38`)

[Previous: SAddEquip](055-37-add-equip.md) | [Server action index](../server-actions.md) | [Next: SSelfLook](057-39-self-look.md)

`SRemoveEquip` is server-to-client action `0x38` in the 4.21 protocol.

**Direction:** server to client

**Encrypted:** Yes. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `slot` | Equipment slot whose identifier and name are cleared. |

## Handler functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x0042EAF0` | `ui_equip_pane_handle_server_packet` | Accepts the action in its pane's Event type 9 packet handler. |

## Handler notes

`Darkages.exe:0x0042EAF0` `ui_equip_pane_handle_server_packet`.

## UI behavior

`ui_equip_pane_remove_item` clears the slot identifier and first byte of its name, then redraws. It does not hide the equipment pane.

See [UI, Input, and Packet Flows](../../architecture/ui-network-flows.md#equipment-and-self-look).
