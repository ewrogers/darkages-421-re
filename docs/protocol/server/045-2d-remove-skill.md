# `SRemoveSkill` (`0x2D`)

[Previous: SAddSkill](044-2c-add-skill.md) | [Server action index](../server-actions.md) | [Next: SFieldMap](046-2e-field-map.md)

`SRemoveSkill` is server-to-client action `0x2D` in the 4.21 protocol.

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
| `Darkages.exe:0x00442990` | `ui_skill_inventory_handle_server_packet` | Accepts the action in its pane's Event type 9 packet handler. |

## Handler notes

`Darkages.exe:0x00442990` `ui_skill_inventory_handle_server_packet` routes this action to the skill-slot removal path. That path unregisters and removes the child, deletes it, and clears the indexed owner pointer.

## UI behavior

Removing a slot does not hide the parent skill inventory. See [UI, Input, and Packet Flows](../../architecture/ui-network-flows.md#skill-and-spell-inventories).
