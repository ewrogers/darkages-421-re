# `CSendPortrait` (`0x4F`)

[Previous: CSpellDelaySay](078-4e-spell-delay-say.md) | [Client action index](../client-actions.md) | [Next: CMulti](087-57-multi.md)

`CSendPortrait` is the supplied later-client message name for client-direction action `0x4F`. It is not RTTI or a symbol recovered from the 4.21 executable.

**Direction:** client to server

**Encrypted:** Yes. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | `payload_length` | `unknown_00` | Payload bytes whose field boundaries are not yet mapped. |

## Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x0040A7E3` | `net_c_build_portrait_response` | `Darkages.exe:0x0040A664` |

## Schema status

The action purpose is independently established: `net_c_build_portrait_response` validates a local portrait and sends it after server action `0x49`. Individual payload fields remain to be mapped.
