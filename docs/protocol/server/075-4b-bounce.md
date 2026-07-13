# `SBounce` (`0x4B`)

[Previous: SRequestPortrait](073-49-request-portrait.md) | [Server action index](../server-actions.md) | [Next: SReconnect](076-4c-reconnect.md)

`SBounce` is the supplied later-client message name for server-direction action `0x4B`. It is not RTTI or a symbol recovered from the 4.21 executable.

**Direction:** server to client

**Encrypted:** Yes. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 2 | `embedded_length` | Unsigned big-endian length of the embedded logical client packet. |
| `0x02` | `embedded_length` | `embedded_client_packet` | Begins with a client-direction action byte and is submitted through `net_c_queue_send`. |
| `0x02 + embedded_length` | variable | `unknown_tail` | Any remaining outer payload bytes; their exact boundary is not yet mapped. |

## Handler functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |
| `Darkages.exe:0x0046CA54` | `net_forward_embedded_client_packet` | Reads the embedded length and submits the embedded client packet. |

## Handler notes

`ui_map_dispatch_server_packet` calls `net_forward_embedded_client_packet`. The helper reads the big-endian embedded length at payload offset `0x00`; the embedded logical client packet begins at payload offset `0x02`.

## Schema status

The later-client `SBounce` name is a cross-version reference. Stone directly establishes the embedded length and forwarding behavior. The server can therefore select a client action dynamically rather than using one of the fixed action constants found in client builders.
