# `CSpellDelaySay` (`0x4E`)

[Previous: CSpellDelayRequest](077-4d-spell-delay-request.md) | [Client action index](../client-actions.md) | [Next: CSendPortrait](079-4f-send-portrait.md)

`CSpellDelaySay` is the supplied later-client message name for client-direction action `0x4E`. It is not RTTI or a symbol recovered from the 4.21 executable.

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
| `Darkages.exe:0x004548F1` | `sub_454884` | `Darkages.exe:0x00454884` |
| `Darkages.exe:0x00455921` | `sub_455834` | `Darkages.exe:0x00455834` |
| `Darkages.exe:0x00455815` | `sub_4556F4` | `Darkages.exe:0x004556F4` |
| `Darkages.exe:0x00456997` | `sub_456940` | `Darkages.exe:0x00456940` |

## Schema status

The `CSpellDelaySay` name is a later-client correlation. Stone emits action `0x4E` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.
