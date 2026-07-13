# `CGiveGold` (`0x2A`)

[Previous: CGive](041-29-give.md) | [Client action index](../client-actions.md) | [Next: CSelfLook](045-2d-self-look.md)

`CGiveGold` is client-to-server action `0x2A` in the 4.21 protocol.

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
| `Darkages.exe:0x0045360B` | `sub_453564` | `Darkages.exe:0x00453564` |

## Schema status

The 4.21 client emits this action at the listed sites. The payload fields and client-side trigger still require tracing.
