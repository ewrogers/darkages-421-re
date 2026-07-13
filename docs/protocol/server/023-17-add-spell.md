# `SAddSpell` (`0x17`)

[Previous: SMapInfo](021-15-map-info.md) | [Server action index](../server-actions.md) | [Next: SRemoveSpell](024-18-remove-spell.md)

`SAddSpell` is server-to-client action `0x17` in the 4.21 protocol.

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

`Darkages.exe:0x00443B10` `ui_spell_inventory_handle_server_packet` removes any existing pane in the selected slot, allocates a `0x214`-byte spell slot pane, and adds and registers it below the spell inventory owner. `Darkages.exe:0x004889F0` also observes this action in its own registered pane context.

## UI behavior

This action changes the spell inventory's child panes but does not select or show the persistent spell inventory. The player selects that parent locally with D or its game button. Right-button up over a populated slot opens `SpellBookDialog` without another server request.

See [UI, Input, and Packet Flows](../../architecture/ui-network-flows.md#skill-and-spell-inventories).
