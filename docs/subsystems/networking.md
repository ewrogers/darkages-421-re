# Networking

The networking subsystem covers socket setup, connection states, receive and send buffering, packet framing, message dispatch, serialization, timeouts, disconnection, and byte transformations.

## High-level operation

This section will explain the lifetime of a connection and how bytes move between the socket, packet layer, message handlers, game state, and user interface. Separate topics will cover transport, framing, serialization, dispatch, connection state, error handling, and XOR or other transformations.

Networking behavior has not been mapped yet. Exact message layouts will be linked from the [Protocol Catalog](../protocol/overview.md).

## Code-level flow

This section will trace socket creation, connection setup, partial receives and sends, buffer management, frame extraction, opcode dispatch, field readers and writers, transformation order, malformed-message handling, disconnect paths, and cleanup.

No networking call tree is documented yet.

## Function table

| Address | Current IDA name | Prototype | Purpose | Call relationships and notes |
|---:|---|---|---|---|
| | | | No functions documented yet | |
