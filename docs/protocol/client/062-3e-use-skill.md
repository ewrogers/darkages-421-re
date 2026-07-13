# `CUseSkill` (`0x3E`)

[Previous: CGetFromContainer](061-3d-get-from-container.md) | [Client action index](../client-actions.md) | [Next: CFieldMap](063-3f-field-map.md)

`CUseSkill` is client-to-server action `0x3E` in the 4.21 protocol.

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
| `Darkages.exe:0x00454C96` | `sub_454C64` | `Darkages.exe:0x00454C64` |

## Schema status

The 4.21 client emits this action at the listed sites. The payload fields and client-side trigger still require tracing.
