# `SChangeWeather` (`0x1F`)

[Previous: SEnterEditingMode](027-1b-enter-editing-mode.md) | [Server action index](../server-actions.md) | [Next: SChangeHour](032-20-change-hour.md)

`SChangeWeather` is the supplied later-client message name for server-direction action `0x1F`. It is not RTTI or a symbol recovered from the 4.21 executable.

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

`Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`.

## Schema status

The `SChangeWeather` name is a later-client cross-version reference. Stone accepts action `0x1F` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.
