# `SExchange` (`0x42`)

[Previous: SSendPatch](064-40-send-patch.md) | [Server action index](../server-actions.md) | [Next: SSpellDelayCancel](072-48-spell-delay-cancel.md)

`SExchange` is server-to-client action `0x42` in the 4.21 protocol.

**Direction:** server to client

**Encrypted:** Yes. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `subtype` | Exchange operation. Values 1 through 5 have mapped Stone branches. |
| `0x01` | `payload_length - 1` | `unknown_01` | Subtype-dependent payload bytes whose field boundaries remain to be mapped. |

## Handler functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00435EA0` | `sub_435EA0` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x004374F0` | `sub_4374F0` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x00437C70` | `sub_437C70` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

## Handler notes

`Darkages.exe:0x00435EA0` `sub_435EA0`, `Darkages.exe:0x004374F0` `sub_4374F0`, `Darkages.exe:0x00437C70` `sub_437C70`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. Exchange-session handlers dispatch on a second byte with established subtypes 1 through 5.

## Schema status

The 4.21 exchange-pane handlers establish the subtype byte and branches 1 through 5. Their individual payload schemas remain to be mapped.
