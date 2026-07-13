# `CGetFromContainer` (`0x3D`)

[Previous: CPutToContainer](060-3c-put-to-container.md) | [Client action index](../client-actions.md) | [Next: CUseSkill](062-3e-use-skill.md)

`CGetFromContainer` is the supplied later-client message name for client-direction action `0x3D`. It is not RTTI or a symbol recovered from the 4.21 executable.

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

The `CGetFromContainer` name is a later-client correlation. Stone emits action `0x3D` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.
