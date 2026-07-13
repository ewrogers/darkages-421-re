# `CExitEditingMode` (`0x23`)

[Previous: CEmotion](029-1d-emotion.md) | [Client action index](../client-actions.md) | [Next: CDropGold](036-24-drop-gold.md)

`CExitEditingMode` is client-to-server action `0x23` in the 4.21 protocol.

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
| `Darkages.exe:0x00493C99` | `sub_493B60` | `Darkages.exe:0x00493B60` |
| `Darkages.exe:0x0049421C` | `sub_494100` | `Darkages.exe:0x00494100` |

## Schema status

The 4.21 client emits this action at the listed sites. The payload fields and client-side trigger still require tracing.
