# Internal Event Routing

The internal event system owns two related paths. It routes typed events through registered panes, and it runs periodic timer callbacks on a worker thread independently of the blocking Windows message pump. Network packets enter the same pane hierarchy through a separate event-manager worker.

## High-level operation

### Dispatcher, event manager, and pane hierarchy

`app_initialize` creates the global `event_dispatcher`, starts its worker, then constructs and starts the separate `event_manager_instance`. The dispatcher owns the pane registry and the timer list. The event manager converts queued work records into Event objects, including socket-packet events, and submits those events to the dispatcher.

The established socket path is:

1. The Socket receive worker posts copied decoded bytes through `event_post_socket_bytes`.
2. `event_process_work_item` converts the queued record to Event type 9.
3. `event_queue_socket_packet` transfers the owned packet pointer and size into the Event queue.
4. `event_dispatch` and `event_dispatch_hierarchy` walk registered panes and invoke the socket-event virtual handler.

Packet ownership and the type 9 fields are documented in [Networking](networking.md#receive-and-event-routing). Pane registration and timer callbacks remain owned by this subsystem.

The dispatcher hierarchy is separate from the Screen composition hierarchy. The event tree stores packed `0x0F`-byte nodes with a pane pointer at `+0x08`, while Screen stores placement and drawing relationships in `0x3F`-byte nodes. Their raw layouts, pane virtual slots, creation and removal flows, and complete vtable inventory are documented in [UI Panes and Registries](ui-panes.md).

Window input uses the same EventMan worker. The main window procedure posts work codes 4 through `0x0B` for mouse movement, buttons, wheel, and physical scan codes. `event_process_work_item` converts those records into mouse and key state or Events before the normal pane hierarchy sees them. The complete mapping is documented in [Input and Windows Events](input-and-events.md#mouse-queue-and-worker-path).

### Worker and timer lifecycle

The Windows message loop is not a frame loop. The dispatcher derives from a cross-subsystem thread-queue base whose constructor creates a thread with `_beginthreadex` and `CREATE_SUSPENDED`. The same base is also used by the Socket and other worker-derived objects. The dispatcher configures that worker before `util_thread_queue_start` resumes it:

| Setting | Established value or rule |
|---|---|
| Work-queue capacity | `0x400` records |
| `WaitForMultipleObjects` timeout | 1 millisecond |
| Requested multimedia timer period | 5 milliseconds clamped to `TIMECAPS.wPeriodMin..wPeriodMax` |
| Timer clock | 32-bit `timeGetTime()` milliseconds |
| Initial next deadline | `UINT32_MAX` |
| Timer-list capacity | 256 records |
| Timer-record size | 20 bytes |
| Worker priority | `THREAD_PRIORITY_TIME_CRITICAL` |

The dispatcher calls `timeGetDevCaps` but does not check its return. It calls `timeBeginPeriod` with the selected period, stores the first `timeGetTime` sample, creates the timer list, and raises the still-suspended worker's priority. A 1 millisecond wait timeout is therefore a polling interval requested from the worker, not a promise that Windows will deliver callbacks every millisecond. The multimedia period is normally 5 milliseconds when the device range permits it.

`util_thread_queue_start` first calls virtual slot `+0x08` so the derived object can add its wait object. It resumes the thread only when `app_error_code` remains zero.

The worker runs `WaitForMultipleObjects(wait_count, wait_handles, FALSE, timeout_ms)` forever. Wait handle index 0 is the queued-work notification. A work wake removes one record and calls the derived work-item virtual method. Other registered handle indices call virtual slot `+0x1C`. After every handled wake, empty queue wake, secondary signal, or timeout, the loop calls virtual slot `+0x10`. In the dispatcher vtable, that periodic slot is `event_dispatcher_tick`.

### Timer records and scheduling

Timer insertion normally reaches `event_dispatcher_process_work_item` as work code 4. Code 5 cancels all timers for a pane. `event_dispatcher_insert_timer` converts the relative delay to an absolute 32-bit deadline by adding it to the dispatcher's most recent `timeGetTime` sample.

Each sorted timer record has this layout:

| Offset | Width | Meaning |
|---:|---:|---|
| `+0x00` | 4 | Receiver pane pointer |
| `+0x04` | 4 | Callback identifier |
| `+0x08` | 4 | Absolute `timeGetTime` deadline |
| `+0x0C` | 4 | First callback payload value |
| `+0x10` | 4 | Second callback payload value |

The insertion scan uses an unsigned comparison and places the record before the first later deadline. Equal deadlines remain in insertion order. If the new record becomes element 0, its deadline is copied to the dispatcher's cached next-deadline field.

`event_dispatcher_remove_pane_timers` scans the complete list and removes every record whose receiver pointer matches the pane. It then sets the cached deadline from the new first record, or to `UINT32_MAX` when the list is empty.

### Periodic tick and deferred deletion

Each `event_dispatcher_tick` first drains `event_deferred_delete_queue`. The drain walks queued object pointers from the last element to the first, calls each object's deleting destructor with flag 1, and removes the entry. This gives client code a deferred-destruction path tied to the dispatcher worker rather than the Windows message thread.

The tick then samples `timeGetTime`, stores the result, and compares it to the cached next deadline as unsigned 32-bit values. If the current tick is earlier, the tick returns. Otherwise it removes only the first timer record, refreshes the cached deadline, verifies that the receiver is still an active object, and calls receiver virtual slot `+0x44` with the callback identifier and the two payload values.

At most one due timer is dispatched per worker iteration. When several timers are overdue, they are removed across successive worker wakes or timeouts.

The code uses plain unsigned addition, sorting, and `current_tick < deadline` comparisons. It does not use a signed modular difference or another visible `timeGetTime` wraparound correction. Implementations that need byte-for-byte behavioral compatibility should preserve that distinction; implementations seeking a robust long-running clock should account for 32-bit wrap separately.

### Shutdown and ownership

`app_shutdown` deletes the event dispatcher before it deletes the global deferred-deletion queue. The dispatcher destructor performs these operations in order:

1. Delete the pane registry.
2. Delete the timer list.
3. Call `timeEndPeriod` with the stored period.
4. Invoke the worker-base destructor.

The worker-base destructor calls `TerminateThread(worker_handle, 0)`, closes the thread handle and every wait handle, and destroys the queued message and synchronization objects. There is no cooperative stop flag, worker acknowledgement, or thread join. The pane and timer containers are released before the worker is forcibly terminated, so shutdown relies on the fixed surrounding teardown sequence and timing rather than an explicit worker quiescence handshake.

## Code-level flow

### Construction and start

1. `Darkages.exe:0x0045CCA0` `app_initialize` allocates the 128-byte dispatcher object and calls `event_dispatcher_ctor`.
2. `Darkages.exe:0x00430FE0` calls `util_thread_queue_ctor(this, 0x400)`. The base constructor creates queue state and a suspended `_beginthreadex` worker stored at object offset `+0x60`.
3. The dispatcher installs `event_dispatcher_vtable`, calls `util_thread_queue_set_wait_timeout(this, 1)`, and allocates its pane registry at `+0x68`.
4. It reads `TIMECAPS`, clamps 5 milliseconds into the reported range, stores the result at `+0x6C`, and calls `timeBeginPeriod`.
5. It stores `timeGetTime()` at `+0x70`, sets cached next deadline `+0x74` to `UINT32_MAX`, and creates a 256-element timer list at `+0x78` with 20-byte elements.
6. It sets the suspended thread to `THREAD_PRIORITY_TIME_CRITICAL`.
7. `app_initialize` stores the object in `event_dispatcher` and calls `util_thread_queue_start`, which creates the derived wait object and resumes the worker.
8. After the screen pane exists, `event_dispatcher_register_pane(screen_pane, 0, 0)` inserts it into the pane registry.
9. `app_initialize` separately allocates the EventMan object, calls `event_manager_ctor`, and calls `util_thread_queue_start`. The constructor uses a work-queue capacity of `0x80`, publishes `event_manager_instance`, and sets its worker timeout from the constructed event-manager state.

Allocation failure sets `app_error_code` to 2 and records the same error in the dispatcher base. The initializer then follows its staged failure cleanup.

### Worker loop

`Darkages.exe:0x004BF250` `util_thread_queue_worker_loop` has no normal exit branch. Its repeating path is equivalent to:

```c
for (;;) {
    uint32_t wait_result;

    wait_result = WaitForMultipleObjects(
        worker->wait_count,
        worker->wait_handles,
        0,
        worker->timeout_ms
    );

    if (wait_result == WAIT_OBJECT_0) {
        process_one_queued_record(worker);
    } else if (wait_result > WAIT_OBJECT_0 &&
               wait_result < WAIT_OBJECT_0 + worker->wait_count) {
        process_secondary_wait_handle(worker, wait_result);
    }

    run_periodic_worker_method(worker);
}
```

The real queue path supports both asynchronous records and synchronous records whose producer waits on an event for a returned result. After processing a synchronous record, the worker writes the result to the matching waiter entry and signals its event.

### Timer insertion and callback

For work code 4, `event_dispatcher_process_work_item` extracts the receiver, callback identifier, delay, and two payload values and calls `event_dispatcher_insert_timer`. The insertion function builds the 20-byte record, performs a linear scan of the sorted list, inserts it, and updates `+0x74` only when inserted at index 0.

The periodic vtable call reaches `event_dispatcher_tick`:

```c
event_deferred_delete_queue_drain(event_deferred_delete_queue);

current_tick = timeGetTime();
dispatcher->current_tick = current_tick;
if (current_tick < dispatcher->next_deadline) {
    return;
}

timer_record = remove_first_timer_record(dispatcher);
refresh_next_deadline(dispatcher);

if (receiver_is_active(timer_record.receiver)) {
    invoke_timer_callback(
        timer_record.receiver,
        timer_record.callback_id,
        timer_record.payload_0,
        timer_record.payload_1
    );
}
```

The implementation removes the timer before checking receiver activity. An inactive receiver therefore loses the due callback rather than retaining or rescheduling it.

### Network event handoff

The separate event manager's mapped packet route is documented in detail under [receive and event routing](networking.md#receive-and-event-routing). The important cross-thread ownership boundary is that `event_post_socket_bytes` copies the decoded packet into a work record. `event_process_work_item` later constructs the Event, and the event manager frees the owned packet after pane dispatch.

### Dispatcher object fields

Offsets below are from the event dispatcher object. The worker-base fields are shared with the event manager and other worker-derived objects.

| Offset | Width | Working field | Reads and writes |
|---:|---:|---|---|
| `+0x00` | 4 | `vtable` | Set to `event_dispatcher_vtable`. |
| `+0x0C` | 4 | `wait_timeout_ms` | Set to 1 by `util_thread_queue_set_wait_timeout`; consumed by `util_thread_queue_worker_loop`. |
| `+0x10` | 1 | `wait_handle_count` | Passed to `WaitForMultipleObjects`. |
| `+0x14` | variable | `wait_handles` | Handle index 0 is the worker queue notification. |
| `+0x60` | 4 | `worker_thread` | Suspended handle created by the base constructor, resumed by `util_thread_queue_start`, forcibly terminated by the base destructor. |
| `+0x68` | 4 | `pane_registry` | Points to a packed hierarchy with stride `0x0F`; each node stores its child list at `+0x04` and pane at `+0x08`. |
| `+0x6C` | 4 | `multimedia_period_ms` | Period passed to `timeBeginPeriod` and later `timeEndPeriod`. |
| `+0x70` | 4 | `current_tick` | Latest `timeGetTime` sample; also the base for newly inserted relative delays. |
| `+0x74` | 4 | `next_deadline` | First timer deadline, or `UINT32_MAX` when empty. |
| `+0x78` | 4 | `timer_list` | Sorted list of 20-byte timer records. |

## Function table

| Address | Current IDA name | Prototype | Purpose | Call relationships and notes |
|---:|---|---|---|---|
| `Darkages.exe:0x0040E4D0` | `event_deferred_delete_queue_ctor` | `void *__thiscall(void *queue_object)` | Construct the deferred-deletion queue. | Called by `app_initialize`; source diagnostics identify the original implementation unit as `BlackHole.cpp`. |
| `Darkages.exe:0x0040E5C0` | `event_deferred_delete_queue_drain` | `void __thiscall(void *queue_object)` | Delete and remove every deferred object. | Called first by each `event_dispatcher_tick` and during shutdown before queue deletion. |
| `Darkages.exe:0x00430F84` | `event_dispatcher_dtor` | `void __thiscall(void *event_dispatcher_object)` | Release dispatcher containers and multimedia timing state. | Calls `timeEndPeriod`, then `util_thread_queue_dtor`. |
| `Darkages.exe:0x00430FE0` | `event_dispatcher_ctor` | `void *__thiscall(void *event_dispatcher_object)` | Construct the dispatcher, timer list, and worker timing state. | Called by `app_initialize`; calls `util_thread_queue_ctor`, `util_thread_queue_set_wait_timeout`, `timeBeginPeriod`, and `SetThreadPriority`. |
| `Darkages.exe:0x00431150` | `event_dispatcher_register_pane` | `void __thiscall(void *event_dispatcher_object, void *pane, int hierarchy, int position)` | Add a pane and hierarchy metadata to the dispatcher registry. | Called by `app_initialize` for the screen pane and by later pane lifecycle paths. |
| `Darkages.exe:0x004311B0` | `event_dispatcher_unregister_pane` | `void __thiscall(void *event_dispatcher_object, void *pane)` | Remove a pane from the dispatcher registry. | Prevents later hierarchy dispatch to the removed pane. |
| `Darkages.exe:0x00431320` | `event_dispatcher_insert_timer` | `void __thiscall(void *event_dispatcher_object, void *receiver, int callback_id, unsigned int delay_ms, int payload_0, int payload_1)` | Insert a relative timer into the absolute-deadline sorted list. | Reached from dispatcher work code 4; updates cached deadline when inserted first. |
| `Darkages.exe:0x00431470` | `event_dispatcher_remove_pane_timers` | `void __thiscall(void *event_dispatcher_object, void *receiver)` | Cancel every timer owned by one pane. | Reached from dispatcher work code 5; refreshes the cached deadline. |
| `Darkages.exe:0x004315B0` | `event_dispatcher_tick` | `void __thiscall(void *event_dispatcher_object)` | Drain deferred deletion and dispatch at most one due timer. | Dispatcher vtable slot `+0x10`; called by `util_thread_queue_worker_loop` after every wake or timeout. |
| `Darkages.exe:0x004316E0` | `event_dispatcher_process_work_item` | `void __thiscall(void *event_dispatcher_object, int code, void *data, int value)` | Execute dispatcher work codes 1 through 5. | Timer insertion is code 4; pane-timer cancellation is code 5. |
| `Darkages.exe:0x00431B84` | `event_dispatch` | `int __thiscall(void *this, void *event)` | Dispatch one internal Event. | Called by the event manager; delegates hierarchy traversal to `event_dispatch_hierarchy`. |
| `Darkages.exe:0x00431D54` | `event_dispatch_hierarchy` | `int __thiscall(void *this, void *event, void *hierarchy)` | Walk registered panes and call the type-specific virtual handler. | Socket Event type 9 reaches pane virtual slot `+0x40`. |
| `Darkages.exe:0x00432630` | `event_manager_ctor` | `void *__thiscall(void *event_manager_object)` | Construct and publish the EventMan singleton. | Calls `util_thread_queue_ctor` with capacity `0x80`, initializes event-manager and Socket-facing state, and stores `event_manager_instance`. |
| `Darkages.exe:0x00432E50` | `event_post_socket_bytes` | `void __thiscall(void *this, const uint8_t *packet, int length)` | Copy decoded packet bytes to event-manager work code `0x0E`. | Called by the Socket receive path. |
| `Darkages.exe:0x00433110` | `event_process_work_item` | `void __thiscall(void *this, int code, void *data, int value)` | Convert event-manager work records into input state or Event objects. | Work codes 4 through `0x0B` handle window input; code `0x0E` creates a socket-packet Event and transfers packet ownership. |
| `Darkages.exe:0x00433DC4` | `event_queue_socket_packet` | `void __stdcall(uint8_t *packet, uint32_t size)` | Queue Event type 9 with packet ownership. | Called by `event_process_work_item`; the event manager frees the packet after dispatch. |
| `Darkages.exe:0x004BE9B0` | `util_thread_queue_ctor` | `void *__thiscall(void *worker_object, int queue_capacity)` | Construct queue state and a suspended worker thread. | Cross-subsystem base constructor used by the dispatcher, event manager, Socket, and other worker-derived objects. |
| `Darkages.exe:0x004BECB0` | `util_thread_queue_dtor` | `void __thiscall(void *worker_object)` | Terminate the worker and close its handles and queue state. | Uses `TerminateThread`; called by derived destructors. |
| `Darkages.exe:0x004BEDF0` | `util_thread_queue_set_wait_timeout` | `void __thiscall(void *worker_object, unsigned int timeout_ms)` | Set the worker wait timeout. | Dispatcher constructor passes 1 millisecond. |
| `Darkages.exe:0x004BEE00` | `util_thread_queue_start` | `void __thiscall(void *worker_object)` | Create the derived wait object and resume the suspended thread. | Called by `app_initialize` for both event workers and by other worker-derived clients. |
| `Darkages.exe:0x004BF250` | `util_thread_queue_worker_loop` | `void __thiscall(void *worker_object)` | Wait for work, signals, or timeout and invoke derived virtual methods. | Infinite worker body; periodic virtual slot `+0x10` reaches `event_dispatcher_tick` for the dispatcher object. |
