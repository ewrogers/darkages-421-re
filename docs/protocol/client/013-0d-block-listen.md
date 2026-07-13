# `CBlockListen` (`0x0D`)

[Previous: CPutGround](012-0c-put-ground.md) | [Client action index](../client-actions.md) | [Next: CSay](014-0e-say.md)

`CBlockListen` is the supplied later-client message name for client-direction action `0x0D`. It is not RTTI or a symbol recovered from the 4.21 executable.

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
| `Darkages.exe:0x004C1691` | `sub_4C1654` | `Darkages.exe:0x004C1654` |
| `Darkages.exe:0x004C1956` | `sub_4C18A4` | `Darkages.exe:0x004C18A4` |
| `Darkages.exe:0x004C1A66` | `sub_4C19B4` | `Darkages.exe:0x004C19B4` |

## Schema status

The `CBlockListen` name is a later-client correlation. Stone emits action `0x0D` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.
