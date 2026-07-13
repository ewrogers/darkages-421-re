# `CChangeSlot` (`0x30`)

[Previous: CGroupToggle](047-2f-group-toggle.md) | [Client action index](../client-actions.md) | [Next: CRefreshUser](056-38-refresh-user.md)

`CChangeSlot` is the supplied later-client message name for client-direction action `0x30`. It is not RTTI or a symbol recovered from the 4.21 executable.

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
| `Darkages.exe:0x0042499F` | `sub_424914` | `Darkages.exe:0x00424914` |
| `Darkages.exe:0x00441216` | `sub_441164` | `Darkages.exe:0x00441164` |
| `Darkages.exe:0x004427DF` | `sub_442684` | `Darkages.exe:0x00442684` |
| `Darkages.exe:0x0044395F` | `sub_443804` | `Darkages.exe:0x00443804` |

## Schema status

The `CChangeSlot` name is a later-client correlation. Stone emits action `0x30` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.
