# `CExitEditingMode` (`0x23`)

[Previous: CEmotion](029-1d-emotion.md) | [Client action index](../client-actions.md) | [Next: CDropGold](036-24-drop-gold.md)

`CExitEditingMode` is client-to-server action `0x23` in the 4.21 protocol.

**Direction:** client to server

**Encrypted:** Yes. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `paper_mode` | Mode byte stored at Paper object `+0x554`. |
| `0x01` | 2 | `text_length` | Big-endian number of following text bytes. |
| `0x03` | `text_length` | `text` | Paper text bytes without a terminator. Carriage returns are changed to tab bytes before submission. |

## Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x00493C99` | `ui_paper_dialog_handle_completion` | `Darkages.exe:0x00493B60` |
| `Darkages.exe:0x0049421C` | `net_c_send_exit_editing_mode` | `Darkages.exe:0x00494100` |

## UI flow

Both builders read the Paper text control, limit the local copy to `0x1F40` bytes, normalize carriage returns, and queue `text_length + 4` logical bytes including the action. `ui_paper_dialog_handle_completion` performs this submission when its completion conditions allow it, then closes the Paper dialog. `net_c_send_exit_editing_mode` performs the same serialization without owning the close transition.

See [UI, Input, and Packet Flows](../../architecture/ui-network-flows.md#users-paper-and-server-menus).
