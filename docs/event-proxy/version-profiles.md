# Event Proxy Version Profiles

An Event Proxy should treat every executable build as a profile, even when later clients descend from the same source tree. Function signatures can relocate well-understood symbols across nearby builds, but they cannot by themselves prove object field offsets, allocation sizes, ownership, or calling conventions.

The recommended strategy for 5.x and 7.x is to reverse one representative executable from each build family, establish the same lifecycle, Event, Socket, pane, and state roots, and generate signatures from that ground truth. Signature remapping can then handle close patch revisions. It should not replace the first semantic analysis of a major client family.

## High-level operation

### Exact profiles first

The resolver should fingerprint the loaded image before it installs any hook. The strongest identity is a hash of the file or its mapped immutable sections. PE timestamp, image size, section sizes, and version resources are useful secondary properties but are not unique enough alone.

The analyzed sample has SHA-256 `36093dc1572521ea8c4c5f25548068c971a3d41c7722ef9fe0c05c94e24d6979`. Its project identification is 4.21 Stone, while its Windows version resource reports `360, 0, 0, 0`; the mismatch is documented in [Analyzed Binary](../appendices/binary-identity.md).

For an exact match, the profile supplies RVAs, expected instructions, vtables, globals, calling conventions, object fields, and capability-specific validators. The runtime address is always:

```text
loaded_module_base + profile_rva
```

Do not store a process-specific absolute address in controller configuration.

### Dark Ages 4.21 core profile

| Symbol | VA | RVA | Profile role |
|---|---:|---:|---|
| `app_shutdown` | `Darkages.exe:0x0045B8F0` | `0x0005B8F0` | Draining and detach boundary. |
| `app_initialize` | `Darkages.exe:0x0045CCA0` | `0x0005CCA0` | Early-injection readiness boundary. |
| `event_dispatcher_queue_event_copy` | `Darkages.exe:0x00431200` | `0x00031200` | Copy a 36-byte Event and post dispatcher work code 3. |
| `event_dispatcher_tick` | `Darkages.exe:0x004315B0` | `0x000315B0` | Dispatcher-thread command pump. |
| `event_dispatch` | `Darkages.exe:0x00431B84` | `0x00031B84` | Central Event hook. |
| `event_dispatch_hierarchy` | `Darkages.exe:0x00431D54` | `0x00031D54` | Recursive pane delivery validator. |
| `event_post_socket_bytes` | `Darkages.exe:0x00432E50` | `0x00032E50` | Decoded server packet hook and injection. |
| `event_manager_queue_event_copy` | `Darkages.exe:0x00432F10` | `0x00032F10` | Optional raw Event injection through EventMan. |
| `event_process_work_item` | `Darkages.exe:0x00433110` | `0x00033110` | EventMan worker-side packet and input adapter. |
| `event_manager_periodic_noop` | `Darkages.exe:0x00434080` | `0x00034080` | EventMan worker command-pump slot. |
| `util_memory_manager_alloc` | `Darkages.exe:0x00470FC0` | `0x00070FC0` | Client-owned inbound packet allocation. |
| `net_c_queue_send` | `Darkages.exe:0x004A3570` | `0x000A3570` | Logical client packet hook and native-queue fallback. |
| `net_poll_receive` | `Darkages.exe:0x004A39C0` | `0x000A39C0` | Socket periodic callback and command-pump hook. |
| `net_process_work_item` | `Darkages.exe:0x004A54D0` | `0x000A54D0` | Socket work-code validator. |
| `net_c_send_packet_body` | `Darkages.exe:0x004A72B4` | `0x000A72B4` | Transform, frame, and send validator. |
| `event_manager_instance` | `Darkages.exe:0x004E32C0` | `0x000E32C0` | EventMan static root. |
| `net_socket_instance` | `Darkages.exe:0x004F51BC` | `0x000F51BC` | Socket static root. |
| `ui_screen_registry` | `Darkages.exe:0x004F51CC` | `0x000F51CC` | Screen tree static root. |
| `event_dispatcher` | `Darkages.exe:0x004F51D0` | `0x000F51D0` | Dispatcher static root. |
| `app_main_window` | `Darkages.exe:0x004F51DC` | `0x000F51DC` | Main `HWND` root. |
| `util_memory_manager_instance` | `Darkages.exe:0x004FA160` | `0x000FA160` | Client allocator static root. |
| `event_dispatcher_vtable` | `Darkages.exe:0x0050D180` | `0x0010D180` | Dispatcher class validator. |
| `event_manager_vtable` | `Darkages.exe:0x0050D720` | `0x0010D720` | EventMan class validator. |
| `event_vtable` | `Darkages.exe:0x0050D740` | `0x0010D740` | Raw Event object validator. |
| `net_socket_vtable` | `Darkages.exe:0x00526940` | `0x00126940` | Socket class validator. |

