# Event Proxy IPC, Rules, and State

The Event Proxy should keep client hooks independent from the external controller. A duplex IPC service carries commands and telemetry, an in-process ruleset makes immediate decisions, and a typed snapshot service supplies current state after late attach or reconnect.

Everything on this page is a proposed proxy protocol built around confirmed 4.21 client roots and object layouts. It does not describe an IPC facility already present in the game.

## High-level operation

### Pipe ownership and reconnect

The injected DLL should be the named-pipe server. A practical per-process name is:

```text
\\.\pipe\darkages-event-proxy-{process_id}
```

The pipe should reject remote-machine clients and grant access only to the current interactive user unless the operator explicitly configures a broader security descriptor. One connection owns mutation privileges. Additional read-only monitor connections can be added later, but they complicate rule ownership and backpressure and are not required for the first implementation.

The accept loop belongs to the DLL and survives client-controller failure:

1. Create a pipe instance and wait for a controller.
2. Complete a versioned handshake and allocate a new connection generation.
3. Apply subscriptions and session rules only after validation.
4. On clean close, broken pipe, or controller process death, remove session rules and cancel that generation's pending commands.
5. Return immediately to the accept state. Do not unload the DLL or remove the pass-through hooks.

Persistent rules are separate from session rules. They survive pipe disconnect and must be installed with an explicit persistence flag. With no rules, no controller, or an IPC failure, the proxy passes client traffic unchanged.

### Command execution and worker wakes

The pipe service and client command execution are separate concerns. The IPC thread parses and validates a command, copies it into a bounded proxy-owned queue, records the current connection generation, and signals the owning client worker. It never invokes a live client object directly.

| Proxy queue | Consumer | Wake mechanism | Typical commands |
|---|---|---|---|
| Socket | Socket worker periodic pump | `ReleaseSemaphore(Socket->wait_handles[0], 1, NULL)` | `SEND_CLIENT_PACKET` and Socket-owned queries. |
| EventMan | EventMan worker periodic pump | `ReleaseSemaphore(EventMan->wait_handles[0], 1, NULL)` | `INJECT_SERVER_PACKET` and normalized `INJECT_INPUT`. |
| Dispatcher | Dispatcher worker periodic pump | `ReleaseSemaphore(Dispatcher->wait_handles[0], 1, NULL)` | Snapshot barriers, pane actions, timers, and typed UI writes. |

The generic worker explicitly tolerates a work-semaphore wake while its native ring is empty and still calls virtual slot `+0x10`. This lets the DLL wake the correct worker without adding an unknown native work code. The Socket slot already calls `net_poll_receive`, the EventMan slot is a no-op, and the dispatcher slot calls `event_dispatcher_tick`.

Admission holds a proxy lifecycle read guard across the final state check, root and vtable validation, queue copy, and `ReleaseSemaphore`. A failed wake marks the command canceled and reports `wake_failed`; it must not be reported as admitted. At `app_shutdown` entry, the DLL changes to `draining`, rejects new guards, and waits for existing admission guards before the client closes worker handles.

All three proxy queues must be bounded. Admission fails immediately with `proxy_queue_full`; it must not wait for capacity while holding an IPC session lock. A successful command has two distinct milestones:

- `admitted` means the command was copied into the proxy queue and the worker was signaled.
- `executed` means the target pump validated the generation and live root and invoked the client-side action.

Neither status means that TCP delivered a client packet or that a pane accepted an injected server packet. Completion records should carry the original request identifier and may arrive asynchronously after the admission response.

The 4.21 native rings are also synchronized for multiple producers, so a dedicated fallback executor may call the copied-input wrappers safely. Their full-queue wait is infinite, however. This fallback must use one executor, never the pipe thread, and must stop before EventMan and Socket teardown. The worker-affine design avoids a foreign producer being asleep inside a native queue when `app_shutdown` destroys it.

### IPC framing

A small fixed header lets either side skip unknown message types and correlate requests with replies. All proposed IPC integers are little-endian because this is a local Win32 protocol, not the game's network byte order.

