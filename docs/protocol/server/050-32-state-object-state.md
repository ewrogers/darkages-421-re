# `SStateObjectState` (`0x32`)

[Previous: SBulletin](049-31-bulletin.md) | [Server action index](../server-actions.md) | [Next: SDrawHumanObject](051-33-draw-human-object.md)

`SStateObjectState` is server-to-client action `0x32` in the 4.21 protocol.

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
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

## Handler notes

`Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. The mapped branch performs inline tile or object updates.

## Schema status

The 4.21 client accepts this action in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.