This table is the core hook profile, not the complete memory profile. UI vtables, roots, and fields remain in [Runtime UI Memory Map](../appendices/runtime-ui-memory-map.md) and [Pane Virtual Table Inventory](../appendices/ui-pane-vtables.md).

### What a profile must contain

A useful profile contains more than symbol addresses:

| Profile section | Required information |
|---|---|
| Image identity | Hash, section layout, preferred base, version metadata, and immutable-section hashes. |
| Functions | RVA, prototype, calling convention, minimum detour length, expected normalized instructions, and semantic validator identifier. |
| Globals | RVA, type, lifecycle, nullable phases, expected publisher and clearer functions. |
| Vtables | RVA, slot count used by the proxy, required slot targets, and class name confidence. |
| Object layouts | Size where established, field offsets and widths, ownership, discriminator, valid ranges, worker wait handle 0, and native ring pointer. |
| Packet and Event layouts | Direction, type or action field, payload representation, owned pointers, and cleanup function. |
| Capabilities | Required symbols and validators for each hook, command, snapshot family, peek path, or typed poke. |

A controller should receive the active profile identifier during `HELLO_REPLY` and repeat it in diagnostic logs and captured metadata. This prevents a memory path from one build being silently applied to another.

### Signature scanning

A function signature is useful only when it survives relocations and still has enough structure to be unique. Generate signatures from decoded instructions, not by copying a long unexamined byte range.

The scanner should:

1. Restrict the search to the expected PE section and executable or read-only data class.
2. Wildcard absolute image addresses, import pointers, relative-call displacements, jump-table addresses, and other relocation-sensitive operands.
3. Preserve stable opcodes, constants, stack cleanup, field displacements, queue codes, and branch relationships when they are part of the established contract.
4. Require a unique candidate before semantic validation.
5. Decode the candidate and verify its control flow, callees, globals, object fields, and ownership behavior.
6. Resolve related symbols as a group and verify their cross-references.

A generic function prologue such as register saves and stack allocation is not a signature. A source filename or nearby assertion string is a useful anchor but is not proof. The final candidate must perform the mapped data flow.

### Semantic validators for the core hooks

| Symbol | Required behavior to validate in an unknown build |
|---|---|
| `event_post_socket_bytes` | Reject nonpositive length, allocate `length + 1`, copy bytes, append zero, and post EventMan work code `0x0E`. |
| `event_dispatch` | Read the dispatcher's pane hierarchy, honor capture state, classify mouse, keyboard, and socket Events, and call the recursive hierarchy function. |
| `event_dispatcher_queue_event_copy` | Allocate the established Event size, shallow-copy the complete Event, and post dispatcher work code 3. |
| `net_c_queue_send` | Accept a pointer and signed length, enforce the transfer exception for action `0x10`, allocate `length + 1`, append zero, and post Socket work code 5. |
| `net_process_work_item` | Route code 5 to the logical body sender and free the queued packet copy afterward. |
| `net_poll_receive` and Socket vtable | Occupy periodic slot `+0x10`, run on the Socket worker after every wake, and poll the configured receive path. |
| `event_manager_periodic_noop` and EventMan vtable | Occupy periodic slot `+0x10` and return without native work. |
| `util_thread_queue_worker_loop` | Treat an empty wait-handle-0 wake as valid and still call periodic slot `+0x10`. |
| `util_ring_buffer_push_wait` | Lock the queue monitor, wait on not-full at capacity, copy one fixed-size element, signal not-empty, and unlock. |
| `event_manager_ctor` and roots | Allocate or construct the Socket at EventMan `+0x68`, publish EventMan and Socket globals, and clear them during destruction. |
| `app_shutdown` | Delete EventMan before final dispatcher destruction; EventMan's destructor deletes its Socket and clears both static roots. |

Validation should also confirm the expected vtable at each live root. A matching function body with a mismatched owner layout is not enough to enable mutation.

### Capability gating

Resolve capabilities independently, then expose only the ones whose complete dependency set passed:

