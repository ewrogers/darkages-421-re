# `CSpellDelaySay` (`0x4E`)

[Previous: CSpellDelayRequest](077-4d-spell-delay-request.md) | [Client action index](../client-actions.md) | [Next: CSendPortrait](079-4f-send-portrait.md)

`CSpellDelaySay` is client-to-server action `0x4E` in the 4.21 protocol.

**Direction:** client to server

**Encrypted:** Yes. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `text_length` | Number of following phrase bytes, truncated to one byte. |
| `0x01` | `text_length` | `text` | Delay phrase bytes copied without a terminator. |

## Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x004548F1` | `net_c_send_skill_delay_say` | `Darkages.exe:0x00454884` |
| `Darkages.exe:0x00455921` | `net_c_send_spell_delay_say` | `Darkages.exe:0x00455834` |
| `Darkages.exe:0x00455815` | `ui_spell_delay_begin` | `Darkages.exe:0x004556F4` |
| `Darkages.exe:0x00456997` | `ui_spell_delay_handle_timer` | `Darkages.exe:0x00456940` |

## Delay flow

Both the skill and spell paths build `CSpellDelaySay` as action `0x4E`, a one-byte length, and the selected phrase. Spell delay phrases are loaded from `SpellBook.cfg` and emitted once per timer step before the held `CUseSpell` packet is released. The skill slot can emit its configured phrase immediately before `CUseSkill`.

See [UI, Input, and Packet Flows](../../architecture/ui-network-flows.md#spell-delay-sequence).
