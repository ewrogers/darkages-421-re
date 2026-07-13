# `SSelfLook` (`0x39`)

[Previous: SRemoveEquip](056-38-remove-equip.md) | [Server action index](../server-actions.md) | [Next: SSpelled](058-3a-spelled.md)

`SSelfLook` is server-to-client action `0x39` in the 4.21 protocol.

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
| `Darkages.exe:0x0042EAF0` | `ui_equip_pane_handle_server_packet` | Accepts the action in its pane's Event type 9 packet handler. |

## Handler notes

`Darkages.exe:0x0042EAF0` `ui_equip_pane_handle_server_packet` calls `ui_equip_pane_apply_self_look`. The parser replaces appearance bytes, several length-prefixed character strings, a big-endian 32-bit value, and embedded legend/detail state, then invalidates the pane.

## UI behavior

This action refreshes the persistent User Equip object but does not show it. Reopening an already selected but hidden equipment pane sends `CSelfLook` `0x2D` and immediately shows the existing object; this response updates it asynchronously.

The static root is `ui_equip_pane` at `Darkages.exe:0x004E32A4`, RVA `0x000E32A4`.

See [UI, Input, and Packet Flows](../../architecture/ui-network-flows.md#equipment-and-self-look).
