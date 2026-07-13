# `SMove` (`0x0B`)

[Previous: SMessage](010-0a-message.md) | [Server action index](../server-actions.md) | [Next: SMoveObject](012-0c-move-object.md)

`SMove` is server-to-client action `0x0B` in the 4.21 protocol.

**Direction:** server to client

**Encrypted:** Yes. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `direction` | Values 0 through 3 acknowledge ordinary directions. Value 4 selects the position-correction path. |
| `0x01` | 2 | `x` | Big-endian 16-bit map coordinate, sign-extended by the handler. Used when `direction` is 4. |
| `0x03` | 2 | `y` | Big-endian 16-bit map coordinate, sign-extended by the handler. Used when `direction` is 4. |
| `0x05` | 4 | `unknown_05` | Present between the mapped coordinate fields and final byte. This handler does not read it. |
| `0x09` | 1 | `unknown_09` | Read as an unsigned byte and discarded. It is not compared with the client's movement sequence. |
| `0x0A` | `payload_length - 0x0A` | `unknown_0A` | Any additional payload bytes; the mapped local-user handler does not read them. |

## Handler functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x004889F0` | `ui_local_user_handle_server_packet` | Accepts action `0x0B` in the local-user Pane's Event type 9 handler. |
| `Darkages.exe:0x004897C4` | `ui_local_user_handle_smove` | Parses the payload, updates movement timing, and applies acknowledgment or correction behavior. |

## Handler notes

`ui_local_user_handle_server_packet` dispatches action `0x0B` to `ui_local_user_handle_smove`. The handler does not perform a decoded-size check before reading through payload offset `0x09`; transport framing and upstream event construction are expected to provide the bytes.

For `direction` 0 through 3, the handler checks that the value matches the local user's current direction, updates the lag indicator, and completes its movement acknowledgment path. For value 4, it compares `x` and `y` with local position fields. A mismatch enters the position-correction path before the same lag update. Other direction values trigger the client's diagnostic assertion and invalid-direction logging path.

Timing is recorded before the direction branch. The handler copies the current client movement sequence from local user `+0x6F28` to `+0x6F29`, then stores this unsigned difference at `+0x6F34`:

```c
uint32_t latest_move_response_ms = timeGetTime() - last_move_send_tick;
```

It calls `ui_lag_indicator_update_from_local_user`, which averages the sample with the previously displayed value. The client does not associate an SMove with a retained per-sequence send timestamp. See [Lag indicator and movement timing](../../appendices/runtime-ui-memory-map.md#lag-indicator-and-movement-timing) for the memory fields and color bands.

## Schema status

The ten-byte payload prefix read or skipped by this handler is established. The meanings of bytes `0x05` through `0x09` remain unknown because the mapped local-user handler either skips or discards them. This handler alone does not establish that offset `0x09` is the final payload byte.
