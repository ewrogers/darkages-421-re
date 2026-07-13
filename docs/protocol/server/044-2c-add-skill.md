# `SAddSkill` (`0x2C`)

[Previous: SRemoveContainer](043-2b-remove-container.md) | [Server action index](../server-actions.md) | [Next: SRemoveSkill](045-2d-remove-skill.md)

`SAddSkill` is server-to-client action `0x2C` in the 4.21 protocol.

**Direction:** server to client

**Encrypted:** Yes. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `slot` | 1-based skill inventory slot, valid range 1 through 36. |
| `0x01` | 2 | `display_id` | Big-endian display identifier retained by the slot pane. |
| `0x03` | 1 | `name_length` | Number of following skill-name bytes. |
| `0x04` | `name_length` | `name` | Skill name bytes. There is no terminator in the packet; the client appends one. |

The payload occupies `4 + name_length` bytes. The string encoding is not established by this parser. `ui_skill_inventory_apply_add_packet` checks the slot range but does not compare `name_length` with the remaining decoded packet size before copying. The downstream slot constructor asserts that the resulting NUL-terminated name is shorter than 128 bytes.

## Handler functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00442990` | `ui_skill_inventory_handle_server_packet` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x00442A34` | `ui_skill_inventory_apply_add_packet` | Parses the fields and replaces the indexed slot child. |
| `Darkages.exe:0x00442C04` | `ui_skill_inventory_remove_slot_pane` | Removes and deletes an existing slot child. |
| `Darkages.exe:0x00442D14` | `ui_skill_inventory_create_slot_pane` | Allocates and registers a `0x294`-byte slot pane. |
| `Darkages.exe:0x00456B90` | `ui_skill_slot_pane_ctor` | Retains the display ID, name, and 1-based slot in the new child. |

## Handler notes

`ui_skill_inventory_apply_add_packet` parses the action-bearing decoded packet at offsets `+1` through `+4`, corresponding to payload offsets `0x00` through `0x03`. It rejects a slot outside 1 through 36. For a valid slot, it removes any existing pane, allocates a `0x294`-byte skill slot pane, and adds and registers it below the skill inventory owner.

## UI behavior

This action changes skill slot children but does not select or show the persistent skill inventory. The player selects that parent locally with S or its game button. Right-button down over a populated slot opens `SkillBookDialog` without another server request.

The parent owns a 36-entry pointer array and the child retains its packet-derived state at stable offsets. See [Runtime UI Memory Map](../../appendices/runtime-ui-memory-map.md#skill-inventory-and-slot-panes) and [UI, Input, and Packet Flows](../../architecture/ui-network-flows.md#skill-and-spell-inventories).
