# `SAddSkill` (`0x2C`)

[Previous: SRemoveContainer](043-2b-remove-container.md) | [Server action index](../server-actions.md) | [Next: SRemoveSkill](045-2d-remove-skill.md)

`SAddSkill` is server-to-client action `0x2C` in the 4.21 protocol.

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

`Darkages.exe:0x00442990` `ui_skill_inventory_handle_server_packet` removes any existing pane in the selected slot, allocates a `0x294`-byte skill slot pane, and adds and registers it below the skill inventory owner.

## UI behavior

This action changes skill slot children but does not select or show the persistent skill inventory. The player selects that parent locally with S or its game button. Right-button down over a populated slot opens `SkillBookDialog` without another server request.

See [UI, Input, and Packet Flows](../../architecture/ui-network-flows.md#skill-and-spell-inventories).
