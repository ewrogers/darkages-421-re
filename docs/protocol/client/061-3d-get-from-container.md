# `CGetFromContainer` (`0x3D`)

[Previous: CPutToContainer](060-3c-put-to-container.md) | [Client action index](../client-actions.md) | [Next: CUseSkill](062-3e-use-skill.md)

`CGetFromContainer` is client-to-server action `0x3D` in the 4.21 protocol.

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
| `Darkages.exe:0x004412D8` | `sub_441234` | `Darkages.exe:0x00441234` |
| `Darkages.exe:0x00453A15` | `sub_4539B4` | `Darkages.exe:0x004539B4` |

## Schema status

The 4.21 client emits this action at the listed sites. The payload fields and client-side trigger still require tracing.
