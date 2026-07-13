# Input and Windows Events

The `DAClass` window procedure is the boundary between the Win32 message queue and the client's asynchronous event system. It handles a fixed set of 15 message values. Keyboard and mouse messages update immediate window or screen state, then enter the EventMan worker queue. Activation, palette, socket, virus-check, and destruction messages remain on the window thread.

## High-level operation

### Message-pump boundary

`app_win_main` blocks in `GetMessageA`, calls `TranslateMessage`, and sends each retrieved message through `DispatchMessageA`. The registered `DAClass` procedure is `win_main_window_proc`. This is an event-driven pump, not the rendering or game-tick loop. Timer work runs on the event-dispatcher worker described in [Internal Event Routing](internal-events.md#worker-and-timer-lifecycle).

The window procedure returns 0 for every message it recognizes. All other messages go to `DefWindowProcA`. This includes `WM_CLOSE`, so the default procedure destroys the window and the resulting `WM_DESTROY` case posts `WM_QUIT`. It also includes `WM_CHAR`, `WM_PAINT`, `WM_SETFOCUS`, `WM_KILLFOCUS`, `WM_TIMER`, and `WM_QUERYNEWPALETTE`. `TranslateMessage` can therefore produce a `WM_CHAR`, but the client has no explicit text-message branch here.

### Complete handled-message table

All handler locations below are inside `Darkages.exe:0x0045BDC0` `win_main_window_proc`.

| Value | Win32 name | Handler address | Payload used | Client action |
|---:|---|---:|---|---|
| `0x0002` | `WM_DESTROY` | `Darkages.exe:0x0045BE77` | None | Call `PostQuitMessage(0)`. |
| `0x001C` | `WM_ACTIVATEAPP` | `Darkages.exe:0x0045C2F8` | `wParam` is the process activation state. | Minimize and deactivate, or check for conflicting software and restore presentation state. |
| `0x0100` | `WM_KEYDOWN` | `Darkages.exe:0x0045C21F` | Scan and extended-key fields from `lParam`. | Queue physical-key work code `0x0A`. |
| `0x0101` | `WM_KEYUP` | `Darkages.exe:0x0045C17A` | Scan and extended-key fields from `lParam`. | Queue key-release work code `0x0B`. |
| `0x0104` | `WM_SYSKEYDOWN` | `Darkages.exe:0x0045C295` | `wParam` is checked only for Alt+F4; the queued key comes from `lParam`. | Apply the conditional Alt+F4 remap, then use the key-down path. |
| `0x0105` | `WM_SYSKEYUP` | `Darkages.exe:0x0045C1F0` | Same interpretation as `WM_SYSKEYDOWN`. | Apply the conditional Alt+F4 remap, then use the key-up path. |
| `0x0200` | `WM_MOUSEMOVE` | `Darkages.exe:0x0045C112` | Low and high words of `lParam`. | Update the software cursor and queue work code 4. |
| `0x0201` | `WM_LBUTTONDOWN` | `Darkages.exe:0x0045C0C4` | Coordinates from `lParam`; `wParam` modifier bits are ignored. | Update the cursor, set left-button state, and queue work code 5. |
| `0x0202` | `WM_LBUTTONUP` | `Darkages.exe:0x0045C07D` | Coordinates from `lParam`; `wParam` modifier bits are ignored. | Update the cursor, clear left-button state, and queue work code 6. |
| `0x0204` | `WM_RBUTTONDOWN` | `Darkages.exe:0x0045C02A` | Coordinates from `lParam`; `wParam` modifier bits are ignored. | Update the cursor, set right-button state, and queue work code 7. |
| `0x0205` | `WM_RBUTTONUP` | `Darkages.exe:0x0045BF9E` | Coordinates from `lParam`; `wParam` modifier bits are ignored. | Update the cursor, clear right-button state, and queue work code 8. |
| `0x020A` | `WM_MOUSEWHEEL` | `Darkages.exe:0x0045BFE8` | Signed high word of `wParam` is the wheel delta. | Update the cursor and queue work code 9. |
| `0x0311` | `WM_PALETTECHANGED` | `Darkages.exe:0x0045C62E` | `wParam` is the window that changed the palette. | Reload and apply `legend.pal` when another window changed the palette. |
| `0x0402` | `WM_USER + 2` | `Darkages.exe:0x0045C2C4` | Low word of `lParam` is a Winsock network event. | Queue socket receive work for `FD_READ`, or disconnect for `FD_CLOSE`. |
| `0x2446` | `WM_USER + 0x2046` | `Darkages.exe:0x0045C412` | Nonzero `wParam` points to a virus-check module-name string; zero selects the scan-failure text. `lParam` is ignored. | Route a virus-check warning to an active UI receiver or a modal `LOD` message box. |

The last two rows are the only application-defined messages recognized by this window procedure.

### Mouse input and message time

Every handled mouse message first performs the same software-cursor update under the critical section at screen-pane offset `+0x534`. The procedure restores the old cursor area with `sub_49D850`, calls `ui_screen_pane_set_cursor_position`, then draws or presents the new cursor area through the entry at `sub_49D120`. This occurs before EventMan input blocking is checked.

Coordinate handling has several exact quirks:

- The low word of `lParam` is zero-extended. Values at least 640 become 639. The apparent negative clamp cannot be reached after zero extension.
- The high word is obtained with a logical shift and is neither sign-extended nor clamped.
- `WM_MOUSEWHEEL` goes through the same coordinate path. Windows supplies wheel coordinates in screen space, but this procedure does not call `ScreenToClient` before treating them like the other mouse coordinates.

Each message then calls `GetMessageTime` and places that 32-bit timestamp in the EventMan work record. Mouse movement allocates and transfers ownership of an eight-byte pair stored internally as `(y, x)`, matching the screen-pane cursor calls. Button messages use the most recently stored coordinates. The worker divides a signed wheel delta by 120 before dispatching its mouse Event.

EventMan field `+0x4B0` bit 0 blocks movement, wheel, button-down, and key-down queueing. Releases are deliberately different: left-button up, right-button up, and key up still queue while input is blocked. Button state is held at `+0x4B4`, with bit 0 for left and bit 1 for right.

The worker recognizes double-clicks itself rather than relying on `WM_LBUTTONDBLCLK` or `WM_RBUTTONDBLCLK`, neither of which is handled. It compares the new down timestamp with its stored timestamp and requires total horizontal plus vertical movement of at most two pixels. It dispatches mouse subtypes 1 and 2 for left single and double down, 3 for left up, 4 and 5 for right single and double down, 6 for right up, and 7 for wheel. Mouse movement uses subtype 0.

### Keyboard input

The client queues physical scan codes rather than the `wParam` virtual-key value. Bits 16 through 23 of `lParam` supply the scan byte. If `KF_EXTENDED` bit 24 is set, the procedure adds `0x80` and keeps the low byte. A resulting zero is ignored. Repeat count and previous-key-state fields do not suppress repeated key-down messages.

```c
uint8_t input_scan_code(uint32_t key_data)
{
    uint8_t scan_code = (uint8_t)(key_data >> 16);

    if ((key_data & 0x01000000U) != 0) {
        scan_code = (uint8_t)(scan_code + 0x80U);
    }

    return scan_code;
}
```

`WM_SYSKEYDOWN` and `WM_SYSKEYUP` contain one special path. If `wParam` is `VK_F4`, the Alt-context bit 29 is set, and `Darkages.exe:0x0043DE20` returns a nonnull UI object, the code rewrites the scan byte to `0x10`, clears the extended and Alt-context bits, and preserves selected high transition bits. The down and up messages then continue through their ordinary paths. The code establishes the remap but does not establish its original design intent.

On the EventMan worker, `event_dispatch_key_down` marks the scan code pressed, applies the client's scan-code mapping and modifier tables, and dispatches key subtype 8 when the mapped value is nonzero. `event_update_key_up` clears pressed and modifier state but does not dispatch a key-up Event to panes.

### Activation, palette, and shutdown messages

The application-level activation message is `WM_ACTIVATEAPP`, not `WM_ACTIVATE`. A zero `wParam` invokes a render-object virtual method, minimizes the window with `SW_MINIMIZE`, emits a 2000 Hz beep for 100 ms, and clears `app_window_active`. A nonzero `wParam` repeats the `Brothers Speeder` top-level-window check. A match sets `app_shutdown_requested`, posts quit, and posts `WM_CLOSE`. Otherwise the client restores a 640 by 480, 8-bit presentation configuration, uses `SW_RESTORE`, sets `app_window_active`, and calls `ui_screen_pane_activate`.

`WM_PALETTECHANGED` is ignored when the sender in `wParam` is the main window or when no screen pane exists. Otherwise `render_screen_pane_load_palette` reads 256 RGB entries from `legend.pal`, preserves the system-reserved palette entries at both ends, applies the palette, and marks the screen palette dirty. The procedure then calls the small screen-pane apply entry at `Darkages.exe:0x0049D0E0`.

`WM_DESTROY` only posts `WM_QUIT`. It does not perform subsystem cleanup itself. The message loop returns to `app_win_main`, which calls `app_shutdown` as described in [Startup and Shutdown](../lifecycle/startup.md#normal-and-abnormal-shutdown).

### Application-defined messages

`WM_USER + 2` is registered by `net_connect_configured_server` through `WSAAsyncSelect` with `FD_READ | FD_CLOSE`. `win_main_window_proc` uses only `LOWORD(lParam)`: `FD_READ` calls `net_queue_receive`, and `FD_CLOSE` calls `net_handle_disconnect`. Other network-event values are ignored. The procedure also ignores the socket in `wParam` and the `WSAGETSELECTERROR` value in the high word of `lParam`. The receive and framing path is documented in [Networking](networking.md#connection-and-windows-notification).

`WM_USER + 0x2046` belongs to the process/module virus-check worker. `app_virus_check_module_callback` copies the first reported module name into the 256-byte `app_virus_report_module` buffer and posts the message with that pointer in `wParam`. The window procedure formats it as a quoted name. A zero `wParam` instead inserts `app_virus_check_failure_detail` into a Korean format string whose meaning is, "Could not perform the virus scan. ( %s ) Would you like to continue?"

The report is sent to the first available target in this order: the object returned by `sub_4AD130`, `dword_4F51AC`, then `receiver`. The procedure allocates a `0x554`-byte report object and initializes it with `sub_498CE0`. If no target exists, it calls `MessageBoxA` with caption `LOD` and type `MB_OK`. The subsequent `IDCANCEL` shutdown test is not reachable through an ordinary `MB_OK` message box, although the code contains it.

## Code-level flow

### Dispatch and default processing

```text
app_win_main
  -> GetMessageA
  -> TranslateMessage
  -> DispatchMessageA
      -> win_main_window_proc
          -> explicit handler for one of 15 message values
          -> DefWindowProcA for every other value
```

`Darkages.exe:0x0045BDC0` `win_main_window_proc` is a direct comparison chain rather than a jump table. Its four parameters have the normal `WNDPROC` contract. Every explicit branch returns zero.

### Mouse queue and worker path

```text
win_main_window_proc
  -> sub_49D850
  -> ui_screen_pane_set_cursor_position
  -> sub_49D120
  -> event_manager_get_instance
  -> GetMessageTime
  -> event_queue_mouse_*                  work code 4 through 9
      -> util_thread_queue posting path
          -> event_process_work_item
              -> event_dispatch_mouse_*   Event subtype 0 through 7
                  -> event dispatcher pane routing
```

| Windows input | Queue function | Work code | Worker handler | Result |
|---|---|---:|---|---|
| Move | `event_queue_mouse_move` | 4 | `event_dispatch_mouse_move` | Mouse subtype 0 with copied coordinates. |
| Left down | `event_queue_left_button_down` | 5 | `event_dispatch_left_button_down` | Mouse subtype 1 or double-click subtype 2. |
| Left up | `event_queue_left_button_up` | 6 | `event_dispatch_left_button_up` | Mouse subtype 3. |
| Right down | `event_queue_right_button_down` | 7 | `event_dispatch_right_button_down` | Mouse subtype 4 or double-click subtype 5. |
| Right up | `event_queue_right_button_up` | 8 | `event_dispatch_right_button_up` | Mouse subtype 6. |
| Wheel | `event_queue_mouse_wheel` | 9 | `event_dispatch_mouse_wheel` | Mouse subtype 7 with `wheel_delta / 120`. |

### Keyboard queue and worker path

```text
WM_KEYDOWN or WM_SYSKEYDOWN
  -> scan-code normalization
  -> event_queue_key_down                 work code 0x0A
      -> event_process_work_item
          -> event_dispatch_key_down      key subtype 8 when mapping succeeds

WM_KEYUP or WM_SYSKEYUP
  -> scan-code normalization
  -> event_queue_key_up                   work code 0x0B
      -> event_process_work_item
          -> event_update_key_up          state update, no pane Event
```

`event_dispatch_key_down` stores pressed state in the per-scan-code table beginning at EventMan offset `+0x390`. It combines the active modifier byte at `+0x490` with mapping tables at `+0x90`, `+0x190`, and `+0x290`. One internal mapped value, `0x88` with state bit 0 set, also sets `app_shutdown_requested` and posts `WM_CLOSE` before the key Event is dispatched.

### Socket and virus-check messages

```text
net_connect_configured_server
  -> WSAAsyncSelect(..., WM_USER + 2, FD_READ | FD_CLOSE)
      -> win_main_window_proc
          -> net_queue_receive
          -> net_handle_disconnect

app_virus_check_thread
  -> module scanner callback
      -> app_virus_check_module_callback
          -> PostMessageA(..., WM_USER + 0x2046,
                          app_virus_report_module, 0)
              -> win_main_window_proc
                  -> active UI report receiver
                  -> MessageBoxA fallback
```

The module callback posts only the first infected-module name because it first requires `app_virus_report_module[0] == 0`. It also clears the virus-check continuation flag after posting, which causes the worker's loaded scanner callbacks to stop their traversal.

## Function table

| Address | Current IDA name | Prototype | Purpose | Call relationships and notes |
|---:|---|---|---|---|
| `Darkages.exe:0x00432BD0` | `event_queue_mouse_move` | `void __thiscall(void *event_manager_object, int mouse_y, int mouse_x, unsigned int message_time)` | Copy coordinates and queue mouse-move work. | Called only by `win_main_window_proc`; work code 4; blocked by `+0x4B0` bit 0. |
| `Darkages.exe:0x00432C50` | `event_queue_left_button_down` | `void __thiscall(void *event_manager_object, unsigned int message_time)` | Set left-button state and queue button-down work. | Work code 5; blocked by `+0x4B0` bit 0. |
| `Darkages.exe:0x00432C90` | `event_queue_left_button_up` | `void __thiscall(void *event_manager_object, unsigned int message_time)` | Clear left-button state and queue release work. | Work code 6; not suppressed by input blocking. |
| `Darkages.exe:0x00432CC0` | `event_queue_right_button_down` | `void __thiscall(void *event_manager_object, unsigned int message_time)` | Set right-button state and queue button-down work. | Work code 7; blocked by `+0x4B0` bit 0. |
| `Darkages.exe:0x00432D00` | `event_queue_right_button_up` | `void __thiscall(void *event_manager_object, unsigned int message_time)` | Clear right-button state and queue release work. | Work code 8; not suppressed by input blocking. |
| `Darkages.exe:0x00432D30` | `event_queue_mouse_wheel` | `void __thiscall(void *event_manager_object, int wheel_delta, unsigned int message_time)` | Queue the signed wheel delta and message time. | Work code 9; blocked by `+0x4B0` bit 0. |
| `Darkages.exe:0x00432D60` | `event_queue_key_down` | `void __thiscall(void *event_manager_object, unsigned char scan_code, unsigned int message_time)` | Queue physical-key-down work. | Work code `0x0A`; blocked by `+0x4B0` bit 0. |
| `Darkages.exe:0x00432D90` | `event_queue_key_up` | `void __thiscall(void *event_manager_object, unsigned char scan_code, unsigned int message_time)` | Queue physical-key-release work. | Work code `0x0B`; not suppressed by input blocking. |
| `Darkages.exe:0x00433060` | `event_manager_get_instance` | `void *__cdecl(void)` | Return the EventMan singleton. | Asserts that `event_manager_instance` is nonnull; used for every queued input message. |
| `Darkages.exe:0x00433434` | `event_dispatch_mouse_move` | `void __thiscall(void *event_manager_object, int mouse_y, int mouse_x, unsigned int message_time)` | Dispatch mouse subtype 0. | Called by `event_process_work_item` for work code 4. |
| `Darkages.exe:0x00433504` | `event_dispatch_left_button_down` | `void __thiscall(void *event_manager_object, unsigned int message_time)` | Dispatch left single- or double-button-down Event. | Called for work code 5; subtypes 1 and 2. |
| `Darkages.exe:0x004336C4` | `event_dispatch_left_button_up` | `void __thiscall(void *event_manager_object, unsigned int message_time)` | Dispatch left-button release. | Called for work code 6; subtype 3. |
| `Darkages.exe:0x00433794` | `event_dispatch_right_button_down` | `void __thiscall(void *event_manager_object, unsigned int message_time)` | Dispatch right single- or double-button-down Event. | Called for work code 7; subtypes 4 and 5. |
| `Darkages.exe:0x00433944` | `event_dispatch_right_button_up` | `void __thiscall(void *event_manager_object, unsigned int message_time)` | Dispatch right-button release. | Called for work code 8; subtype 6. |
| `Darkages.exe:0x00433A14` | `event_dispatch_mouse_wheel` | `void __thiscall(void *event_manager_object, int wheel_delta, unsigned int message_time)` | Normalize and dispatch wheel movement. | Called for work code 9; divides delta by 120 and emits subtype 7. |
| `Darkages.exe:0x00433AB4` | `event_dispatch_key_down` | `void __thiscall(void *event_manager_object, unsigned char scan_code, unsigned int message_time)` | Update key state and dispatch a mapped key Event. | Called for work code `0x0A`; can also request `WM_CLOSE`. |
| `Darkages.exe:0x00433C34` | `event_update_key_up` | `void __thiscall(void *event_manager_object, unsigned char scan_code, unsigned int message_time)` | Clear pressed and modifier state. | Called for work code `0x0B`; does not dispatch a pane Event. |
| `Darkages.exe:0x0045BDC0` | `win_main_window_proc` | `LRESULT __stdcall(HWND window, UINT message, WPARAM wparam, LPARAM lparam)` | Route all explicit `DAClass` messages. | Registered by `app_win_main`; recognizes the 15 values in the table above and delegates all others. |
| `Darkages.exe:0x00498430` | `app_virus_check_module_callback` | `int __cdecl(void *process_context, void *module_context, const char *module_name, FILE *log_stream)` | Report the first infected module to the window thread. | Called through the loaded module scanner; posts `WM_USER + 0x2046`. |
| `Darkages.exe:0x00498780` | `app_virus_check_thread` | `void __cdecl(char *log_filename)` | Run virus checking on a worker thread. | Invokes loaded process and module enumerators, cleans up, then calls `_endthread`. |
| `Darkages.exe:0x0049DA30` | `render_screen_pane_load_palette` | `void __thiscall(void *screen_pane, const char *palette_name)` | Load and apply a 256-entry palette. | `WM_PALETTECHANGED` passes `legend.pal`; other UI transitions also call it. |
| `Darkages.exe:0x0049E820` | `ui_screen_pane_set_cursor_position` | `void __thiscall(void *screen_pane, int mouse_y, int mouse_x)` | Update the 32 by 32 software-cursor rectangle. | Called by the shared mouse-message path while holding the screen-pane critical section. |
