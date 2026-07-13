# `CQuit` (`0x0B`)

[Previous: CDrop](008-08-drop.md) | [Client action index](../client-actions.md) | [Next: CPutGround](012-0c-put-ground.md)

`CQuit` is client-to-server action `0x0B` in the 4.21 protocol.

**Direction:** client to server

**Encrypted:** Yes. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `subtype` | The exit-wait constructor sends `0x01`; the `SReconnect` subtype-one response sends `0x00`. Values used by the other builders are not yet compared. |
| `0x01` | `payload_length - 1` | `unknown_01` | Any additional payload bytes. |

## Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x0046471C` | `sub_4646F0` | `Darkages.exe:0x004646F0` |
| `Darkages.exe:0x004889DC` | `sub_488974` | `Darkages.exe:0x00488974` |
| `Darkages.exe:0x00488E2E` | `sub_488B44` | `Darkages.exe:0x00488B44` |
| `Darkages.exe:0x004922B1` | `ui_exit_wait_handle_server_packet` | `Darkages.exe:0x004921F0` |
| `Darkages.exe:0x0049236B` | `ui_exit_wait_pane_ctor` | `Darkages.exe:0x00492310` |
| `Darkages.exe:0x0049544D` | `sub_495414` | `Darkages.exe:0x00495414` |

## Schema status

The two exit-wait call sites establish the subtype exchange. The remaining four builders still need to be compared before assigning a complete subtype directory.
