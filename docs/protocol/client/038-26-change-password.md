# `CChangePassword` (`0x26`)

[Previous: CDropGold](036-24-drop-gold.md) | [Client action index](../client-actions.md) | [Next: CGive](041-29-give.md)

`CChangePassword` is client-to-server action `0x26` in the 4.21 protocol.

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
| `Darkages.exe:0x00463A45` | `sub_4638C4` | `Darkages.exe:0x004638C4` |
| `Darkages.exe:0x004642F0` | `sub_464174` | `Darkages.exe:0x00464174` |

## Schema status

The 4.21 client emits this action at the listed sites. The payload fields and client-side trigger still require tracing.
