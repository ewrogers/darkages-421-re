# `SShowPaper` (`0x35`)

[Previous: SObjectInfo](052-34-object-info.md) | [Server action index](../server-actions.md) | [Next: SShowUsers](054-36-show-users.md)

`SShowPaper` is server-to-client action `0x35` in the 4.21 protocol.

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
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

## Handler notes

`ui_map_dispatch_server_packet` calls `Darkages.exe:0x0046C3F4` `ui_map_handle_show_paper`. The handler allocates a `0x560`-byte Paper dialog and calls `ui_paper_dialog_ctor` in mode 1. Its parser copies the server text and flags, then `ui_paper_dialog_build_content` adds the dialog below `BackgroundPane` and registers it for Events.

## UI flow

This action creates a new dynamic dialog. It does not reuse a static Paper root. `SEnterEditingMode` `0x1B` uses the same class in constructor mode 0. Paper submission can send `CExitEditingMode` `0x23` with a mode byte, big-endian text length, and normalized text.

See [UI, Input, and Packet Flows](../../architecture/ui-network-flows.md#users-paper-and-server-menus).
