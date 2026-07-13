# `SLevelPoint` (`0x3D`)

[Previous: SMapPart](060-3c-map-part.md) | [Server action index](../server-actions.md) | [Next: SWindowChange](062-3e-window-change.md)

`SLevelPoint` is the supplied later-client message name for server-direction action `0x3D`. It is not RTTI or a symbol recovered from the 4.21 executable.

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
| `Darkages.exe:0x0043F420` | `sub_43F420` | Accepts the action in its pane's Event type 9 packet handler. |

## Handler notes

`Darkages.exe:0x0043F420` `sub_43F420`.

## Schema status

The `SLevelPoint` name is a later-client cross-version reference. Stone accepts action `0x3D` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.
