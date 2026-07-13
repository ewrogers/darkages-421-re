# `SSelfSaveOk` (`0x21`)

[Previous: SChangeHour](032-20-change-hour.md) | [Server action index](../server-actions.md) | [Next: SActionChange](038-26-action-change.md)

`SSelfSaveOk` is server-to-client action `0x21` in the 4.21 protocol.

**Direction:** server to client

**Encrypted:** Yes. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

This action has no payload fields read by the OptionPane handler.

## Handler functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x004909E0` | `ui_option_pane_handle_server_packet` | Accepts the action in OptionPane's Event type 9 packet handler. |

## Handler notes

`Darkages.exe:0x004909E0` `ui_option_pane_handle_server_packet` compares the decoded action byte to `0x21`. It does not read later bytes.

## UI behavior

While OptionPane is registered, this action allocates and constructs a local message dialog containing "Saved." The packet acknowledges an earlier option save such as `CUserSetting` `0x1B`; it does not create OptionPane itself.

See [UI, Input, and Packet Flows](../../architecture/ui-network-flows.md#server-actions-that-affect-ui).
