# `SSendPatch` (`0x40`)

[Previous: SActionCoolTime](063-3f-action-cool-time.md) | [Server action index](../server-actions.md) | [Next: SExchange](066-42-exchange.md)

`SSendPatch` is the supplied later-client message name for server-direction action `0x40`. It is not RTTI or a symbol recovered from the 4.21 executable.

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

`Darkages.exe:0x004955C0` `sub_4955C0`. XOR bypass. The later-client scope may be login or lobby.

## Schema status

The `SSendPatch` name is a later-client cross-version reference. Stone accepts action `0x40` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.
