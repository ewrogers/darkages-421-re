# `SRequestPortrait` (`0x49`)

[Previous: SSpellDelayCancel](072-48-spell-delay-cancel.md) | [Server action index](../server-actions.md) | [Next: SBounce](075-4b-bounce.md)

`SRequestPortrait` is the supplied later-client message name for server-direction action `0x49`. It is not RTTI or a symbol recovered from the 4.21 executable.

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

Both the later-client `SRequestPortrait` reference and the Stone diagnostic `kServerRequestPortrait` identify this action. The handler initiates client action `0x4F`; its request payload, if any beyond the sentinel, remains to be mapped.
