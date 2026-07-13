# `CUseSkill` (`0x3E`)

[Previous: CGetFromContainer](061-3d-get-from-container.md) | [Client action index](../client-actions.md) | [Next: CFieldMap](063-3f-field-map.md)

`CUseSkill` is client-to-server action `0x3E` in the 4.21 protocol.

**Direction:** client to server

**Encrypted:** Yes. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `skill_slot` | Slot byte read from the activated skill pane at object `+0x276`. |

## Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x00454C96` | `net_c_send_use_skill` | `Darkages.exe:0x00454C64` |

## UI flow

`Darkages.exe:0x00454C64` `net_c_send_use_skill` builds exactly `[0x3E, skill_slot]`. The slot activation path can first send a length-prefixed `CSpellDelaySay` `0x4E` phrase loaded for the skill, then sends `CUseSkill`. Right-button down is separate and opens the local `SkillBookDialog` without network traffic.

See [UI, Input, and Packet Flows](../../architecture/ui-network-flows.md#skill-and-spell-inventories).