| Capability | Minimum validated dependencies |
|---|---|
| Decoded ingress observation and block | `event_post_socket_bytes`, EventMan root and vtable, safe detour bytes. |
| Central Event observation and block | `event_dispatch`, Event layout, dispatcher root and vtable, caller cleanup. |
| Client packet observation | `net_c_queue_send`, Socket root and vtable, signed length and transfer-gate fields. |
| Worker-affine client packet send | Socket root and vtable, `net_poll_receive`, `net_c_send_packet_body`, wait handle 0, signed length, and transfer-gate fields. |
| Worker-affine server packet injection | EventMan root and vtable, periodic slot, `event_process_work_item`, memory-manager root and allocator, packet ownership, and wait handle 0. |
| Window-faithful input | Main `HWND`, window procedure behavior, message schema. |
| Internal input injection | EventMan root plus each requested input queue function and normalized scan-code contract. |
| Typed UI snapshot | Root or tree walker, vtable, every requested field offset, stable-copy rules. |
| Typed poke | All snapshot dependencies plus an established mutation method or safe scalar field contract. |

If a required mutation dependency fails, disable that capability. Do not fall back to an unvalidated nearest address. Passive build diagnostics can remain available without client hooks.

## Porting to 5.x and 7.x

### Why one-time reversing is still required

Shared source lineage is valuable because queue codes, class roles, packet actions, assertion text, vtable shapes, and ownership patterns may remain recognizable. It does not guarantee stable machine code or memory layouts. These changes can invalidate a 4.21 profile while leaving the game visibly similar:

- compiler or optimization changes;
- inserted class fields that shift every later offset;
- new virtual methods that shift vtable slots;
- inlining or outlining of queue wrappers;
- different static initialization and linker ordering;
- larger packet, map, UI, or transport buffers;
- changed ownership or thread-affinity rules;
- security additions around packet processing.

Function scanning also does not find heap fields automatically. Character statistics, inventory, and world-object walking require profile-specific owner roots, object discriminators, container layouts, and deletion rules. These should be established once for each family and then compared structurally with 4.21.

### Recommended porting order

1. Identify the exact executable and record its hash without adding it to the repository.
2. Map WinMain, `app_initialize`, the message procedure, and `app_shutdown`.
3. Locate the generic worker queue and distinguish the dispatcher, EventMan, and Socket objects.
4. Trace decoded receive bytes into pane delivery and packet builders into the logical send queue.
5. Establish Event type layout, Event cleanup, pane handler vslots, and timer delivery.
6. Find EventMan, Socket, dispatcher, Screen, MapPane, and persistent character-pane roots.
7. Map state fields from packet handlers and multiple callers, not from string proximity.
8. Update IDA names, types, comments, and vtable slots.
9. Produce an exact profile and instruction-aware signatures from the established functions.
10. Run passive telemetry first. Enable blocking, injection, and writes only after runtime observations agree with the static model.

Start with one representative 5.x build and one representative 7.x build. If several patch revisions differ only slightly, signatures and semantic validators can likely cover them. If allocation sizes, vtable layouts, or packet ownership change, split them into separate profiles even when the major version is the same.

### Comparing profiles

Use semantic identifiers across profiles while storing version-specific details beneath them. For example, all profiles can export `logical_client_send`, but each entry supplies its own RVA, ABI, transfer-gate offset, sentinel rule, and validators. The IPC command stays stable while the injected adapter changes by profile.

For memory, compare paths rather than absolute offsets alone:

```text
semantic root -> owner relationship -> container kind -> element discriminator -> field
```

If `EventMan -> owned Socket` remains true but the owner offset changes, the profile records the new offset. If a later client moves the Socket to another singleton, the semantic path changes and must be re-established rather than forced into the old shape.

## Code-level flow

### Resolver flow

```text
load image metadata
  -> exact immutable-section hash match
      -> load exact profile
  -> otherwise scan instruction-aware signatures
      -> collect unique candidates
      -> run per-symbol semantic validators
      -> validate roots, vtables, and cross-references
      -> build a temporary resolved profile
  -> evaluate capability dependency sets
  -> install only fully validated capabilities
```

The temporary resolved profile should be exportable as diagnostic data, but it should not be promoted to a maintained profile until the same addresses and layouts are confirmed in IDA and, where needed, by controlled runtime observation.

### Normalized instruction matching

For matching purposes, represent each instruction by opcode class and stable operand properties. Relative targets can be expressed as "call to candidate with validator X" rather than a fixed displacement. Absolute globals can be expressed as "writable image address published by constructor Y." This is more durable than a byte mask alone and permits cross-symbol validation.

A byte mask can remain the fast first stage. The instruction model is the proof stage before hook installation.

### Field-offset recovery

