# `SRequestPortrait` (`0x49`)

[Previous: SSpellDelayCancel](072-48-spell-delay-cancel.md) | [Server action index](../server-actions.md) | [Next: SBounce](075-4b-bounce.md)

`SRequestPortrait` is server-to-client action `0x49` in the 4.21 protocol.

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
| `Darkages.exe:0x0040B3A0` | `ui_handle_server_request_portrait` | Handles the portrait request and starts client portrait serialization. |

## Handler notes

`Darkages.exe:0x0040B3A0` `ui_handle_server_request_portrait`. An embedded diagnostic names `kServerRequestPortrait` and states decimal value 73. The handler initiates client action `0x4F`.

## Schema status

The 4.21 diagnostic `kServerRequestPortrait` identifies this action. The handler initiates client action `0x4F`; any request payload remains to be mapped.