| Offset | Width | Field | Meaning |
|---:|---:|---|---|
| `0x00` | 4 | `magic` | ASCII `DAEP`. |
| `0x04` | 2 | `protocol_version` | IPC schema version. |
| `0x06` | 2 | `header_size` | `0x28` for this header. |
| `0x08` | 2 | `message_type` | Command, response, telemetry, or control type. |
| `0x0A` | 2 | `flags` | Type-specific flags. Unknown required flags reject the message. |
| `0x0C` | 8 | `connection_generation` | Changes on every successful controller handshake. |
| `0x14` | 8 | `request_id` | Nonzero on a command and copied to its response. |
| `0x1C` | 8 | `sequence` | Monotonic telemetry or replay sequence, zero when not applicable. |
| `0x24` | 4 | `payload_length` | Bytes after the header. |

The DLL must reject an unsupported version, a header smaller than the required fields, integer overflow in `header_size + payload_length`, and payloads above a configured limit before allocating memory.

### Message families

| Message | Direction | Purpose |
|---|---|---|
| `HELLO` / `HELLO_REPLY` | Both | Negotiate IPC version, build profile, capabilities, controller role, and connection generation. |
| `SUBSCRIBE` | Controller to DLL | Select phases, directions, actions, Event types, origins, and payload inclusion. |
| `EVENT_RECORD` | DLL to controller | Report decoded ingress, central Event dispatch, logical egress, rule decisions, or loss markers. |
| `RULESET_REPLACE` | Controller to DLL | Compile and atomically replace session or persistent rules. |
| `SEND_CLIENT_PACKET` | Controller to DLL | Admit `[client action][payload]` to the Socket proxy queue for worker-affine sending. |
| `INJECT_SERVER_PACKET` | Controller to DLL | Admit `[server action][payload]` to the EventMan proxy queue for worker-affine Event creation. |
| `INJECT_INPUT` | Controller to DLL | Post Win32-faithful input or use an established EventMan input queue. |
| `SNAPSHOT_REQUEST` / `SNAPSHOT_RESPONSE` | Both | Return typed current state with a dispatch sequence boundary. |
| `PEEK_REQUEST` / `PEEK_RESPONSE` | Both | Read a bounded validated memory region or pointer path. |
| `POKE_REQUEST` / `POKE_RESPONSE` | Both | Perform an explicitly enabled, width-checked, compare-before-write mutation. |
| `PING` / `PONG` | Both | Detect a dead peer without coupling hook execution to liveness. |
| `GAP` | DLL to controller | Report telemetry loss or a replay request older than the retained ring. |

A packet command response reports local admission or execution only. The Socket and Winsock send paths do not provide delivery confirmation, so the IPC protocol must not describe either status as network delivery.

### Nonblocking telemetry

Hooks must not write to a pipe, wait on a controller, perform name resolution, or take a lock that the IPC thread can hold. Each hook should create a compact record in a bounded multi-producer queue. The IPC thread drains records, applies subscription payload policy, and writes them to the pipe.

Every record should include:

- global sequence number;
- phase such as `decoded_ingress`, `event_pre_dispatch`, or `logical_egress`;
- origin such as `client`, `controller`, or `local_rule`;
- process and thread identifier;
- high-resolution proxy timestamp;
- client message timestamp when the Event contains one;
- Event type or directional packet action;
- rule generation and matched rule identifier;
- original and modified lengths where applicable;
- copied logical payload when subscribed.

When the queue is full, increment a loss counter and continue the original client call. The next successfully delivered record includes the accumulated loss count, and the IPC thread also emits `GAP`. Blocking the game to preserve telemetry defeats the pass-through requirement.

### Local rules

The controller compiles policy in the DLL once and the hooks evaluate it without an IPC round trip. A useful rule model contains:

| Rule field | Examples |
|---|---|
| Scope | Decoded ingress, central Event, logical egress. |
| Origin mask | Native client, controller injection, local-rule emission. |
| Primary discriminator | Event type, server action, or client action. |
| Payload predicate | Minimum length, byte equality, masked byte equality, bounded integer field. |
| State predicate | Active client phase or validated pane-class presence. Keep state reads small and profile-backed. |
| Action | Pass, block, mirror, bounded byte patch, or enqueue a validated packet template. |
| Lifetime | Session or persistent. |
| Priority and terminal flag | Deterministic ordering and whether later rules may run. |

