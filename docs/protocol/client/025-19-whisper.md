# `CWhisper` (`0x19`)

[Previous: CWho](024-18-who.md) | [Client action index](../client-actions.md) | [Next: CUserSetting](027-1b-user-setting.md)

`CWhisper` is client-to-server action `0x19` in the 4.21 protocol.

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
| `Darkages.exe:0x004C1182` | `sub_4C10D0` | `Darkages.exe:0x004C10D0` |

## Schema status

The 4.21 client emits this action at the listed sites. The payload fields and client-side trigger still require tracing.
