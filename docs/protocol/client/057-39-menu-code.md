# `CMenuCode` (`0x39`)

[Previous: CRefreshUser](056-38-refresh-user.md) | [Client action index](../client-actions.md) | [Next: CMessage](058-3a-message.md)

`CMenuCode` is client-to-server action `0x39` in the 4.21 protocol.

**Direction:** client to server

**Encrypted:** Yes. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `menu_type` | Server-supplied menu or interaction type copied from the active menu pane. |
| `0x01` | 4 | `object_id` | Big-endian server object identifier copied from the active menu pane. |
| `0x05` | 2 | `menu_code` | Big-endian selected menu code or control identifier. |
| `0x07` | variable | `variant_data` | Optional selection byte, length-prefixed text, or class-specific values. Absent in the eight-byte logical form. |

## Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x0047B8CD` | `sub_47B850` | `Darkages.exe:0x0047B850` |
| `Darkages.exe:0x0047BBB7` | `sub_47BB00` | `Darkages.exe:0x0047BB00` |
| `Darkages.exe:0x0047C175` | `sub_47C100` | `Darkages.exe:0x0047C100` |
| `Darkages.exe:0x0047C2CF` | `sub_47C220` | `Darkages.exe:0x0047C220` |
| `Darkages.exe:0x0047C60F` | `sub_47C560` | `Darkages.exe:0x0047C560` |
| `Darkages.exe:0x0047C8C8` | `sub_47C7C0` | `Darkages.exe:0x0047C7C0` |
| `Darkages.exe:0x0047D3BF` | `sub_47D310` | `Darkages.exe:0x0047D310` |
| `Darkages.exe:0x0047D46D` | `sub_47D3E0` | `Darkages.exe:0x0047D3E0` |
| `Darkages.exe:0x0047D7DC` | `sub_47D6E4` | `Darkages.exe:0x0047D6E4` |

## UI flow

All nine traced builders share the seven-byte payload prefix above. They are called by controls owned by the dynamic `SScreenMenu` interaction. The simplest form submits eight logical bytes including the action. Other variants append a one-byte length and text, a selection byte, or three small class-specific values.

See [UI, Input, and Packet Flows](../../architecture/ui-network-flows.md#users-paper-and-server-menus).
