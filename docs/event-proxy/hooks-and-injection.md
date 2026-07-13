# Event Proxy Hooks and Injection

The 4.21 client exposes logical boundaries on both sides of the internal Event system. Decoded server packets can be intercepted before Event creation, all pane-bound Events can be intercepted once before recursive delivery, and client packets can be intercepted before transport transformation. These are internal functions, so an injected module needs x86 inline detours or equivalent call-site instrumentation rather than Winsock import hooks.

This page distinguishes confirmed client behavior from recommended proxy behavior. Hook return values, copied-buffer ownership, queue codes, Event fields, and static roots are confirmed. Rule evaluation, telemetry, and IPC commands are proposed.

## High-level operation

### Primary hook matrix

| Phase | Function | Data visible to the hook | Recommended use |
|---|---|---|---|
| Decoded ingress | `event_post_socket_bytes` | `[server action][payload]`, length excludes the local zero sentinel | Observe, block, or copy-on-write modify server packets before Event creation. |
| Central Event | `event_dispatch` | One 36-byte Event before capture-aware hierarchy routing | Observe or block mouse, keyboard, and socket Events once per logical Event. |
| Pane traversal | `event_dispatch_hierarchy` | Event plus the current packed hierarchy list | Optional targeted diagnostics where child and parent delivery order matters. |
| Logical egress | `net_c_queue_send` | `[client action][payload]`, length excludes the sentinel and all wire metadata | Observe, block, or modify native client packets; optional native-queue injection adapter. |
| Socket pump | `net_poll_receive` | Socket worker context after a native wake | Bounded worker-affine client packet injection. |
| EventMan pump | `event_manager_periodic_noop` vtable slot | EventMan worker context after a native wake | Bounded server packet and normalized input injection. |
| Dispatcher tick | `event_dispatcher_tick` | Dispatcher worker context before or after timer processing | Bounded command pump for direct pane, timer, write, and snapshot operations. |

The first and fourth hooks avoid the protocol transformation entirely. Inbound data has already been decoded. Outbound data has not yet received its sequence byte, XOR passes, frame marker, or big-endian frame length.

### Packet blocking and mutation

Blocking is simplest at the copied-input boundaries:

- An inbound block skips `event_post_socket_bytes`. The receive framer has already consumed the frame, and no EventMan-owned copy exists yet.
- An outbound block skips `net_c_queue_send`. The producer retains its stack or object buffer, and the Socket queue has not allocated a copy.
- A central Event block skips the original `event_dispatch`, returns 1 as the consumed result, and does not free the Event or packet. `event_dispatcher_process_work_item` still performs normal socket-payload cleanup and deletes the queued Event after the hook returns.

Packet mutation should be copy-on-write. The proxy copies the logical packet into proxy-owned scratch storage, applies validated edits, and passes that copy to the original client function. Both packet-boundary functions make their own owned copy before returning, so the proxy scratch buffer can be released immediately afterward.

Do not replace `Event +0x14` with memory allocated by the DLL's C runtime. The dispatcher releases an owned socket packet through the client's memory manager. Inbound packet replacement belongs at `event_post_socket_bytes`, where the client performs the allocation itself.

### Client packet injection

A controller command for an arbitrary logical client packet should contain exactly:

```text
[uint8 action] [payload bytes...]
```

It must not contain `0xAA`, the two-byte frame length, a sequence byte, transformed bytes, or the trailing zero sentinel.

The recommended stable path queues the command in a bounded proxy-owned Socket queue, signals the Socket worker's existing wait handle 0, and executes it from the Socket periodic callback. The worker-affine adapter checks the transfer gate, appends the sentinel, and calls:

```c
net_c_send_packet_body(
    net_socket_instance,
    packet_with_sentinel,
    logical_length + 1
);
```

This is the same Socket-worker function reached by native work code 5. It updates sequence and transformation state and performs normal framing and transport output. It reads the supplied packet synchronously and does not retain or free it, so the pump can release its proxy-owned command after the call.

The native-queue alternative calls `net_c_queue_send(net_socket_instance, logical_packet, logical_length)` from a dedicated proxy executor. The confirmed wrapper contract has several consequences:

