# `CException` (`0x42`)

[Previous: CGetParcel](065-41-get-parcel.md) | [Client action index](../client-actions.md) | [Next: CRequestObjectInfo](067-43-request-object-info.md)

`CException` is the supplied later-client message name for client-direction action `0x42`. It is not RTTI or a symbol recovered from the 4.21 executable.

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
| `Darkages.exe:0x00434B43` | `sub_434A40` | `Darkages.exe:0x00434A40` |

## Schema status

The `CException` name is a later-client correlation. Stone emits action `0x42` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.
