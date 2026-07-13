# `SStatus` (`0x08`)

[Previous: SDrawObjects](007-07-draw-objects.md) | [Server action index](../server-actions.md) | [Next: SMessage](010-0a-message.md)

`SStatus` is server-to-client action `0x08` in the 4.21 protocol.

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
| `Darkages.exe:0x0043F420` | `sub_43F420` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x00441850` | `sub_441850` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |
| `Darkages.exe:0x004889F0` | `sub_4889F0` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x004927F0` | `sub_4927F0` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x00494C60` | `sub_494C60` | Accepts the action in its pane's Event type 9 packet handler. |

## Handler notes

`Darkages.exe:0x0043F420` `sub_43F420`, `Darkages.exe:0x00441850` `sub_441850`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, `Darkages.exe:0x004889F0` `sub_4889F0`, `Darkages.exe:0x004927F0` `sub_4927F0`, `Darkages.exe:0x00494C60` `sub_494C60`.

The action is observed by several already registered game panes because status fields are distributed across more than one UI owner.

## UI behavior

`SStatus` updates fields and redraw state but does not select or show the persistent status content pane. Status visibility is a client-only A/S/D/F/G transition. G or the status game button calls `ui_game_buttons_select_status`, hides the previous content pane, and shows the already populated status pane.

See [UI, Input, and Packet Flows](../../architecture/ui-network-flows.md#persistent-content-selection).