Do not copy 4.21 offsets into an unknown profile. Recover offsets from established functions and require agreement across independent uses. Examples include:

- Event type reads in all three classifier functions;
- packet pointer and size writes in socket Event construction and reads in dispatcher cleanup;
- Socket transfer-gate reads in logical send queueing;
- Socket connection flag writes in connect, receive-zero, and disconnect paths;
- Pane visibility reads, show writes, and hide writes;
- parent slot-array reads in add, remove, destructor, and activation paths.

Only publish a field after its owner, width, signedness, and lifecycle agree across the relevant sites.

## Function table

| Address | Current IDA name | Prototype | Purpose | Call relationships and notes |
|---:|---|---|---|---|
| `Darkages.exe:0x00431200` | `event_dispatcher_queue_event_copy` | `void __thiscall(void *event_dispatcher_object, const void *event)` | Copy one established Event into dispatcher work. | Signature validator includes 36-byte copy and work code 3. |
| `Darkages.exe:0x00431B84` | `event_dispatch` | `int __thiscall(void *event_dispatcher_object, void *event)` | Central logical Event routing. | Signature validator includes capture, hierarchy, and three Event category tests. |
| `Darkages.exe:0x00432E50` | `event_post_socket_bytes` | `void __thiscall(void *event_manager_object, const uint8_t *packet, int length)` | Copy decoded server bytes into EventMan work. | Signature validator includes `length + 1`, zero sentinel, and code `0x0E`. |
| `Darkages.exe:0x00432630` | `event_manager_ctor` | `void *__thiscall(void *event_manager_object)` | Construct EventMan and its owned Socket. | Allocates the `0x780D4`-byte Socket, stores it at `+0x68`, and publishes both roots in 4.21. |
| `Darkages.exe:0x00433110` | `event_process_work_item` | `void __thiscall(void *event_manager_object, int code, void *data, int value)` | Execute EventMan worker records. | Code `0x0E` transfers a client-allocated decoded packet into Event type 9. |
| `Darkages.exe:0x00434080` | `event_manager_periodic_noop` | `void __thiscall(void *event_manager_object)` | Native EventMan periodic callback. | Confirms a no-op `+0x10` worker slot for a profile-gated command pump. |
| `Darkages.exe:0x0045B8F0` | `app_shutdown` | `void __cdecl(void)` | Destroy global subsystems. | Cross-profile lifecycle and detach anchor. |
| `Darkages.exe:0x0045CCA0` | `app_initialize` | `void __cdecl(void)` | Build the main client subsystem graph. | Cross-profile allocation and publication anchor. |
| `Darkages.exe:0x00470FC0` | `util_memory_manager_alloc` | `void *__thiscall(void *memory_manager, int size)` | Allocate client-owned storage. | Required for EventMan worker injection whose payload reaches dispatcher cleanup. |
| `Darkages.exe:0x004A32F0` | `net_socket_ctor` | `void *__thiscall(void *socket_object, void *event_sink)` | Construct Socket state and worker base. | Vtable, buffer-size, event-sink, and default-key anchor. |
| `Darkages.exe:0x004A3570` | `net_c_queue_send` | `void __thiscall(void *socket_object, const uint8_t *packet, int16_t length)` | Copy logical client egress. | Signature validator includes transfer gate, `0x10` exception, sentinel, and work code 5. |
| `Darkages.exe:0x004A39C0` | `net_poll_receive` | `void __thiscall(void *socket_object)` | Run Socket periodic receive polling. | Socket vtable slot `+0x10`; profile-gated Socket command-pump hook. |
| `Darkages.exe:0x004A54D0` | `net_process_work_item` | `void __thiscall(void *socket_object, int code, void *data, int value)` | Dispatch Socket worker records. | Validates queue code relationships and queued-buffer cleanup. |
| `Darkages.exe:0x004A72B4` | `net_c_send_packet_body` | `void __thiscall(void *socket_object, const uint8_t *packet, int16_t length)` | Transform, frame, and send on the Socket worker. | Worker-affine injection adapter; validate that it does not retain or free its input. |
| `Darkages.exe:0x00498080` | `util_ring_buffer_push_wait` | `void __thiscall(void *ring_buffer, const void *element)` | Synchronize bounded-ring admission. | Cross-family proof that native multi-producer admission is race-safe but can block. |
| `Darkages.exe:0x004BF440` | `util_thread_queue_post_async` | `void __thiscall(void *worker_object, int code, void *data, int value)` | Append a raw async record and signal the worker. | Cross-family worker-queue anchor; can block for capacity and caller wrappers establish ownership. |
