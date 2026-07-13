# `CMessage` (`0x3A`)

[Previous: CMenuCode](057-39-menu-code.md) | [Client action index](../client-actions.md) | [Next: CBulletin](059-3b-bulletin.md)

`CMessage` is client-to-server action `0x3A` in the 4.21 protocol.

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
| `Darkages.exe:0x0047E85C` | `sub_47E7D0` | `Darkages.exe:0x0047E7D0` |
| `Darkages.exe:0x0047F9AC` | `sub_47F920` | `Darkages.exe:0x0047F920` |
| `Darkages.exe:0x0047FA89` | `sub_47F9C0` | `Darkages.exe:0x0047F9C0` |
| `Darkages.exe:0x004805CC` | `sub_480540` | `Darkages.exe:0x00480540` |
| `Darkages.exe:0x004806A9` | `sub_4805E0` | `Darkages.exe:0x004805E0` |
| `Darkages.exe:0x0048113C` | `sub_4810B0` | `Darkages.exe:0x004810B0` |
| `Darkages.exe:0x004812AE` | `sub_481150` | `Darkages.exe:0x00481150` |
| `Darkages.exe:0x00481EEC` | `sub_481E60` | `Darkages.exe:0x00481E60` |
| `Darkages.exe:0x0048205B` | `sub_481F00` | `Darkages.exe:0x00481F00` |
| `Darkages.exe:0x00482B5C` | `sub_482AD0` | `Darkages.exe:0x00482AD0` |
| `Darkages.exe:0x00482F1C` | `sub_482B70` | `Darkages.exe:0x00482B70` |

## Schema status

The 4.21 client emits this action at the listed sites. The payload fields and client-side trigger still require tracing.
