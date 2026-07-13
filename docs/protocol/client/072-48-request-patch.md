# `CRequestPatch` (`0x48`)

[Previous: CAddStat](071-47-add-stat.md) | [Client action index](../client-actions.md) | [Next: CExchange](074-4a-exchange.md)

`CRequestPatch` is the supplied later-client message name for client-direction action `0x48`. It is not RTTI or a symbol recovered from the 4.21 executable.

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
| `Darkages.exe:0x0048763C` | `sub_487604` | `Darkages.exe:0x00487604` |
| `Darkages.exe:0x004953EC` | `sub_495380` | `Darkages.exe:0x00495380` |
| `Darkages.exe:0x00495F55` | `sub_495E80` | `Darkages.exe:0x00495E80` |
| `Darkages.exe:0x0049609D` | `sub_496054` | `Darkages.exe:0x00496054` |

## Schema status

The `CRequestPatch` name is a later-client correlation. Stone emits action `0x48` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.
