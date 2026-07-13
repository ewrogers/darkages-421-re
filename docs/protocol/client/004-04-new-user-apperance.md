# `CNewUserApperance` (`0x04`)

[Previous: CLogin](003-03-login.md) | [Client action index](../client-actions.md) | [Next: CMapRequest](005-05-map-request.md)

`CNewUserApperance` is client-to-server action `0x04` in the 4.21 protocol.

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
| `Darkages.exe:0x00461725` | `sub_4616A4` | `Darkages.exe:0x004616A4` |

## Schema status

The 4.21 client emits this action at the listed sites. The payload fields and client-side trigger still require tracing.
