# `SSendPatch` (`0x40`)

[Previous: SActionCoolTime](063-3f-action-cool-time.md) | [Server action index](../server-actions.md) | [Next: SExchange](066-42-exchange.md)

`SSendPatch` is server-to-client action `0x40` in the 4.21 protocol.

**Direction:** server to client

**Encrypted:** No. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | `payload_length` | `unknown_00` | Payload bytes whose field boundaries are not yet mapped. |

## Handler functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x004955C0` | `sub_4955C0` | Accepts the action in its pane's Event type 9 packet handler. |

## Handler notes

`Darkages.exe:0x004955C0` `sub_4955C0`. XOR bypass. May be used during login or lobby.

## Schema status

The 4.21 client accepts this action in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.
