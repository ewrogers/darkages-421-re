# `SAddSpell` (`0x17`)

[Previous: SMapInfo](021-15-map-info.md) | [Server action index](../server-actions.md) | [Next: SRemoveSpell](024-18-remove-spell.md)

`SAddSpell` is server-to-client action `0x17` in the 4.21 protocol.

**Direction:** server to client

**Encrypted:** Yes. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `slot` | 1-based spell inventory slot, valid range 1 through 36. |
| `0x01` | 2 | `display_id` | Big-endian display identifier retained by the slot pane. |
| `0x03` | 1 | `spell_type` | Signed byte in the established range 1 through 8; selects the client activation path. |
| `0x04` | 1 | `name_length` | Number of following spell-name bytes. |
| `0x05` | `name_length` | `name` | Spell name bytes. There is no terminator in the packet; the client appends one. |
| `0x05 + name_length` | 1 | `comment_length` | Number of following comment bytes. |
| `0x06 + name_length` | `comment_length` | `comment` | Spell comment bytes. There is no terminator in the packet; the client appends one. |
| `0x06 + name_length + comment_length` | 1 | `unknown_06_plus_name_length_plus_comment_length` | Trailing byte retained at slot pane `+0x1F9` and passed to SpellBookDialog. |

The payload occupies `7 + name_length + comment_length` bytes. The string encoding and trailing-byte meaning are not established by this parser. `ui_spell_inventory_apply_add_packet` explicitly checks the slot range but does not compare either string length with the remaining decoded packet size before copying. The downstream constructor asserts that each NUL-terminated string is shorter than 128 bytes and that `spell_type` is 1 through 8.

## Handler functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00443B10` | `ui_spell_inventory_handle_server_packet` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x00443BB4` | `ui_spell_inventory_apply_add_packet` | Parses the fields and replaces the indexed slot child. |
| `Darkages.exe:0x00443E14` | `ui_spell_inventory_remove_slot_pane` | Removes and deletes an existing slot child. |
| `Darkages.exe:0x00443F24` | `ui_spell_inventory_create_slot_pane` | Allocates and registers a `0x214`-byte slot pane. |
| `Darkages.exe:0x00456C70` | `ui_spell_slot_pane_ctor` | Retains the slot, display ID, type, strings, and trailing byte. |
| `Darkages.exe:0x004889F0` | `ui_local_user_handle_server_packet` | Accepts the action in the local-user Pane's Event type 9 packet handler. |

## Handler notes

`ui_spell_inventory_apply_add_packet` parses the action-bearing decoded packet at offsets `+1` onward, corresponding to the payload table above. It rejects a slot outside 1 through 36. For a valid slot, it removes any existing pane, allocates a `0x214`-byte spell slot pane, and adds and registers it below the spell inventory owner. `Darkages.exe:0x004889F0` also observes this action in its own registered pane context.

## UI behavior

This action changes the spell inventory's child panes but does not select or show the persistent spell inventory. The player selects that parent locally with D or its game button. Right-button up over a populated slot opens `SpellBookDialog` without another server request.

The parent owns a 36-entry pointer array and the child retains all packet-derived state at stable offsets. See [Runtime UI Memory Map](../../appendices/runtime-ui-memory-map.md#spell-inventory-and-slot-panes) and [UI, Input, and Packet Flows](../../architecture/ui-network-flows.md#skill-and-spell-inventories).
