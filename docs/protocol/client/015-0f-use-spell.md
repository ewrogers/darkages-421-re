# `CUseSpell` (`0x0F`)

[Previous: CSay](014-0e-say.md) | [Client action index](../client-actions.md) | [Next: CTransferServer](016-10-transfer-server.md)

`CUseSpell` is the documentation name for client-to-server action `0x0F` in the 4.21 client. The friendly source class name is not retained, but the action's spell-slot construction and delayed submission are established directly.

**Direction:** client to server

**Encrypted:** Yes. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Confirmed immediate payload form

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `spell_slot` | Slot byte read from the activated spell pane at object `+0xF4`. |

`Darkages.exe:0x00455684` `ui_spell_slot_begin_cast` builds exactly two logical bytes, `[0x0F, spell_slot]`, for the confirmed immediate path. Other targeting modes enter separate input panes; their final packet fields are not yet mapped.

## Delayed submission

`ui_spell_delay_begin` copies the complete logical packet and its length into the delay controller before deciding when to send it. With a zero delay it calls `net_c_queue_send` immediately. With a nonzero delay it sends `CSpellDelayRequest` `0x4D`, emits configured `CSpellDelaySay` `0x4E` phrases on 1000 ms timers, and queues the held `0x0F` packet on the final tick.

The delay phrases come from `SpellBook.cfg` and are matched to the spell name. This local delay means the final `CUseSpell` submission may be owned by a timer callback rather than the original mouse Event.

## Functions

| Address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00455684` | `ui_spell_slot_begin_cast` | Build the confirmed `[0x0F, slot]` form. |
| `Darkages.exe:0x004556F4` | `ui_spell_delay_begin` | Copy and hold the logical packet, or submit it immediately. |
| `Darkages.exe:0x00456940` | `ui_spell_delay_handle_timer` | Submit the held packet on the final delay tick. |

See [UI, Input, and Packet Flows](../../architecture/ui-network-flows.md#spell-delay-sequence) for the input-to-timer-to-network path.
