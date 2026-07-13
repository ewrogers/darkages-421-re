# `CSpellDelayRequest` (`0x4D`)

[Previous: CExchange](074-4a-exchange.md) | [Client action index](../client-actions.md) | [Next: CSpellDelaySay](078-4e-spell-delay-say.md)

`CSpellDelayRequest` is client-to-server action `0x4D` in the 4.21 protocol.

**Direction:** client to server

**Encrypted:** Yes. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `delay_seconds` | Configured number of one-second delay steps before the held spell packet is submitted. |

## Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x00455CDF` | `net_c_send_spell_delay_request` | `Darkages.exe:0x00455CA4` |

## Delay flow

`Darkages.exe:0x00455CA4` `net_c_send_spell_delay_request` submits exactly `[0x4D, delay_seconds]`. It is called after the final `CUseSpell` logical packet has already been copied into the spell-delay controller. Timed `CSpellDelaySay` packets follow, and the controller releases the held cast on the final tick.

See [UI, Input, and Packet Flows](../../architecture/ui-network-flows.md#spell-delay-sequence).
