# `SRemoveEquip` (`0x38`)

[Previous: SAddEquip](055-37-add-equip.md) | [Server action index](../server-actions.md) | [Next: SSelfLook](057-39-self-look.md)

`SRemoveEquip` is the supplied later-client message name for server-direction action `0x38`. It is not RTTI or a symbol recovered from the 4.21 executable.

**Direction:** server to client

**Encrypted:** Yes. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | `payload_length` | `unknown_00` | Payload bytes whose field boundaries are not yet mapped. |

## Handler functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x0042EAF0` | `sub_42EAF0` | Accepts the action in its pane's Event type 9 packet handler. |

## Handler notes

`Darkages.exe:0x0042EAF0` `sub_42EAF0`.

## Schema status

The `SRemoveEquip` name is a later-client cross-version reference. Stone accepts action `0x38` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.