- `logical_length` includes the action byte and must be positive.
- The parameter is signed 16-bit. The function adds one for the sentinel before placing the value in another signed 16-bit work field, so the practical maximum input is 32,766 bytes.
- The function copies the packet and appends the sentinel before it returns.
- Socket work code 5 performs the later transformation and send.
- When Socket `transfer_gate` at `+0x780D2` is 1, the function drops every action except `0x10`.
- Its 128-record ring is protected by a monitor and not-full condition, so simultaneous native and proxy producers do not corrupt queue state.
- The push waits indefinitely when the ring is full. The wrapper is asynchronous with respect to completion, not nonblocking with respect to admission.
- The function returns no queue or transport success value. An IPC acknowledgement can mean "validated and admitted," not "sent on TCP." A later Socket-worker or `send` observation would be required for delivery-attempt telemetry.

Before either adapter runs, validate that `net_socket_instance` is nonnull and its first pointer is `net_socket_vtable`. A useful command response distinguishes `not_ready`, `not_connected`, `blocked_by_rule`, `blocked_by_transfer_gate`, `admitted`, `executed`, and `invalid_length`.

### Server packet injection

To make a logical server packet enter the client as though it had just been decoded, queue the command for the EventMan worker. The pump allocates `logical_length + 1` through the client's `util_memory_manager_alloc`, copies the packet, appends zero, and invokes the established worker case:

```c
void *memory_manager;
uint8_t *client_owned_packet;

memory_manager = util_memory_manager_get_instance();
client_owned_packet = util_memory_manager_alloc(
    memory_manager,
    logical_length + 1
);
memcpy(client_owned_packet, logical_packet, logical_length);
client_owned_packet[logical_length] = 0;

event_process_work_item(
    event_manager_instance,
    0x0e,
    client_owned_packet,
    logical_length
);
```

The input begins with the server action and excludes the sentinel. The pump must validate a live MemoryMan root before allocation and free the new buffer itself if command execution stops before work code `0x0E` accepts it. Once called, work code `0x0E` creates Event type 9, routes it through `event_dispatch`, and transfers the packet to dispatcher cleanup after pane delivery. Allocation through the executable is required because the dispatcher later calls `util_memory_manager_free`.

The native-queue alternative calls `event_post_socket_bytes(event_manager_instance, logical_packet, logical_length)` from a dedicated proxy executor. That wrapper performs the same client allocation and posts EventMan code `0x0E`. Its 128-record ring is synchronized for multiple producers but can block for capacity, so it must not run on the named-pipe thread or on the EventMan consumer itself.

This path preserves normal pane filtering. A packet is acted on only if an active registered pane claims its action. For example, injecting an in-game action while only `MainMenuPane` is registered does not bypass the UI state machine.

### Input injection

Two input modes are useful:

| Mode | Mechanism | Semantics |
|---|---|---|
| Window-faithful input | `PostMessageA(app_main_window, ...)` | Runs the original window procedure, including software-cursor updates, scan-code extraction, EventMan input blocking, and message-specific quirks. |
| Internal input | Call `event_queue_mouse_*` or `event_queue_key_*` on `event_manager_instance` | Enters the existing EventMan queue directly but does not perform the window procedure's cursor drawing and coordinate preprocessing. |

The complete handled `WM_*`, `WM_USER + 2` Winsock notification, and application-defined `WM_USER + 0x2046` behavior is indexed in [Input and Windows Events](../subsystems/input-and-events.md). Window-faithful injection should be limited to messages whose parameter layout is established there.

For an internal mouse click at a new position, queue a mouse move first, then the button transition. Button work uses the current EventMan mouse coordinates. Queue order preserves the move-before-click relationship. Releases should normally be paired even when a press was blocked because the client deliberately allows button-up and key-up work while input blocking is active.

For the worker-affine architecture, the proxy command queue preserves the same move-before-click ordering and the EventMan pump invokes the corresponding `event_process_work_item` cases directly. Calling an input queue wrapper from the EventMan consumer can self-block if concurrent producers refill the native ring before the wrapper pushes.

Direct keyboard injection uses the client's physical scan-code representation. Extended keys have `0x80` added to the base scan byte. Passing a virtual-key code instead produces the wrong mapping. The controller should either supply the normalized scan byte or request a window-faithful `WM_KEYDOWN` or `WM_KEYUP` with a correctly formed `lParam`.

### Thread-affinity matrix

