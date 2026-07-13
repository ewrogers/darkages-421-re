# `SRemoveItem` (`0x10`)

[Previous: SAddItem](015-0f-add-item.md) | [Server action index](../server-actions.md) | [Next: SChangeDirection](017-11-change-direction.md)

`SRemoveItem` is server-to-client action `0x10` in the 4.21 protocol.

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
| `Darkages.exe:0x00441850` | `sub_441850` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x004889F0` | `ui_local_user_handle_server_packet` | Accepts the action in the local-user Pane's Event type 9 packet handler. |

## Handler notes

`Darkages.exe:0x00441850` `sub_441850`, `Darkages.exe:0x004889F0` `ui_local_user_handle_server_packet`.

## Schema status

The 4.21 client accepts this action in the listed functions. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.
