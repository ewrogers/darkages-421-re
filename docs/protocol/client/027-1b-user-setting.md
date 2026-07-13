# `CUserSetting` (`0x1B`)

[Previous: CWhisper](025-19-whisper.md) | [Client action index](../client-actions.md) | [Next: CUse](028-1c-use.md)

`CUserSetting` is client-to-server action `0x1B` in the 4.21 protocol.

**Direction:** client to server

**Encrypted:** Yes. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `setting_code` | Option selection code chosen by the active OptionPane-family control. Confirmed callers pass constants `0` through `8` or a control-derived byte. |

## Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x00491AC4` | `net_c_send_user_setting` | `Darkages.exe:0x00491AA4` |

## UI flow

Window button Q creates the dynamic `OptionPane` without sending a packet. Its controls and related option subdialogs call `net_c_send_user_setting`, which submits exactly `[0x1B, setting_code]`. The server can acknowledge a saved selection with `SSelfSaveOk` `0x21`; the registered OptionPane then creates a local "Saved." message dialog.

The individual meanings of setting codes `0` through `8` have not yet been assigned because the current evidence establishes control branches but not their server-side setting names.

See [UI, Input, and Packet Flows](../../architecture/ui-network-flows.md#users-paper-and-server-menus).
