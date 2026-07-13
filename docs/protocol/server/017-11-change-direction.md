# `SChangeDirection` (`0x11`)

[Previous: SRemoveItem](016-10-remove-item.md) | [Server action index](../server-actions.md) | [Next: SDamageEffect](019-13-damage-effect.md)

`SChangeDirection` is server-to-client action `0x11` in the 4.21 protocol.

**Direction:** server to client

**Encrypted:** Yes. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 4 | `object_id` | Unsigned big-endian object identifier. |
| `0x04` | 1 | `direction` | Direction value. The mapped handler applies values 0 through 3. |
| `0x05` | `payload_length - 5` | `unknown_05` | Any additional payload bytes; the mapped handler does not read them. |

## Handler functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |
| `Darkages.exe:0x0046B574` | `ui_map_handle_object_direction` | Reads the object ID and direction and updates the matching map object. |

## Handler notes

`ui_map_dispatch_server_packet` calls `ui_map_handle_object_direction`. The helper reads the object ID at payload offset `0x00` and the direction at payload offset `0x04`.

## Schema status

The handler reads `object_id`, locates the map object, and applies `direction` only when its value is from 0 through 3. This payload prefix is established directly from the 4.21 instructions.