The hot representation should be immutable. Compile a replacement table on the IPC thread, validate every offset and action, then swap one pointer atomically. The old table is retired only after all hooks that acquired it have exited. This can use a small active-reader counter or an epoch scheme.

Rules must not allocate, parse regular expressions, or perform unbounded string work inside a hook. Payload predicates need a proven minimum length before every read. A patch must preserve the configured maximum length and should create a scratch copy rather than edit the client's source buffer.

Rule-generated packets go to a proxy command queue rather than calling back into a client function while a hook is midway through its own decision. The generated command carries origin `local_rule`, a parent sequence, and a depth. A fixed maximum depth prevents recursive rules from emitting forever.

### Late-connect state synchronization

The controller needs a baseline before it can interpret live deltas. A useful reconnect protocol is:

1. Establish the connection and begin retaining telemetry for the new generation.
2. Request a dispatcher-bound snapshot barrier.
3. At a bounded point between dispatcher work items, record `last_applied_event_sequence` and copy the small typed state roots.
4. Send the snapshot with its sequence and state epoch.
5. Send retained records newer than that sequence.
6. If the retention ring overflowed, send `GAP` and require a new snapshot.

The central Event hook can update `last_applied_event_sequence` after the original dispatch returns. Most pane state changes caused by input and server packets occur during that call. A compact snapshot taken from the dispatcher worker can therefore describe state after a known Event. The capture must still validate pointers and vtables because rendering, window processing, and teardown involve other threads.

Large or not-yet-mapped world state should not be copied without a time budget on the time-critical dispatcher worker. A later implementation can maintain a proxy-owned shadow model from decoded packets or divide large snapshots into stable chunks. The first version should favor small typed UI and character-pane state.

## Runtime state and memory access

### Static roots

All addresses below are for the analyzed image base `0x00400000`. A profile and external controller must use `loaded_module_base + rva`.

| VA | RVA | Current IDA name | State reached |
|---:|---:|---|---|
| `Darkages.exe:0x004E2D70` | `0x000E2D70` | `ui_bulletin_session` | Active bulletin or mail session, child history, active child, and request-wait state. |
| `Darkages.exe:0x004E32A4` | `0x000E32A4` | `ui_equip_pane` | Persistent equipment identifiers, names, self-look fields, and legend child. |
| `Darkages.exe:0x004E32C0` | `0x000E32C0` | `event_manager_instance` | EventMan input state and owned Socket pointer at object `+0x68`. |
| `Darkages.exe:0x004E3564` | `0x000E3564` | `ui_users_dialog_pane` | Reusable users dialog and its owned row/list objects. |
| `Darkages.exe:0x004F51AC` | `0x000F51AC` | `ui_main_menu_pane` | Current main-menu pane, or null outside that phase. |
| `Darkages.exe:0x004F51B0` | `0x000F51B0` | `ui_map_pane` | Current in-game MapPane root. World collections inside it are not yet mapped. |
| `Darkages.exe:0x004F51B4` | `0x000F51B4` | `ui_background_pane` | Current in-game background pane. |
| `Darkages.exe:0x004F51BC` | `0x000F51BC` | `net_socket_instance` | Active Socket object, transport state, framing buffers, connection flag, and transfer gate. |
| `Darkages.exe:0x004F51C8` | `0x000F51C8` | `ui_screen_pane` | Active screen pane and software cursor state. |
| `Darkages.exe:0x004F51CC` | `0x000F51CC` | `ui_screen_registry` | Packed Screen composition tree. |
| `Darkages.exe:0x004F51D0` | `0x000F51D0` | `event_dispatcher` | Packed Event pane tree, capture pane, current timer state, and dispatcher worker. |
| `Darkages.exe:0x004F51DC` | `0x000F51DC` | `app_main_window` | Main `HWND` used for window-faithful input. |
| `Darkages.exe:0x004FA160` | `0x000FA160` | `util_memory_manager_instance` | Client allocator root required when a worker-injected server packet transfers to dispatcher-owned cleanup. |
| `Darkages.exe:0x004FD640` | `0x000FD640` | `ui_terminal_pane` | Active terminal pane. |

