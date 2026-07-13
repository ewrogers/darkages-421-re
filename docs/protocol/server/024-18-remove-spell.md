# `SRemoveSpell` (`0x18`)

[Previous: SAddSpell](023-17-add-spell.md) | [Server action index](../server-actions.md) | [Next: SSoundEffect](025-19-sound-effect.md)

`SRemoveSpell` is server-to-client action `0x18` in the 4.21 protocol.

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
| `Darkages.exe:0x00443B10` | `ui_spell_inventory_handle_server_packet` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x004889F0` | `sub_4889F0` | Accepts the action in its pane's Event type 9 packet handler. |

## Handler notes

`Darkages.exe:0x00443B10` `ui_spell_inventory_handle_server_packet` routes this action to the spell-slot removal path. That path unregisters and removes the child, deletes it, and clears the indexed owner pointer. `Darkages.exe:0x004889F0` also observes the action in its own pane context.

## UI behavior

Removing a slot does not hide the parent spell inventory. See [UI, Input, and Packet Flows](../../architecture/ui-network-flows.md#skill-and-spell-inventories).