| Operation | Required execution context | Reason |
|---|---|---|
| Pipe accept, read, write, handshake | Proxy IPC thread | May block on IPC and must not own client state. |
| Rule compilation and subscription changes | Proxy IPC thread | Publishes immutable state atomically; no client call is needed. |
| Client packet injection | Socket worker pump | Serializes sequence, XOR, framing, and transport state. |
| Server packet and normalized input injection | EventMan worker pump | Preserves input state and packet ownership before dispatcher handoff. |
| Pane method, timer, typed UI write, snapshot barrier | Dispatcher worker pump | These structures have no established external-thread lock. |
| Hook telemetry publication | Calling client thread | Bounded copy only; never writes the pipe or waits. |

The proxy can keep a dedicated executor that calls native queue wrappers as a simpler implementation, but it is an admission adapter rather than the IPC pump. Use one executor, one in-flight client call, bounded proxy input, and an `app_shutdown` drain gate. The worker-affine pumps remove that foreign-thread lifetime edge and are the recommended production layout.

### Generic Event injection

The Event object is 36 bytes and uses `event_vtable`. The mapped variants share these fields:

| Offset | Width | Event context | Meaning |
|---:|---:|---|---|
| `+0x00` | 4 | All | `event_vtable` pointer. |
| `+0x04` | 4 | All | `unknown_04` base-object marker initialized by the common constructor. |
| `+0x08` | 4 | All | `unknown_08`. |
| `+0x0C` | 1 | All | Type: mouse subtype 0 through 7, keyboard 8, socket packet 9. |
| `+0x10` | 4 | Mouse | X coordinate. |
| `+0x14` | 4 | Mouse | Y coordinate. |
| `+0x18` | 1 | Mouse | Modifier and button state byte. |
| `+0x1C` | 4 | Wheel | Wheel steps after signed delta division by 120. |
| `+0x20` | 4 | Mouse | Message timestamp. |
| `+0x10` | 1 | Keyboard | Mapped internal key value. |
| `+0x11` | 1 | Keyboard | Active modifier byte. |
| `+0x14` | 4 | Keyboard | Message timestamp. |
| `+0x14` | 4 | Socket | Owned decoded-packet pointer. |
| `+0x18` | 4 | Socket | Positive decoded-packet size. |

`event_manager_queue_event_copy` allocates and shallow-copies all 36 bytes, then posts EventMan work code `0x0F`. The EventMan worker forwards another copy to the dispatcher and deletes its first copy. This function has no native call sites in the analyzed build, although its implementation and worker case are complete.

Use this generic path only for a fully mapped non-owning Event layout. Native input queue functions are safer for mouse and keyboard because they also update EventMan state. `event_post_socket_bytes` is safer for socket packets because it establishes correct packet allocation and ownership. An IPC `INJECT_EVENT_RAW` command should therefore be disabled by default and restricted to a version profile with explicit field validation.

### Timers and pane-specific actions

Pane timers do not become Events. `event_dispatcher_tick` removes a timer record and calls receiver vtable slot `+0x44` with a callback identifier and two payload values. `event_dispatcher_insert_timer` edits the dispatcher's timer list directly and has no visible lock.

A proxy timer command must execute on the dispatcher worker, validate that the receiver is still registered, and use only a callback contract established for that pane class. An arbitrary receiver, callback identifier, or payload can violate pane state assumptions. Packet-driven UI should normally be exercised by injecting its server packet, and user-driven UI should normally be exercised by input injection, rather than calling class handlers directly.

## Code-level flow

### Inbound hook flow

```text
hook_event_post_socket_bytes(event_manager, packet, length)
  -> validate length and read action packet[0]
  -> evaluate local ingress rules
  -> enqueue telemetry copy without waiting
  -> block: return
  -> modify: call original with scratch packet
  -> pass: call original with original packet
      -> copy length + 1
      -> append zero
      -> util_thread_queue_post_async(code 0x0E)
```

The hook must not retain the caller's packet pointer. Normal TCP receive uses a persistent Socket decode buffer that is reused for later frames.

### Central Event hook flow

```text
event_dispatcher_process_work_item(code 3, event)
  -> hook_event_dispatch(dispatcher, event)
      -> classify event[0x0C]
      -> evaluate local Event rules
      -> enqueue bounded telemetry
      -> block: return consumed
      -> pass: call original event_dispatch
  -> if socket Event, free event[0x14]
  -> delete Event
```

