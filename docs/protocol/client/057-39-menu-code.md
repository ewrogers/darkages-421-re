# `CMenuCode` (`0x39`)

[Previous: CRefreshUser](056-38-refresh-user.md) | [Client action index](../client-actions.md) | [Next: CMessage](058-3a-message.md)

`CMenuCode` is the supplied later-client message name for client-direction action `0x39`. It is not RTTI or a symbol recovered from the 4.21 executable.

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
| `Darkages.exe:0x0047B8CD` | `sub_47B850` | `Darkages.exe:0x0047B850` |
| `Darkages.exe:0x0047BBB7` | `sub_47BB00` | `Darkages.exe:0x0047BB00` |
| `Darkages.exe:0x0047C175` | `sub_47C100` | `Darkages.exe:0x0047C100` |
| `Darkages.exe:0x0047C2CF` | `sub_47C220` | `Darkages.exe:0x0047C220` |
| `Darkages.exe:0x0047C60F` | `sub_47C560` | `Darkages.exe:0x0047C560` |
| `Darkages.exe:0x0047C8C8` | `sub_47C7C0` | `Darkages.exe:0x0047C7C0` |
| `Darkages.exe:0x0047D3BF` | `sub_47D310` | `Darkages.exe:0x0047D310` |
| `Darkages.exe:0x0047D46D` | `sub_47D3E0` | `Darkages.exe:0x0047D3E0` |
| `Darkages.exe:0x0047D7DC` | `sub_47D6E4` | `Darkages.exe:0x0047D6E4` |

## Schema status

The `CMenuCode` name is a later-client correlation. Stone emits action `0x39` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.
