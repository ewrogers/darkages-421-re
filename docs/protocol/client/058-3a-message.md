# `CMessage` (`0x3A`)

[Previous: CMenuCode](057-39-menu-code.md) | [Client action index](../client-actions.md) | [Next: CBulletin](059-3b-bulletin.md)

`CMessage` is client-to-server action `0x3A` in the 4.21 protocol.

**Direction:** client to server

**Encrypted:** Yes. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `dialog_type` | Type byte retained from the server-created message dialog. |
| `0x01` | 4 | `object_id` | Big-endian object identifier retained by the dialog. |
| `0x05` | 2 | `pursuit_id` | Big-endian server pursuit or interaction identifier. |
| `0x07` | 2 | `next_step` | Big-endian stored step value plus one. |
| `0x09` | 1 | `response_type` | Value `1` in the confirmed question-answer builders. |
| `0x0A` | 1 | `answer` | Selected answer index passed to the dialog's answer virtual. |

## Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x0047E85C` | `sub_47E7D0` | `Darkages.exe:0x0047E7D0` |
| `Darkages.exe:0x0047F9AC` | `sub_47F920` | `Darkages.exe:0x0047F920` |
| `Darkages.exe:0x0047FA89` | `sub_47F9C0` | `Darkages.exe:0x0047F9C0` |
| `Darkages.exe:0x004805CC` | `sub_480540` | `Darkages.exe:0x00480540` |
| `Darkages.exe:0x004806A9` | `sub_4805E0` | `Darkages.exe:0x004805E0` |
| `Darkages.exe:0x0048113C` | `sub_4810B0` | `Darkages.exe:0x004810B0` |
| `Darkages.exe:0x004812AE` | `sub_481150` | `Darkages.exe:0x00481150` |
| `Darkages.exe:0x00481EEC` | `sub_481E60` | `Darkages.exe:0x00481E60` |
| `Darkages.exe:0x0048205B` | `sub_481F00` | `Darkages.exe:0x00481F00` |
| `Darkages.exe:0x00482B5C` | `sub_482AD0` | `Darkages.exe:0x00482AD0` |
| `Darkages.exe:0x00482F1C` | `sub_482B70` | `Darkages.exe:0x00482B70` |

## Confirmed question answer form

`ui_question_message_dialog_send_answer` and `ui_question_message_face_dialog_send_answer` each submit 12 logical bytes including the action, exactly matching the fields above. These dialogs are created by `SPursuitMenu` subtypes `2` and `6`. Other `CMessage` builders use related dialog variants and still require separate field comparison before their additional semantics are named.

See [UI, Input, and Packet Flows](../../architecture/ui-network-flows.md#users-paper-and-server-menus).