Because cleanup belongs to the caller, the hook must never delete the Event. A telemetry record must copy all needed scalar fields and packet bytes before returning.

### Outbound hook flow

```text
native client packet builder
  -> hook_net_c_queue_send(socket, packet, length)
      -> validate action and signed length
      -> evaluate local egress rules
      -> enqueue telemetry copy without waiting
      -> block: return
      -> modify: call original with scratch packet
      -> pass: call original with original packet
          -> enforce transfer gate
          -> allocate and copy length + 1
          -> append zero
          -> util_thread_queue_post_async(code 5)
              -> net_process_work_item
                  -> net_c_send_packet_body
```

The hook should tag origin as `client`, `controller`, or `local_rule`. A thread-local reentry counter and a maximum rule-action depth prevent an injected or rule-generated packet from triggering an unbounded emission cycle. The packet should still be observable unless a subscription explicitly excludes non-client origins.

Worker-affine injected packets do not pass through `net_c_queue_send`. The Socket pump runs the same immutable outbound rules, publishes a logical-egress record with the command origin, appends the sentinel, and calls `net_c_send_packet_body`. This avoids recursive hook entry while retaining one telemetry model.

```text
proxy Socket command
  -> wake Socket wait handle 0
  -> worker calls original net_poll_receive
  -> Socket pump validates root, vtable, generation, and transfer gate
  -> evaluate outbound rules and publish telemetry
  -> net_c_send_packet_body(packet_with_sentinel, logical_length + 1)
```

### Detour ABI requirements

These functions are x86 `__thiscall`. The object pointer arrives in `ECX`, while the remaining arguments are on the stack. A detour wrapper must preserve the nonvolatile registers, stack cleanup convention, and original `ECX`. The trampoline must relocate complete overwritten instructions and any relative branches or calls.

Before patching a target, compare its bytes with the selected build profile and decode enough whole instructions for the jump stub. Do not install a detour from a raw address match alone. The [Version Profiles](version-profiles.md) page defines the required semantic validation.

## Function table

