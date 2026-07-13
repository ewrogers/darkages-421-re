# `SScreenMenu` (`0x2F`)

[Previous: SFieldMap](046-2e-field-map.md) | [Server action index](../server-actions.md) | [Next: SPursuitMenu](048-30-pursuit-menu.md)

`SScreenMenu` is server-to-client action `0x2F` in the 4.21 protocol.

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
| `Darkages.exe:0x00479480` | `sub_479480` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x0047AFD0` | `sub_47AFD0` | Accepts the action in its pane's Event type 9 packet handler. |

## Handler notes

`Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, `Darkages.exe:0x00479480` `sub_479480`, `Darkages.exe:0x0047AFD0` `sub_47AFD0`.

## Schema status

The 4.21 client accepts this action in the listed functions. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.
