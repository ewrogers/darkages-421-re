# Protocol Catalog

The Stone client uses a direction-sensitive action byte inside an `0xAA` binary frame. This catalog separates client actions from server actions because equal numeric values can have unrelated behavior.

## Transport and framing

The standard network transport is an IPv4 TCP byte stream. Frames have this wire layout:

```text
AA [uint16 body_length, big-endian] [body bytes...]
```

| Offset | Width | Meaning |
|---:|---:|---|
| `0x00` | 1 | `0xAA` frame marker. |
| `0x01` | 2 | Unsigned big-endian body length. |
| `0x03` | 1 | Direction-specific action. |
| `0x04` | variable | Transformation metadata or payload, depending on action. |

`body_length` includes the action and every remaining wire body byte. It excludes the marker and its own two bytes. Total frame length is `body_length + 3`.

Ordinary transformed bodies have this form:

```text
[action] [sequence] [XOR-transformed payload and zero sentinel]
```

The logical packet delivered to or accepted from application code is:

```text
[action] [payload] [zero sentinel]
```

The sequence byte and XOR passes are omitted for client actions `0x00`, `0x10`, and `0x48`, and for server actions `0x00`, `0x03`, and `0x40`. Only transformed client packets consume and increment the client send sequence. See [Sequence and XOR Transformation](xor-transformation.md) for the exact pass order, negotiated key, and generated-table formulas.

## Directional catalogs

| Direction | Fixed action values | Index |
|---|---:|---|
| Client to server | 51 | [Client actions](client-actions.md), with one linked page per action |
| Server to client | 59 | [Server actions](server-actions.md), with one linked page per action |

Thirty-five action values occur in both lists. Direction is part of a packet's identity.

The server action `0x4B` contains a big-endian length followed by a logical client packet. The client forwards that embedded packet through its normal send queue. This permits a server-selected action in addition to the 51 constants present in client packet builders.

## Current interpretation level

The indexes distinguish established action support from payload details that still require tracing:

- Direction-prefixed `C...` and `S...` class names identify messages in the 4.21 protocol catalog.
- Payload fields and behavioral meanings are documented only when instructions, data flow, diagnostics, or visible state changes establish them.
- Builder and handler addresses are included even when individual payload fields remain to be analyzed.

The indexes are exhaustive for fixed action constants accepted or emitted by the traced Stone packet path. They are not claims that every payload schema is already understood.

Direction-specific network-owned IDA names use `net_c_...` for client-to-server code and `net_s_...` for server-to-client code. Direction-neutral transport code retains `net_...`. Methods owned primarily by another subsystem retain that subsystem prefix, such as `ui_...` for pane packet dispatch.
