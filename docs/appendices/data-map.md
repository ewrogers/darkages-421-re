# Data Map

This appendix tracks important globals, tables, structures, virtual tables, and persistent buffers.

| Address | Current IDA name | Type or size | Owner | Purpose | Detailed page | Notes |
|---:|---|---|---|---|---|---|
| `Darkages.exe:0x004D64C0` | `app_window_active` | `uint8_t` | Application and Win32 | Records whether the main window is treated as active. | [Startup and Shutdown](../lifecycle/startup.md#message-pump-and-activation) | Set before the message loop and by `WM_ACTIVATEAPP` handling. |
| `Darkages.exe:0x004E0F00` | `net_xor_table` | `uint32_t[256]` | Socket transformation | Generated XOR byte table. | [Sequence and XOR Transformation](../protocol/xor-transformation.md#table-generators) | Each dword repeats one byte four times. Initial contents implement function 0. |
| `Darkages.exe:0x004E32C0` | `event_manager_instance` | `void *` | Events and network | EventMan singleton that converts worker records into Event objects. | [Internal Event Routing](../subsystems/internal-events.md#dispatcher-event-manager-and-pane-hierarchy) | Published by `event_manager_ctor` and started separately from `event_dispatcher`. |
| `Darkages.exe:0x004F51AC` | `ui_main_menu_pane` | `void *`, object `0x124` bytes | UI | Current heap `MainMenuPane`. | [UI Panes and Registries](../subsystems/ui-panes.md#runtime-observation-roots) | Non-null during the main-menu phase and cleared on transition or teardown. RVA `0x000F51AC`. |
| `Darkages.exe:0x004F51B0` | `ui_map_pane` | `void *`, object `0xF38` bytes | UI | Current heap `MapPane`. | [UI Panes and Registries](../subsystems/ui-panes.md#runtime-observation-roots) | Published after game-pane creation and cleared during game teardown. RVA `0x000F51B0`. |
| `Darkages.exe:0x004F51B4` | `ui_background_pane` | `void *`, object `0x144` bytes | UI | Current heap `BackgroundPane`. | [UI Panes and Registries](../subsystems/ui-panes.md#runtime-observation-roots) | Published with the game UI and cleared during game teardown. RVA `0x000F51B4`. |
| `Darkages.exe:0x004F51B8` | `event_deferred_delete_queue` | `void *` | Events | Holds objects awaiting deletion by the dispatcher tick. | [Internal Event Routing](../subsystems/internal-events.md#periodic-tick-and-deferred-deletion) | Created during initialization, drained from last element to first, and deleted after the dispatcher. |
| `Darkages.exe:0x004F51C4` | `app_config` | `void *` | Application | Global configuration object. | [Startup and Shutdown](../lifecycle/startup.md#subsystem-initialization) | Constructed from `Darkages.cfg` when `English.nfo` exists, otherwise from `Legend.cfg`. |
| `Darkages.exe:0x004F51C8` | `ui_screen_pane` | `void *` | UI and rendering | Global 640 by 480, 8-bit screen pane. | [Startup and Shutdown](../lifecycle/startup.md#subsystem-initialization) | Registered with the event dispatcher and reactivated around the message loop. |
| `Darkages.exe:0x004F51CC` | `ui_screen_registry` | `void *` | UI | Heap Screen composition hierarchy. | [UI Panes and Registries](../subsystems/ui-panes.md#packed-hierarchy-representation) | Stride `0x3F`; list fields are at `+0x0C`, `+0x14`, `+0x18`, and `+0x1C`. RVA `0x000F51CC`. |
| `Darkages.exe:0x004F51D0` | `event_dispatcher` | `void *` | Events | Global event dispatcher and timer-worker object. | [Internal Event Routing](../subsystems/internal-events.md#worker-and-timer-lifecycle) | Created and started by `app_initialize`; destroyed by `app_shutdown`. |
| `Darkages.exe:0x004F51D4` | `app_error_code` | `int32_t` | Application | Process-wide subsystem initialization error. | [Startup and Shutdown](../lifecycle/startup.md#initialization-and-failure-ownership) | Checked after most constructors and before the message loop. |
| `Darkages.exe:0x004F51DC` | `app_main_window` | `HWND` | Application and Win32 | Main `DAClass` window handle. | [Startup and Shutdown](../lifecycle/startup.md#window-creation) | A null result from `CreateWindowExA` is stored but not rejected. |
| `Darkages.exe:0x004F51E0` | `app_instance` | `HINSTANCE` | Application and Win32 | Saved process instance handle. | [Startup and Shutdown](../lifecycle/startup.md#process-entry-and-window-class) | Assigned by `app_win_main` after class registration. |
| `Darkages.exe:0x004F51F4` | `app_shutdown_requested` | `uint8_t` | Application | Requests termination of the blocking message loop. | [Startup and Shutdown](../lifecycle/startup.md#message-pump-and-activation) | Observed only after `GetMessageA` returns. Several writers also post a message to wake the loop. |
| `Darkages.exe:0x004FA0E0` | `app_single_instance_mutex` | `HANDLE` | Application | Handle for `Nexon.SingleInstance`. | [Startup and Shutdown](../lifecycle/startup.md#preflight-checks-and-single-instance-enforcement) | Closed and cleared only after normal `app_shutdown`. |
| `Darkages.exe:0x004FD320` | `app_virus_check_failure_detail` | `char[256]` | Application checks | Detail inserted into the virus-scan failure prompt. | [Input and Windows Events](../subsystems/input-and-events.md#application-defined-messages) | Used when `WM_USER + 0x2046` arrives with zero `wParam`. |
| `Darkages.exe:0x004FD420` | `app_virus_report_module` | `char[256]` | Application checks | First infected-module name reported to the window thread. | [Input and Windows Events](../subsystems/input-and-events.md#application-defined-messages) | Its address is posted in `wParam` for `WM_USER + 0x2046`. |
| `Darkages.exe:0x004FD5A0` | `net_xor_key` | key storage, at least 13 bytes | Socket transformation | Runtime repeating XOR key. | [Sequence and XOR Transformation](../protocol/xor-transformation.md#runtime-state-and-defaults) | Constructor default is ASCII `NexonInc.`. Negotiated updates require at most 9 bytes. |
| `Darkages.exe:0x004FD5AC` | `net_xor_key_length` | `int32_t` | Socket transformation | Number of bytes in the active key. | [Sequence and XOR Transformation](../protocol/xor-transformation.md#runtime-state-and-defaults) | Default is 9. |
| `Darkages.exe:0x004FD5B0` | `net_xor_key_repeated` | `uint8_t[48]` | Socket transformation | Four adjacent copies of the key. | [Sequence and XOR Transformation](../protocol/xor-transformation.md#runtime-state-and-defaults) | Supports optimized four-byte XOR loops. |
| `Darkages.exe:0x004FD5E0` | `net_c_send_sequence` | `uint8_t` | Socket transformation | Sequence inserted after ordinary transformed client actions. | [Sequence and XOR Transformation](../protocol/xor-transformation.md#runtime-state-and-defaults) | Zero-initialized at process load and incremented modulo 256 only by the transformed client encoder; cleartext bypass actions do not consume it. |
| `Darkages.exe:0x004FD640` | `ui_terminal_pane` | `void *`, object `0xF78` bytes | UI | Current heap `TerminalPane`. | [UI Panes and Registries](../subsystems/ui-panes.md#runtime-observation-roots) | Published by the constructor, returned by `ui_get_terminal_pane`, and cleared by the destructor. RVA `0x000FD640`. |
| `Darkages.exe:0x00500700` | `ui_background_pane_vtable` | virtual table | UI | Concrete BackgroundPane handler table. | [UI Panes and Registries](../subsystems/ui-panes.md#named-persistent-panes-and-handlers) | Socket slot `+0x40` handles server portrait requests. |
| `Darkages.exe:0x0050D180` | `event_dispatcher_vtable` | `void *[8]` | Events | Virtual table for pane dispatch, work processing, wait handling, and periodic ticks. | [Internal Event Routing](../subsystems/internal-events.md#dispatcher-object-fields) | Slot `+0x10` points to `event_dispatcher_tick`. |
| `Darkages.exe:0x0050D720` | `event_manager_vtable` | `void *[8]` | Events and network | EventMan worker and dispatch virtual table. | [Internal Event Routing](../subsystems/internal-events.md#network-event-handoff) | Its work-item processing path reaches `event_process_work_item`. |
| `Darkages.exe:0x005177C0` | `ui_main_menu_pane_vtable` | virtual table | `MainMenuPane` | Includes mouse, keyboard, and server-packet handlers. | [UI Panes and Registries](../subsystems/ui-panes.md#named-persistent-panes-and-handlers) | Packet handler is `ui_main_menu_handle_server_packet`. |
| `Darkages.exe:0x00519280` | `ui_map_pane_vtable` | virtual table | `MapPane` | Includes mouse, keyboard, timer, and in-game server handlers. | [UI Panes and Registries](../subsystems/ui-panes.md#named-persistent-panes-and-handlers) | Socket Event virtual slot `+0x40` points to `ui_map_dispatch_server_packet`. |
| `Darkages.exe:0x005207E0` | `ui_exit_wait_pane_vtable` | virtual table | exit-wait pane | Includes the server action `0x4C` handler used by the orderly-exit exchange. | [SReconnect](../protocol/server/076-4c-reconnect.md) | Installed by `ui_exit_wait_pane_ctor`; the source-level C++ class name is not yet established. |
| `Darkages.exe:0x005213A0` | `ui_pane_vtable` | virtual table | UI | Base Pane lifecycle and event slots. | [UI Panes and Registries](../subsystems/ui-panes.md#common-pane-layout-and-virtual-slots) | Screen slots are `+0x28/+0x2C`; event registration is `+0x30/+0x34`; handlers begin at `+0x38`. |
| `Darkages.exe:0x00524CA0` | `ui_screen_registry_vtable` | virtual table | UI | Screen composition registry table. | [UI Panes and Registries](../subsystems/ui-panes.md#packed-hierarchy-representation) | Owns packed `0x3F`-byte Screen nodes. |
| `Darkages.exe:0x00524CE0` | `ui_screen_pane_vtable` | virtual table | UI and rendering | Persistent ScreenPane handler table. | [UI Panes and Registries](../subsystems/ui-panes.md#named-persistent-panes-and-handlers) | Keyboard and timer slots have concrete overrides. |
| `Darkages.exe:0x00526140` | `ui_server_select_dialog_pane_vtable` | virtual table | `ServerSelectDialogPane` | Includes the action `0x56` server-list handler. | [UI Panes and Registries](../subsystems/ui-panes.md#named-persistent-panes-and-handlers) | Constructor and deletion diagnostics establish the class name. |
| `Darkages.exe:0x00526940` | `net_socket_vtable` | virtual table | `Socket` | Socket work-item interface. | [Networking](../subsystems/networking.md) | Installed by `net_socket_ctor`. |
| `Darkages.exe:0x00529000` | `ui_terminal_pane_vtable` | virtual table | `TerminalPane` | Includes mouse, keyboard, and terminal server-packet handlers. | [UI Panes and Registries](../subsystems/ui-panes.md#named-persistent-panes-and-handlers) | Installed by `ui_terminal_pane_ctor`. |

## Event dispatcher fields

Offsets below are from the object stored in `event_dispatcher`. The earlier fields belong to its worker-queue base.

| Offset | Width | Working field | Reads and writes |
|---:|---:|---|---|
| `+0x0000` | 4 | `vtable` | Set to `event_dispatcher_vtable`. |
| `+0x000C` | 4 | `wait_timeout_ms` | Set to 1 and passed to `WaitForMultipleObjects`. |
| `+0x0010` | 1 | `wait_handle_count` | Number of entries beginning at `+0x14`. |
| `+0x0014` | variable | `wait_handles` | Handle index 0 is the queued-work notification. |
| `+0x0060` | 4 | `worker_thread` | Created suspended, resumed during initialization, and terminated by the base destructor. |
| `+0x0068` | 4 | `pane_registry` | Registry traversed by internal event dispatch. |
| `+0x006C` | 4 | `multimedia_period_ms` | Passed to `timeBeginPeriod` and `timeEndPeriod`. |
| `+0x0070` | 4 | `current_tick` | Latest 32-bit `timeGetTime` sample and base for relative timer insertion. |
| `+0x0074` | 4 | `next_deadline` | First timer deadline, or `UINT32_MAX` when no timer is pending. |
| `+0x0078` | 4 | `timer_list` | Sorted list with capacity 256 and 20-byte records. |

## Event timer record

| Offset | Width | Meaning |
|---:|---:|---|
| `+0x00` | 4 | Receiver pane pointer. |
| `+0x04` | 4 | Callback identifier. |
| `+0x08` | 4 | Absolute 32-bit `timeGetTime` deadline. |
| `+0x0C` | 4 | First callback payload value. |
| `+0x10` | 4 | Second callback payload value. |

## Event manager input fields

Offsets below are from the object stored in `event_manager_instance`. They are used by the window-message input queue and the EventMan worker.

| Offset | Width | Working field | Reads and writes |
|---:|---:|---|---|
| `+0x006C` | 4 | `mouse_y` | Updated from the first word of internal `(y, x)` mouse-move work and copied into button and wheel Events. |
| `+0x0070` | 4 | `mouse_x` | Updated from the second word of internal `(y, x)` mouse-move work and copied into button and wheel Events. |
| `+0x007C` | 4 | `double_click_time_limit` | A second button down is eligible only when its unsigned elapsed message time is strictly less than this value. |
| `+0x0080` | 1 | `previous_click_button` | Value 1 records a prior left down; value 0 records a prior right down. |
| `+0x0084` | 4 | `previous_click_y` | First coordinate used by internal double-click recognition. |
| `+0x0088` | 4 | `previous_click_x` | Second coordinate used by internal double-click recognition. |
| `+0x008C` | 4 | `previous_click_time` | `GetMessageTime` value for the prior button-down candidate. |
| `+0x0390` | 256 | `scan_code_pressed` | Per-scan-code state; bit 7 is set on key down and cleared on key up. |
| `+0x0490` | 1 | `input_modifier_state` | Combined mouse-button and mapped keyboard modifier bits placed in Events. |
| `+0x04B0` | 4 | `input_block_flags` | Bit 0 suppresses movement, wheel, button-down, and key-down queueing. |
| `+0x04B4` | 4 | `mouse_button_state` | Bit 0 tracks left and bit 1 tracks right at window-message enqueue time. |

## Socket fields used by binary TCP

Offsets below are from the Socket object passed as `this`. They are established by the constructor and the mapped receive and send paths. The complete class layout is not yet named.

| Offset | Width | Working field | Reads and writes |
|---:|---:|---|---|
| `+0x0068` | 4 | `event_sink` | Used as `this` for `event_post_socket_bytes`. |
| `+0x006C` | 4 | `receive_cache` | Points at the object's `0x18000` byte transport cache. |
| `+0x48074` | 4 | `receive_cursor` | Index of the next cached transport byte. |
| `+0x48078` | 4 | `receive_remaining` | Cached bytes remaining after the current byte. |
| `+0x4807C` | 4 | `current_body_length` | Big-endian length assembled from the two frame header bytes. |
| `+0x48084` | 4 | `serial_handle` | Set to `INVALID_HANDLE_VALUE` when unused. |
| `+0x480BC` | 1 | `socket_connected` | Cleared on `recv == 0` and disconnect cleanup. |
| `+0x480C0` | 4 | `socket` | Winsock `SOCKET`; `INVALID_SOCKET` is `0xFFFFFFFF`. |
| `+0x480C5` | `0x10000` | `wire_body_buffer` | Receives body bytes beginning with the server action. |
| `+0x580C8` | 4 | `frame_offset` | Counts length-header and body bytes while a frame is open. |
| `+0x680CC` | at least `0x10000` | `decoded_or_encoded_buffer` | Destination for packet-body XOR transformation. |
| `+0x780CC` | 1 | `transport_mode` | Value 5 selects the normal TCP byte path. |
| `+0x780CD` | 1 | `frame_open` | Set after consuming `0xAA`; cleared on body completion. |
| `+0x780CF` | 1 | `legacy_send_sequence` | Decimal digit sequence used by the printable legacy transport. |
| `+0x780D2` | 1 | `transfer_gate` | When set, only client action `0x10` may be queued. |

## Socket Event fields

The Event object created for a decoded packet has these fields:

| Offset | Width | Meaning | Lifetime |
|---:|---:|---|---|
| `+0x0C` | 1 | Event type, value 9 for a socket packet. | Set before queueing. |
| `+0x14` | 4 | Owned decoded-packet pointer. | Freed by the event manager after pane dispatch. |
| `+0x18` | 4 | Decoded packet size. | Positive when the pointer is present. |

Structures should use `unknown_XX` for an established field location whose meaning remains unknown.
