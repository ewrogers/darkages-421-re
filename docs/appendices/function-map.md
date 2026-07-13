# Function Map

This appendix is a cross-subsystem index of important functions, grouped by primary subsystem. Cross-subsystem functions appear under the subsystem that owns their main control flow. Detailed decisions belong in the relevant subsystem chapter. An address remains the durable identifier when a name improves later.

## Application lifecycle

| Address | Current IDA name | Subsystem | Prototype | Purpose | Detailed page |
|---:|---|---|---|---|---|
| `Darkages.exe:0x0041D390` | `app_check_conflicting_software` | Application | `int __cdecl(void)` | Prompt for four process suffixes and two files before startup. | [Startup and Shutdown](../lifecycle/startup.md#preflight-checks-and-single-instance-enforcement) |
| `Darkages.exe:0x0041D690` | `win_top_level_window_exists` | Win32 | `BOOL __cdecl(LPCSTR class_name, LPCSTR window_title)` | Test for a matching top-level window. | [Startup and Shutdown](../lifecycle/startup.md#preflight-checks-and-single-instance-enforcement) |
| `Darkages.exe:0x0041D6C0` | `app_check_runservices_entries` | Application | `int __cdecl(void)` | Examine and optionally delete three `RunServices` values. | [Startup and Shutdown](../lifecycle/startup.md#preflight-checks-and-single-instance-enforcement) |
| `Darkages.exe:0x0045B5F0` | `app_win_main` | Application and Win32 | `int __stdcall(HINSTANCE, HINSTANCE, LPSTR, int)` | Own class registration, checks, window creation, the message loop, and normal shutdown. | [Startup and Shutdown](../lifecycle/startup.md) |
| `Darkages.exe:0x0045B8F0` | `app_shutdown` | Application | `void __cdecl(void)` | Delete the initialized subsystem graph in fixed order. | [Startup and Shutdown](../lifecycle/startup.md#normal-and-abnormal-shutdown) |
| `Darkages.exe:0x0045C840` | `app_set_working_directory_from_command_line` | Application | `void __cdecl(void)` | Parse the executable directory from the raw command line and make it current. | [Startup and Shutdown](../lifecycle/startup.md#command-line-directory-parsing) |
| `Darkages.exe:0x0045CCA0` | `app_initialize` | Application | `void __cdecl(void)` | Construct resources, services, screen state, and event workers. | [Startup and Shutdown](../lifecycle/startup.md#subsystem-initialization) |
| `Darkages.exe:0x00498430` | `app_virus_check_module_callback` | Application and Win32 | `int __cdecl(void *, void *, const char *, FILE *)` | Post the first virus-check module report to the main window. | [Input and Windows Events](../subsystems/input-and-events.md#application-defined-messages) |
| `Darkages.exe:0x00498780` | `app_virus_check_thread` | Application | `void __cdecl(char *log_filename)` | Run the process and module virus checker on a worker thread. | [Input and Windows Events](../subsystems/input-and-events.md#application-defined-messages) |
| `Darkages.exe:0x0049D790` | `ui_screen_pane_activate` | UI and Win32 | `void __thiscall(void *screen_pane)` | Restore active screen and presentation state. | [Startup and Shutdown](../lifecycle/startup.md#message-pump-and-activation) |
| `Darkages.exe:0x004C62A5` | `app_crt_entry` | Application and CRT | `void __cdecl(void)` | Initialize the Microsoft C runtime and call `app_win_main`. | [Startup and Shutdown](../lifecycle/startup.md#crt-handoff) |

## Events

| Address | Current IDA name | Subsystem | Prototype | Purpose | Detailed page |
|---:|---|---|---|---|---|
| `Darkages.exe:0x0040E4D0` | `event_deferred_delete_queue_ctor` | Events | `void *__thiscall(void *queue_object)` | Construct the deferred-deletion queue. | [Internal Event Routing](../subsystems/internal-events.md#periodic-tick-and-deferred-deletion) |
| `Darkages.exe:0x0040E5C0` | `event_deferred_delete_queue_drain` | Events | `void __thiscall(void *queue_object)` | Delete and remove every deferred object. | [Internal Event Routing](../subsystems/internal-events.md#periodic-tick-and-deferred-deletion) |
| `Darkages.exe:0x00430F84` | `event_dispatcher_dtor` | Events and timing | `void __thiscall(void *event_dispatcher_object)` | Release dispatcher containers and balance multimedia timing state. | [Internal Event Routing](../subsystems/internal-events.md#shutdown-and-ownership) |
| `Darkages.exe:0x00430FE0` | `event_dispatcher_ctor` | Events and timing | `void *__thiscall(void *event_dispatcher_object)` | Construct the dispatcher, timer list, and worker timing state. | [Internal Event Routing](../subsystems/internal-events.md#worker-and-timer-lifecycle) |
| `Darkages.exe:0x00431150` | `event_dispatcher_register_pane` | Events | `void __thiscall(void *event_dispatcher_object, void *pane, int hierarchy, int position)` | Register a pane and hierarchy metadata. | [Internal Event Routing](../subsystems/internal-events.md#dispatcher-event-manager-and-pane-hierarchy) |
| `Darkages.exe:0x004311B0` | `event_dispatcher_unregister_pane` | Events | `void __thiscall(void *event_dispatcher_object, void *pane)` | Remove a pane from dispatcher traversal. | [Internal Event Routing](../subsystems/internal-events.md#dispatcher-event-manager-and-pane-hierarchy) |
| `Darkages.exe:0x00431320` | `event_dispatcher_insert_timer` | Events and timing | `void __thiscall(void *event_dispatcher_object, void *receiver, int callback_id, unsigned int delay_ms, int payload_0, int payload_1)` | Insert a timer into the absolute-deadline sorted list. | [Internal Event Routing](../subsystems/internal-events.md#timer-records-and-scheduling) |
| `Darkages.exe:0x00431470` | `event_dispatcher_remove_pane_timers` | Events and timing | `void __thiscall(void *event_dispatcher_object, void *receiver)` | Cancel every timer for one pane. | [Internal Event Routing](../subsystems/internal-events.md#timer-records-and-scheduling) |
| `Darkages.exe:0x004315B0` | `event_dispatcher_tick` | Events and timing | `void __thiscall(void *event_dispatcher_object)` | Drain deferred deletion and dispatch at most one due timer. | [Internal Event Routing](../subsystems/internal-events.md#periodic-tick-and-deferred-deletion) |
| `Darkages.exe:0x004316E0` | `event_dispatcher_process_work_item` | Events | `void __thiscall(void *event_dispatcher_object, int code, void *data, int value)` | Execute dispatcher work codes, including timer insert and cancellation. | [Internal Event Routing](../subsystems/internal-events.md#timer-insertion-and-callback) |
| `Darkages.exe:0x00431B84` | `event_dispatch` | Events | `int __thiscall(void *this, void *event)` | Dispatch one internal Event. | [Internal Event Routing](../subsystems/internal-events.md#dispatcher-event-manager-and-pane-hierarchy) |
| `Darkages.exe:0x00431D54` | `event_dispatch_hierarchy` | Events | `int __thiscall(void *this, void *event, void *hierarchy)` | Walk panes and call the type-specific virtual handler. | [Internal Event Routing](../subsystems/internal-events.md#dispatcher-event-manager-and-pane-hierarchy) |
| `Darkages.exe:0x00432630` | `event_manager_ctor` | Events and network | `void *__thiscall(void *event_manager_object)` | Construct and publish the EventMan singleton. | [Internal Event Routing](../subsystems/internal-events.md#construction-and-start) |
| `Darkages.exe:0x00432BD0` | `event_queue_mouse_move` | Events and input | `void __thiscall(void *, int, int, unsigned int)` | Queue copied mouse coordinates and message time as work code 4. | [Input and Windows Events](../subsystems/input-and-events.md#mouse-queue-and-worker-path) |
| `Darkages.exe:0x00432C50` | `event_queue_left_button_down` | Events and input | `void __thiscall(void *, unsigned int)` | Set left-button state and queue work code 5. | [Input and Windows Events](../subsystems/input-and-events.md#mouse-queue-and-worker-path) |
| `Darkages.exe:0x00432C90` | `event_queue_left_button_up` | Events and input | `void __thiscall(void *, unsigned int)` | Clear left-button state and queue work code 6. | [Input and Windows Events](../subsystems/input-and-events.md#mouse-queue-and-worker-path) |
| `Darkages.exe:0x00432CC0` | `event_queue_right_button_down` | Events and input | `void __thiscall(void *, unsigned int)` | Set right-button state and queue work code 7. | [Input and Windows Events](../subsystems/input-and-events.md#mouse-queue-and-worker-path) |
| `Darkages.exe:0x00432D00` | `event_queue_right_button_up` | Events and input | `void __thiscall(void *, unsigned int)` | Clear right-button state and queue work code 8. | [Input and Windows Events](../subsystems/input-and-events.md#mouse-queue-and-worker-path) |
| `Darkages.exe:0x00432D30` | `event_queue_mouse_wheel` | Events and input | `void __thiscall(void *, int, unsigned int)` | Queue wheel delta and message time as work code 9. | [Input and Windows Events](../subsystems/input-and-events.md#mouse-queue-and-worker-path) |
| `Darkages.exe:0x00432D60` | `event_queue_key_down` | Events and input | `void __thiscall(void *, unsigned char, unsigned int)` | Queue a physical key press as work code `0x0A`. | [Input and Windows Events](../subsystems/input-and-events.md#keyboard-queue-and-worker-path) |
| `Darkages.exe:0x00432D90` | `event_queue_key_up` | Events and input | `void __thiscall(void *, unsigned char, unsigned int)` | Queue a physical key release as work code `0x0B`. | [Input and Windows Events](../subsystems/input-and-events.md#keyboard-queue-and-worker-path) |
| `Darkages.exe:0x00432E50` | `event_post_socket_bytes` | Events and network | `void __thiscall(void *this, const uint8_t *packet, int length)` | Copy decoded packet bytes to work code `0x0E`. | [Internal Event Routing](../subsystems/internal-events.md#network-event-handoff) |
| `Darkages.exe:0x00433060` | `event_manager_get_instance` | Events | `void *__cdecl(void)` | Assert and return the EventMan singleton. | [Input and Windows Events](../subsystems/input-and-events.md#mouse-queue-and-worker-path) |
| `Darkages.exe:0x00433110` | `event_process_work_item` | Events | `void __thiscall(void *this, int code, void *data, int value)` | Convert event-manager worker records to Event objects. | [Internal Event Routing](../subsystems/internal-events.md#network-event-handoff) |
| `Darkages.exe:0x00433434` | `event_dispatch_mouse_move` | Events and input | `void __thiscall(void *, int, int, unsigned int)` | Dispatch mouse subtype 0. | [Input and Windows Events](../subsystems/input-and-events.md#mouse-queue-and-worker-path) |
| `Darkages.exe:0x00433504` | `event_dispatch_left_button_down` | Events and input | `void __thiscall(void *, unsigned int)` | Dispatch left single- or double-button-down input. | [Input and Windows Events](../subsystems/input-and-events.md#mouse-queue-and-worker-path) |
| `Darkages.exe:0x004336C4` | `event_dispatch_left_button_up` | Events and input | `void __thiscall(void *, unsigned int)` | Dispatch left-button release. | [Input and Windows Events](../subsystems/input-and-events.md#mouse-queue-and-worker-path) |
| `Darkages.exe:0x00433794` | `event_dispatch_right_button_down` | Events and input | `void __thiscall(void *, unsigned int)` | Dispatch right single- or double-button-down input. | [Input and Windows Events](../subsystems/input-and-events.md#mouse-queue-and-worker-path) |
| `Darkages.exe:0x00433944` | `event_dispatch_right_button_up` | Events and input | `void __thiscall(void *, unsigned int)` | Dispatch right-button release. | [Input and Windows Events](../subsystems/input-and-events.md#mouse-queue-and-worker-path) |
| `Darkages.exe:0x00433A14` | `event_dispatch_mouse_wheel` | Events and input | `void __thiscall(void *, int, unsigned int)` | Divide the wheel delta by 120 and dispatch subtype 7. | [Input and Windows Events](../subsystems/input-and-events.md#mouse-queue-and-worker-path) |
| `Darkages.exe:0x00433AB4` | `event_dispatch_key_down` | Events and input | `void __thiscall(void *, unsigned char, unsigned int)` | Update mapped key state and dispatch key subtype 8. | [Input and Windows Events](../subsystems/input-and-events.md#keyboard-queue-and-worker-path) |
| `Darkages.exe:0x00433C34` | `event_update_key_up` | Events and input | `void __thiscall(void *, unsigned char, unsigned int)` | Clear pressed and modifier state without pane dispatch. | [Input and Windows Events](../subsystems/input-and-events.md#keyboard-queue-and-worker-path) |
| `Darkages.exe:0x00433DC4` | `event_queue_socket_packet` | Events and network | `void __stdcall(uint8_t *packet, uint32_t size)` | Queue Event type 9 with packet ownership. | [Internal Event Routing](../subsystems/internal-events.md#network-event-handoff) |
| `Darkages.exe:0x004BE9B0` | `util_thread_queue_ctor` | Cross-subsystem worker | `void *__thiscall(void *worker_object, int queue_capacity)` | Construct queue state and a suspended worker thread. | [Internal Event Routing](../subsystems/internal-events.md#construction-and-start) |
| `Darkages.exe:0x004BECB0` | `util_thread_queue_dtor` | Cross-subsystem worker | `void __thiscall(void *worker_object)` | Terminate the worker and release its handles and queues. | [Internal Event Routing](../subsystems/internal-events.md#shutdown-and-ownership) |
| `Darkages.exe:0x004BEDF0` | `util_thread_queue_set_wait_timeout` | Cross-subsystem worker | `void __thiscall(void *worker_object, unsigned int timeout_ms)` | Set the `WaitForMultipleObjects` timeout. | [Internal Event Routing](../subsystems/internal-events.md#worker-and-timer-lifecycle) |
| `Darkages.exe:0x004BEE00` | `util_thread_queue_start` | Cross-subsystem worker | `void __thiscall(void *worker_object)` | Create the derived wait object and resume the worker. | [Internal Event Routing](../subsystems/internal-events.md#construction-and-start) |
| `Darkages.exe:0x004BF250` | `util_thread_queue_worker_loop` | Cross-subsystem worker | `void __thiscall(void *worker_object)` | Wait for work or timeout and invoke derived worker methods. | [Internal Event Routing](../subsystems/internal-events.md#worker-loop) |

## Networking

| Address | Current IDA name | Subsystem | Prototype | Purpose | Detailed page |
|---:|---|---|---|---|---|
| `Darkages.exe:0x0040A664` | `net_c_build_portrait_response` | Network and UI | established `__thiscall` builder | Build client action `0x4F` from the local portrait. | [Networking](../subsystems/networking.md) |
| `Darkages.exe:0x0046CA54` | `net_forward_embedded_client_packet` | Network | established `MapPane` action handler | Forward the client packet embedded in server action `0x4B`. | [Client actions](../protocol/client-actions.md#server-directed-dynamic-send) |
| `Darkages.exe:0x004A32F0` | `net_socket_ctor` | Network | `void *__thiscall(void *this, void *event_sink)` | Construct and initialize the Socket object. | [Networking](../subsystems/networking.md#startup-and-connection) |
| `Darkages.exe:0x004A3570` | `net_c_queue_send` | Network | `void __thiscall(void *this, const uint8_t *packet, int16_t length)` | Copy, terminate, and queue a logical client packet. | [Networking](../subsystems/networking.md#send-path) |
| `Darkages.exe:0x004A36C0` | `net_queue_xor_key` | Network | `void __thiscall(void *this, unsigned int key_length, const uint8_t *key)` | Copy and queue a negotiated XOR key. | [Sequence and XOR Transformation](../protocol/xor-transformation.md#server-controlled-key-and-table) |
| `Darkages.exe:0x004A3700` | `net_queue_xor_table_function` | Network | `void __thiscall(void *this, unsigned int function_index)` | Queue selection of table function 0 through 9. | [Sequence and XOR Transformation](../protocol/xor-transformation.md#server-controlled-key-and-table) |
| `Darkages.exe:0x004A3A74` | `net_s_receive_frames` | Network | `void __thiscall(void *this)` | Extract, decode, and post binary frames. | [Networking](../subsystems/networking.md#receive-decode-and-dispatch) |
| `Darkages.exe:0x004A44D4` | `net_s_decode_packet_body` | Network | `int __stdcall(const uint8_t *src, int src_size, uint8_t *dest)` | Reverse the ordinary server-body XOR transformation. | [Sequence and XOR Transformation](../protocol/xor-transformation.md#server-decoding) |
| `Darkages.exe:0x004A4D34` | `net_read_transport_byte` | Network | `int __thiscall(void *this, uint8_t *out_byte)` | Read a cached byte, refilling with `recv` or `ReadFile`. | [Networking](../subsystems/networking.md#receive-and-event-routing) |
| `Darkages.exe:0x004A54D0` | `net_process_work_item` | Network | `void __thiscall(void *this, int code, void *data, int value)` | Execute queued Socket operations. | [Networking](../subsystems/networking.md#code-level-flow) |
| `Darkages.exe:0x004A59F4` | `net_connect_configured_server` | Network | `void __thiscall(void *this)` | Establish the configured socket and async notification. | [Networking](../subsystems/networking.md#startup-and-connection) |
| `Darkages.exe:0x004A72B4` | `net_c_send_packet_body` | Network | `void __thiscall(void *this, const uint8_t *packet, int16_t length)` | Transform, frame, and call `send`. | [Networking](../subsystems/networking.md#build-encode-and-send) |
| `Darkages.exe:0x004A79E4` | `net_c_encode_packet_body` | Network | `int __stdcall(const uint8_t *src, int src_size, uint8_t *dest)` | Add sequence and apply ordinary client-body XOR passes. | [Sequence and XOR Transformation](../protocol/xor-transformation.md#client-encoding) |
| `Darkages.exe:0x004A8414` | `net_set_xor_key` | Network | `void __stdcall(int length, const uint8_t *key)` | Replace and repeat the runtime XOR key. | [Sequence and XOR Transformation](../protocol/xor-transformation.md#runtime-state-and-defaults) |
| `Darkages.exe:0x004A8590` | `net_build_xor_table` | Network | `void __stdcall(int function_index)` | Generate the 256-entry XOR table. | [Sequence and XOR Transformation](../protocol/xor-transformation.md#table-generators) |

## UI protocol dispatch

| Address | Current IDA name | Subsystem | Prototype | Purpose | Detailed page |
|---:|---|---|---|---|---|
| `Darkages.exe:0x0040B3A0` | `ui_handle_server_request_portrait` | Network and UI | `int __thiscall(void *this, void *event)` | Handle server action `0x49`. | [Server actions](../protocol/server-actions.md) |
| `Darkages.exe:0x0045F780` | `ui_main_menu_handle_server_packet` | Network and UI | `int __thiscall(void *this, void *event)` | Handle initial main-menu server packets. | [Server actions](../protocol/server-actions.md) |
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Network and UI | `int __thiscall(void *this, void *event)` | Dispatch 31 in-game server actions. | [Server actions](../protocol/server-actions.md#mappane-switch-coverage) |
| `Darkages.exe:0x0046B574` | `ui_map_handle_object_direction` | Network and UI | `int __thiscall(void *this, const uint8_t *packet)` | Apply server action `0x11` to a map object's direction. | [SChangeDirection](../protocol/server/017-11-change-direction.md) |
| `Darkages.exe:0x004921F0` | `ui_exit_wait_handle_server_packet` | Network and UI | `int __thiscall(void *this, void *event)` | Complete the subtype-one safe-exit exchange for server action `0x4C`. | [SReconnect](../protocol/server/076-4c-reconnect.md) |
| `Darkages.exe:0x00492310` | `ui_exit_wait_pane_ctor` | UI and network | established `__thiscall` constructor | Create the exit-wait pane and send `CQuit` subtype `0x01`. | [SReconnect](../protocol/server/076-4c-reconnect.md) |
| `Darkages.exe:0x004A2ED0` | `ui_server_select_handle_server_packet` | Network and UI | `int __thiscall(void *this, void *event)` | Deserialize the action `0x56` server table. | [Server actions](../protocol/server-actions.md) |

## Input and Windows integration

| Address | Current IDA name | Subsystem | Prototype | Purpose | Detailed page |
|---:|---|---|---|---|---|
| `Darkages.exe:0x0045BDC0` | `win_main_window_proc` | Win32, UI, input, and network | `LRESULT __stdcall(HWND, UINT, WPARAM, LPARAM)` | Route the 15 explicit `DAClass` messages and delegate all others. | [Input and Windows Events](../subsystems/input-and-events.md#complete-handled-message-table) |
| `Darkages.exe:0x0049DA30` | `render_screen_pane_load_palette` | Rendering and Win32 | `void __thiscall(void *, const char *)` | Load and apply the 256-entry screen palette. | [Input and Windows Events](../subsystems/input-and-events.md#activation-palette-and-shutdown-messages) |
| `Darkages.exe:0x0049E820` | `ui_screen_pane_set_cursor_position` | UI and input | `void __thiscall(void *, int, int)` | Update the 32 by 32 software-cursor rectangle. | [Input and Windows Events](../subsystems/input-and-events.md#mouse-input-and-message-time) |

## Entry requirements

Add a function when it is useful outside its immediate subsystem context. Record the calling convention and types established by the code, and link to the detailed subsystem explanation.
