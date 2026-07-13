# `SNewUserCheck` (`0x01`)

[Previous: SVersionCheck](000-00-version-check.md) | [Server action index](../server-actions.md) | [Next: SLoginCheck](002-02-login-check.md)

`SNewUserCheck` is the supplied later-client message name for server-direction action `0x01`. It is not RTTI or a symbol recovered from the 4.21 executable.

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
| `Darkages.exe:0x00461080` | `sub_461080` | Accepts the action in its pane's Event type 9 packet handler. |

## Handler notes

`Darkages.exe:0x00461080` `sub_461080`. Later-client scope: login or lobby.

## Schema status

The `SNewUserCheck` name is a later-client cross-version reference. Stone accepts action `0x01` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.