The complete Screen and Event tree algorithms and current class fields are in [Runtime UI Memory Map](../appendices/runtime-ui-memory-map.md).

### Current typed snapshot coverage

| State family | Current confidence and coverage |
|---|---|
| Active pane tree and UI state | High. Screen and Event trees, common bounds, relative origin, visibility, and known pane vtables are mapped. |
| Equipment and self-look | High for the documented fields, 13 equipment slots, item names, appearance bytes, and bounded strings. |
| Skills and spells | High for both 36-pointer parent arrays, populated slot objects, names, identifiers, type, slot, and cooldown state. |
| GameButtons content | High. Current content and persistent chat, status, equipment, skill, and spell pane pointers are mapped. |
| Bulletin and mail | High for session state and active child ownership. Child-class field schemas vary and are only partly mapped. |
| Users dialog | Partial. Pane visibility and owned list objects are mapped; row record internals are not. |
| Character statistics and ordinary item inventory | Not yet mapped to the publication threshold. The relevant panes can be found, but typed fields require another reversing pass. |
| Map tiles, characters, monsters, and world objects | Not yet mapped to the publication threshold beyond the `ui_map_pane` root and packet handlers. |
| Network transport | High for Socket root, connection state, frame state, buffers, transfer gate, sequence, key, and XOR table. |

The proxy protocol should expose capability bits per snapshot family. It must not return guessed fields just because an object can be reached. A raw peek can still return bytes from a validated address, but the response should label them untyped.

### Pointer-path reads

IPC requests should avoid controller-supplied absolute addresses where a stable descriptor is possible. A pointer path can contain:

1. a named profile root or module RVA;
2. whether to dereference it;
3. a bounded signed offset;
4. an expected vtable or readable region class;
5. another dereference step if required;
6. final byte count and scalar or string interpretation.

For example, a typed path can start at `ui_equip_pane`, dereference the root, validate `ui_equip_pane_vtable`, then read the item identifier expression for a checked slot. GameButtons is reached by scanning a stable Screen or Event tree snapshot for its vtable, then following the mapped content fields.

Before every dereference, validate the memory range and overflow, and confirm class vtables where available. After copying class-specific state, re-read both the root that led to it and its vtable. Discard and retry if either changed. Packed Screen nodes use stride `0x3F` and Event nodes use `0x0F`; they must be parsed with unaligned-safe reads rather than native structure casts.

### Peek and poke policy

`PEEK_REQUEST` can support a named pointer path and an explicitly enabled raw-address mode. Bound each request, validate every page as readable, and return the resolved path, bytes, and profile identity. A raw address is meaningful only for the current process and build.

`POKE_REQUEST` should be disabled by default. A safe typed write includes:

- named field and exact width;
- expected root and vtable;
- expected old bytes for compare-before-write;
- allowed value range or bit mask;
- execution context, normally the dispatcher command pump for pane state;
- a response containing whether the compare and write succeeded.

Do not permit ordinary writes to vtables, function pointers, packed-list metadata, owned pointers, executable pages, or object lifetime fields. Writing a Pane visibility byte alone is not a valid show or hide operation because Screen composition, Event eligibility, capture, timers, and owner pointers must remain consistent. Prefer an established UI method, input event, or server packet for stateful changes. An explicit unsafe developer mode may offer raw writes, but it should be clearly separate from the typed API and never enabled by a controller handshake alone.

## Code-level flow

### Hot-hook rule evaluation

```text
hook entry
  -> increment active hook count
  -> acquire immutable rule pointer
  -> classify fixed fields and validate payload bounds
  -> evaluate rules in priority order
  -> try to append telemetry record
  -> release rule pointer
  -> pass, block, or call original with a scratch copy
  -> decrement active hook count
```

The IPC thread performs ruleset parsing and compilation. It does not hold the active rules pointer while waiting on pipe I/O. A ruleset replacement response is sent only after the new table has been validated and published.

