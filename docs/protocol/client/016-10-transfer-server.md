# `CTransferServer` (`0x10`)

[Previous: CSay](014-0e-say.md) | [Client action index](../client-actions.md) | [Next: CChangeDirection](017-11-change-direction.md)

`CTransferServer` is client-to-server action `0x10` in the 4.21 protocol.

**Direction:** client to server

**Encrypted:** No. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | `payload_length` | `unknown_00` | Payload bytes whose field boundaries are not yet mapped. |

## Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x00460617` | `sub_460584` | `Darkages.exe:0x00460584` |
| `Darkages.exe:0x00463119` | `sub_463010` | `Darkages.exe:0x00463010` |
| `Darkages.exe:0x0046C901` | `sub_46C7D4` | `Darkages.exe:0x0046C7D4` |

## Schema status

The payload fields remain placeholders. Stone diagnostics independently name `kClientTransferServer`, and the Socket transfer gate permits only this action while a server transfer is active.
