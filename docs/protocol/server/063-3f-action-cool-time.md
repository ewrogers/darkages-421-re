# `SActionCoolTime` (`0x3F`)

[Previous: SWindowChange](062-3e-window-change.md) | [Server action index](../server-actions.md) | [Next: SSendPatch](064-40-send-patch.md)

`SActionCoolTime` is server-to-client action `0x3F` in the 4.21 protocol.

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
| `Darkages.exe:0x00442990` | `ui_skill_inventory_handle_server_packet` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x00443B10` | `ui_spell_inventory_handle_server_packet` | Accepts the action in its pane's Event type 9 packet handler. |

## Handler notes

`Darkages.exe:0x00442990` `ui_skill_inventory_handle_server_packet`, `Darkages.exe:0x00443B10` `ui_spell_inventory_handle_server_packet`.

## Schema status

The 4.21 client accepts this action in the listed functions. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.
