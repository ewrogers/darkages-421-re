# `CNewUserApperance` (`0x04`)

[Previous: CLogin](003-03-login.md) | [Client action index](../client-actions.md) | [Next: CMapRequest](005-05-map-request.md)

`CNewUserApperance` is the supplied later-client message name for client-direction action `0x04`. It is not RTTI or a symbol recovered from the 4.21 executable.

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

The `CNewUserApperance` name is a later-client correlation. Stone emits action `0x04` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.
