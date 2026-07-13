# `CChangeDirection` (`0x11`)

[Previous: CTransferServer](016-10-transfer-server.md) | [Client action index](../client-actions.md) | [Next: CAttack](019-13-attack.md)

`CChangeDirection` is client-to-server action `0x11` in the 4.21 protocol.

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
| `Darkages.exe:0x0046822F` | `sub_4681E4` | `Darkages.exe:0x004681E4` |
| `Darkages.exe:0x004884D4` | `sub_4884B4` | `Darkages.exe:0x004884B4` |

## Schema status

The 4.21 client emits this action at the listed sites. The payload fields and client-side trigger still require tracing.