| Address | Current IDA name | Prototype | Purpose | Call relationships and notes |
|---:|---|---|---|---|
| `Darkages.exe:0x00431200` | `event_dispatcher_queue_event_copy` | `void __thiscall(void *event_dispatcher_object, const void *event)` | Allocate and shallow-copy a 36-byte Event, then post dispatcher work code 3. | Native mouse, key, and socket Event producers use this queue. |
| `Darkages.exe:0x004315B0` | `event_dispatcher_tick` | `void __thiscall(void *event_dispatcher_object)` | Run periodic dispatcher work. | Optional bounded dispatcher-thread proxy command pump. |
| `Darkages.exe:0x004316E0` | `event_dispatcher_process_work_item` | `void __thiscall(void *event_dispatcher_object, int code, void *data, int value)` | Process dispatcher queue records. | Code 3 dispatches, frees an owned socket payload, and deletes the Event. |
| `Darkages.exe:0x00431B84` | `event_dispatch` | `int __thiscall(void *event_dispatcher_object, void *event)` | Apply capture handling and dispatch one Event. | Primary one-record-per-Event hook. |
| `Darkages.exe:0x00431D54` | `event_dispatch_hierarchy` | `int __thiscall(void *event_dispatcher_object, void *event, void *hierarchy)` | Recursively call pane input or socket handlers. | Child lists are visited before their current pane. |
| `Darkages.exe:0x00432BD0` | `event_queue_mouse_move` | `void __thiscall(void *event_manager_object, int mouse_y, int mouse_x, uint32_t message_time)` | Queue internal mouse movement. | Work code 4; updates EventMan coordinates when processed. |
| `Darkages.exe:0x00432C50` | `event_queue_left_button_down` | `void __thiscall(void *event_manager_object, uint32_t message_time)` | Queue left-button press. | Work code 5; uses current EventMan coordinates. |
| `Darkages.exe:0x00432C90` | `event_queue_left_button_up` | `void __thiscall(void *event_manager_object, uint32_t message_time)` | Queue left-button release. | Work code 6; not suppressed by input blocking. |
| `Darkages.exe:0x00432CC0` | `event_queue_right_button_down` | `void __thiscall(void *event_manager_object, uint32_t message_time)` | Queue right-button press. | Work code 7. |
| `Darkages.exe:0x00432D00` | `event_queue_right_button_up` | `void __thiscall(void *event_manager_object, uint32_t message_time)` | Queue right-button release. | Work code 8; not suppressed by input blocking. |
| `Darkages.exe:0x00432D30` | `event_queue_mouse_wheel` | `void __thiscall(void *event_manager_object, int wheel_delta, uint32_t message_time)` | Queue signed wheel input. | Work code 9; worker divides by 120. |
| `Darkages.exe:0x00432D60` | `event_queue_key_down` | `void __thiscall(void *event_manager_object, uint8_t scan_code, uint32_t message_time)` | Queue normalized physical key press. | Work code `0x0A`. |
| `Darkages.exe:0x00432D90` | `event_queue_key_up` | `void __thiscall(void *event_manager_object, uint8_t scan_code, uint32_t message_time)` | Queue normalized physical key release. | Work code `0x0B`; updates state without a pane Event. |
| `Darkages.exe:0x00432E50` | `event_post_socket_bytes` | `void __thiscall(void *event_manager_object, const uint8_t *packet, int length)` | Copy and queue a decoded server packet. | Work code `0x0E`; preferred native-ingress block boundary and synchronized fallback injection adapter. |
| `Darkages.exe:0x00432F10` | `event_manager_queue_event_copy` | `void __thiscall(void *event_manager_object, const void *event)` | Copy and queue a raw Event through EventMan. | Work code `0x0F`; no native call sites in this build. |
| `Darkages.exe:0x00433110` | `event_process_work_item` | `void __thiscall(void *event_manager_object, int code, void *data, int value)` | Convert EventMan records into state updates or Events. | Codes 4 through `0x0B`, `0x0E`, and `0x0F` are relevant to injection. |
| `Darkages.exe:0x00433DC4` | `event_queue_socket_packet` | `void __stdcall(uint8_t *packet, uint32_t size)` | Build Event type 9 and submit it to the dispatcher. | Transfers packet ownership to dispatcher cleanup. |
| `Darkages.exe:0x00434080` | `event_manager_periodic_noop` | `void __thiscall(void *event_manager_object)` | Native EventMan periodic callback. | Vtable slot `+0x10`; practical worker-affine command-pump slot. |
| `Darkages.exe:0x0045BDC0` | `win_main_window_proc` | `LRESULT __stdcall(HWND, UINT, WPARAM, LPARAM)` | Convert handled Win32 input and socket notifications. | Window-faithful input injection uses `PostMessageA` to this procedure's window. |
| `Darkages.exe:0x00470F80` | `util_memory_manager_get_instance` | `void *__cdecl(void)` | Return the client MemoryMan singleton. | Required before allocating an owned server-injection buffer. |
| `Darkages.exe:0x00470FC0` | `util_memory_manager_alloc` | `void *__thiscall(void *memory_manager, int size)` | Allocate through the executable's CRT wrapper. | Use for buffers that dispatcher cleanup will own. |
| `Darkages.exe:0x004A3570` | `net_c_queue_send` | `void __thiscall(void *socket_object, const uint8_t *packet, int16_t length)` | Copy and queue a logical client packet. | Work code 5; transfer gate permits only `0x10` during transfer. |
| `Darkages.exe:0x004A39C0` | `net_poll_receive` | `void __thiscall(void *socket_object)` | Poll Socket receive state from periodic worker execution. | Vtable slot `+0x10`; recommended Socket command-pump hook. |
| `Darkages.exe:0x004A54D0` | `net_process_work_item` | `void __thiscall(void *socket_object, int code, void *data, int value)` | Execute Socket queue records. | Code 5 calls `net_c_send_packet_body` and frees the queued copy. |
| `Darkages.exe:0x004A72B4` | `net_c_send_packet_body` | `void __thiscall(void *socket_object, const uint8_t *packet, int16_t length)` | Transform, frame, and call the configured transport. | Socket-worker adapter; reads but does not retain or free the input. |
| `Darkages.exe:0x004BF440` | `util_thread_queue_post_async` | `void __thiscall(void *worker_object, int code, void *data, int value)` | Post one raw asynchronous work record. | Does not copy `data` or wait for completion, but can block while the native ring is full. |
