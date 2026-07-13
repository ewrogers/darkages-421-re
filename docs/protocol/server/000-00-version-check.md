# `SVersionCheck` (`0x00`)

[Server action index](../server-actions.md) | [Next: SNewUserCheck](001-01-new-user-check.md)

`SVersionCheck` is the supplied later-client message name for server-direction action `0x00`. It is not RTTI or a symbol recovered from the 4.21 executable.

**Direction:** server to client

**Encrypted:** No. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `subtype` | `0x00` negotiates transformation state, `0x01` reports an outdated client, and `0x02` enters the patch path. |
| `0x01` | 1 | `server_version` | Subtype `0x00`: stored and compared with the active server version. |
| `0x02` | 1 | `table_function` | Subtype `0x00`: queues table function 0 through 9. |
| `0x03` | 1 | `key_length` | Subtype `0x00`: required to equal 9 by the main-menu handler. |
| `0x04` | `key_length` | `key` | Subtype `0x00`: replacement repeating XOR key. |

## Handler functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x0045F780` | `ui_main_menu_handle_server_packet` | Accepts the action in the main-menu packet handler. |
| `Darkages.exe:0x004A3700` | `net_queue_xor_table_function` | Queues the subtype-zero table-function selector as Socket work code 11. |
| `Darkages.exe:0x004A36C0` | `net_queue_xor_key` | Copies and queues the subtype-zero key as Socket work code 10. |

## Handler notes

The subtype-zero branch reads the server version, queues the one-byte table selector, requires a nine-byte key, and queues that key for the socket worker. Action `0x00` bypasses the sequence and XOR decoder, so negotiation does not depend on the previous transformation state.

## Schema status

The subtype-zero payload prefix is established through the field readers and both queue calls. The data consumed by subtypes `0x01` and `0x02` still requires separate tracing.
