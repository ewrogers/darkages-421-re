# `CVersion` (`0x00`)

[Client action index](../client-actions.md) | [Next: CNewUser](002-02-new-user.md)

`CVersion` is client-to-server action `0x00` in the 4.21 protocol.

**Direction:** client to server

**Encrypted:** No. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 2 | `data_version` | Unsigned big-endian value selected by the lobby startup code. |
| `0x02` | 2 | `client_tag` | Literal ASCII `LK`. |

## Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x004AD6A3` | `sub_4AD140` | `Darkages.exe:0x004AD140` |

## Schema status

The builder establishes the complete four-byte payload. It submits the action followed by these four bytes before the send queue adds the trailing zero.
