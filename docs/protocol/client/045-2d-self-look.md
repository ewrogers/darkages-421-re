# `CSelfLook` (`0x2D`)

[Previous: CGiveGold](042-2a-give-gold.md) | [Client action index](../client-actions.md) | [Next: CGroup](046-2e-group.md)

`CSelfLook` is client-to-server action `0x2D` in the 4.21 protocol.

**Direction:** client to server

**Encrypted:** Yes. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

`CSelfLook` has no payload. `Darkages.exe:0x0043D4C4` `net_c_send_self_look` submits exactly one logical byte, the action.

## Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x0043D4DC` | `net_c_send_self_look` | `Darkages.exe:0x0043D4C4` |

## UI flow

The inventory/equipment selector sends this action only when the equipment pane is already the current content pane but has been hidden by selecting it twice. It sends `CSelfLook` before showing the pane again. Switching to equipment from another A/S/D/F/G content pane does not send it. `SSelfLook` `0x39` refreshes fields asynchronously and does not control visibility.

See [UI, Input, and Packet Flows](../../architecture/ui-network-flows.md#equipment-and-self-look).
