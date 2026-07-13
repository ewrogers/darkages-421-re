# `SChangeHour` (`0x20`)

[Previous: SChangeWeather](031-1f-change-weather.md) | [Server action index](../server-actions.md) | [Next: SSelfSaveOk](033-21-self-save-ok.md)

`SChangeHour` is server-to-client action `0x20` in the 4.21 protocol.

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
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

## Handler notes

`Darkages.exe:0x0043F420` `sub_43F420`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`.

## Schema status

The 4.21 client accepts this action in the listed functions. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.