### Stable typed snapshot

```text
controller snapshot request
  -> proxy queues dispatcher barrier command
  -> signal dispatcher wait handle 0
  -> dispatcher tick pump drains bounded command
      -> record last applied Event sequence
      -> read roots and expected vtables
      -> copy packed registry metadata and nodes
      -> copy mapped class-specific fields
      -> re-read roots, metadata, and vtables
      -> retry or mark one family unstable
  -> IPC thread serializes snapshot
  -> replay retained telemetry newer than snapshot sequence
```

One unstable family should not invalidate unrelated stable families. The response can mark equipment stable while reporting that a bulletin child changed during capture.

### Controller disconnect

```text
pipe read or write reports disconnect
  -> invalidate connection generation
  -> cancel pending commands from that generation
  -> atomically remove session rules
  -> retain explicitly persistent rules
  -> release queued controller responses
  -> create the next pipe instance
  -> continue client pass-through
```

No hook waits for this cleanup. A command executor checks the generation again immediately before any mutation so a queued command cannot run after its owner disconnected.

## Function table

| Address | Current IDA name | Prototype | Purpose | Call relationships and notes |
|---:|---|---|---|---|
| `Darkages.exe:0x004315B0` | `event_dispatcher_tick` | `void __thiscall(void *event_dispatcher_object)` | Periodic dispatcher-worker entry. | Proposed bounded snapshot barrier and pane-command pump location. |
| `Darkages.exe:0x00431B84` | `event_dispatch` | `int __thiscall(void *event_dispatcher_object, void *event)` | Deliver one logical Event. | Proxy can establish an applied Event sequence around the original call. |
| `Darkages.exe:0x00432E50` | `event_post_socket_bytes` | `void __thiscall(void *event_manager_object, const uint8_t *packet, int length)` | Copy decoded ingress into EventMan. | Native ingress hook and synchronized, blocking fallback for `INJECT_SERVER_PACKET`. |
| `Darkages.exe:0x00434080` | `event_manager_periodic_noop` | `void __thiscall(void *event_manager_object)` | EventMan periodic worker callback. | Native no-op vtable slot used by the proposed EventMan command pump. |
| `Darkages.exe:0x0043CF20` | `ui_game_buttons_handle_key_event` | `int __thiscall(void *game_buttons_pane, void *event)` | Select one persistent game content pane from keyboard input. | Evidence that input injection should prefer normal Event routing over pointer writes. |
| `Darkages.exe:0x0043D164` | `ui_game_buttons_select_content` | `void __thiscall(void *game_buttons_pane, int8_t index)` | Hide the old persistent content pane and show the selected one. | Candidate typed action only when the pane and thread context are validated. |
| `Darkages.exe:0x004594A0` | `ui_hierarchy_get_node` | `void *__thiscall(void *hierarchy, int index)` | Compute a packed node address from list stride and index. | Confirms unaligned Screen and Event node traversal. |
| `Darkages.exe:0x00492C40` | `ui_pane_show` | `void __thiscall(void *pane)` | Show and invalidate a Pane. | Prefer this mapped method over writing only the common visibility byte. |
| `Darkages.exe:0x00492D50` | `ui_pane_hide` | `void __thiscall(void *pane)` | Hide and invalidate a Pane. | Must still be coordinated with owning-pane selection state. |
| `Darkages.exe:0x004A3570` | `net_c_queue_send` | `void __thiscall(void *socket_object, const uint8_t *packet, int16_t length)` | Copy and queue logical client egress. | Native egress hook and synchronized, blocking fallback for `SEND_CLIENT_PACKET`; returns no send result. |
| `Darkages.exe:0x004A39C0` | `net_poll_receive` | `void __thiscall(void *socket_object)` | Socket periodic worker callback. | Recommended Socket command-pump detour; call the original before a bounded proxy drain. |
| `Darkages.exe:0x004BF440` | `util_thread_queue_post_async` | `void __thiscall(void *worker_object, int code, void *data, int value)` | Append one asynchronous worker record. | Native ring access is synchronized but can block for capacity; raw use does not establish data ownership. |
