# `SReconnect` (`0x4C`)

[Previous: SBounce](075-4b-bounce.md) | [Server action index](../server-actions.md) | [Next: SBlockInput](081-51-block-input.md)

`SReconnect` is server-to-client action `0x4C` in the 4.21 protocol.

**Direction:** server to client

**Encrypted:** Yes. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `subtype` | Value `0x01` completes the Stone safe-exit exchange. Other values are consumed without the mapped side effects. |
| `0x01` | `payload_length - 1` | `unknown_01` | Any additional payload bytes. |

## Handler functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x004921F0` | `ui_exit_wait_handle_server_packet` | Consumes `0x4C`; subtype `0x01` completes the safe-exit exchange. |
| `Darkages.exe:0x00492310` | `ui_exit_wait_pane_ctor` | Creates the exit-wait pane, shows the warning, and sends `CQuit` subtype `0x01`. |

## Handler notes

`Darkages.exe:0x004921F0` `ui_exit_wait_handle_server_packet`. The Stone handler consumes all `0x4C` packets. Subtype `0x01` sends `CQuit` subtype `0x00` and changes an exit-wait pane to the safe-exit message; it does not call the Socket reconnect path.

## Schema status

`SReconnect` is the 4.21 class name. In the mapped exit-wait path, `ui_exit_wait_handle_server_packet` claims every `0x4C` packet. Subtype `0x01` sets the pane completion flag, sends `CQuit` subtype `0x00`, and replaces the warning with the safe-to-exit text. The paired `ui_exit_wait_pane_ctor` sends `CQuit` subtype `0x01` when the wait pane is created. No Socket connection or reconnect function is called by this handler.
