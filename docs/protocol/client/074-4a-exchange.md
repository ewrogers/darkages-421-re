# `CExchange` (`0x4A`)

[Previous: CRequestPatch](072-48-request-patch.md) | [Client action index](../client-actions.md) | [Next: CSpellDelayRequest](077-4d-spell-delay-request.md)

`CExchange` is the supplied later-client message name for client-direction action `0x4A`. It is not RTTI or a symbol recovered from the 4.21 executable.

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
| `Darkages.exe:0x0043071A` | `sub_4306C4` | `Darkages.exe:0x004306C4` |
| `Darkages.exe:0x0043596E` | `sub_435904` | `Darkages.exe:0x00435904` |
| `Darkages.exe:0x00435A5A` | `sub_435A04` | `Darkages.exe:0x00435A04` |
| `Darkages.exe:0x00435ACA` | `sub_435A74` | `Darkages.exe:0x00435A74` |
| `Darkages.exe:0x00435E90` | `sub_435E24` | `Darkages.exe:0x00435E24` |
| `Darkages.exe:0x00437310` | `sub_4372A4` | `Darkages.exe:0x004372A4` |
| `Darkages.exe:0x00437AE8` | `sub_437A64` | `Darkages.exe:0x00437A64` |

## Schema status

The `CExchange` name is a later-client correlation. Stone emits action `0x4A` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.
