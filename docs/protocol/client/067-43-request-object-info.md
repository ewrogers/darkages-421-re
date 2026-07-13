# `CRequestObjectInfo` (`0x43`)

[Previous: CException](066-42-exception.md) | [Client action index](../client-actions.md) | [Next: CRemoveEquip](068-44-remove-equip.md)

`CRequestObjectInfo` is client-to-server action `0x43` in the 4.21 protocol.

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
| `Darkages.exe:0x0047B755` | `sub_47B704` | `Darkages.exe:0x0047B704` |
| `Darkages.exe:0x00488723` | `sub_4886A0` | `Darkages.exe:0x004886A0` |
| `Darkages.exe:0x0048C820` | `sub_48C774` | `Darkages.exe:0x0048C774` |
| `Darkages.exe:0x0048F17F` | `sub_48F114` | `Darkages.exe:0x0048F114` |

## Schema status

The 4.21 client emits this action at the listed sites. The payload fields and client-side trigger still require tracing.
