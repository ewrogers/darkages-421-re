# `CBulletin` (`0x3B`)

[Previous: CMessage](058-3a-message.md) | [Client action index](../client-actions.md) | [Next: CPutToContainer](060-3c-put-to-container.md)

`CBulletin` is the supplied later-client message name for client-direction action `0x3B`. It is not RTTI or a symbol recovered from the 4.21 executable.

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
| `Darkages.exe:0x0040ECE1` | `sub_40ECA4` | `Darkages.exe:0x0040ECA4` |
| `Darkages.exe:0x004109EC` | `sub_410964` | `Darkages.exe:0x00410964` |
| `Darkages.exe:0x0041294C` | `sub_4128C4` | `Darkages.exe:0x004128C4` |
| `Darkages.exe:0x004145B4` | `sub_414534` | `Darkages.exe:0x00414534` |
| `Darkages.exe:0x00415738` | `sub_4156B4` | `Darkages.exe:0x004156B4` |
| `Darkages.exe:0x00415934` | `sub_4158C4` | `Darkages.exe:0x004158C4` |
| `Darkages.exe:0x0041715C` | `sub_4170D4` | `Darkages.exe:0x004170D4` |
| `Darkages.exe:0x0041817E` | `sub_417F94` | `Darkages.exe:0x00417F94` |
| `Darkages.exe:0x004187E8` | `sub_418764` | `Darkages.exe:0x00418764` |
| `Darkages.exe:0x0041A73C` | `sub_41A6B4` | `Darkages.exe:0x0041A6B4` |
| `Darkages.exe:0x0041AE45` | `sub_41AB84` | `Darkages.exe:0x0041AB84` |
| `Darkages.exe:0x0041B4E4` | `sub_41B474` | `Darkages.exe:0x0041B474` |
| `Darkages.exe:0x0041B60B` | `sub_41B5A4` | `Darkages.exe:0x0041B5A4` |
| `Darkages.exe:0x0041B8A8` | `sub_41B824` | `Darkages.exe:0x0041B824` |
| `Darkages.exe:0x0041BA1B` | `sub_41B9B4` | `Darkages.exe:0x0041B9B4` |

## Schema status

The `CBulletin` name is a later-client correlation. Stone emits action `0x3B` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.
