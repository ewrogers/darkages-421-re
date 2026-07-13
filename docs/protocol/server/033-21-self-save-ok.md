# `SSelfSaveOk` (`0x21`)

[Previous: SChangeHour](032-20-change-hour.md) | [Server action index](../server-actions.md) | [Next: SActionChange](038-26-action-change.md)

`SSelfSaveOk` is server-to-client action `0x21` in the 4.21 protocol.

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
| `Darkages.exe:0x004909E0` | `sub_4909E0` | Accepts the action in its pane's Event type 9 packet handler. |

## Handler notes

`Darkages.exe:0x004909E0` `sub_4909E0`.

## Schema status

The 4.21 client accepts this action in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.
