# Startup and Shutdown

The client uses a conventional Win32 process entry and a blocking `GetMessageA` loop, but its recurring game and timer work does not run in that loop. `WinMain` owns process checks, the main window, subsystem construction, and orderly teardown. A separate internal event worker supplies periodic callbacks. See [Internal Event Routing](../subsystems/internal-events.md#worker-and-timer-lifecycle) for that worker.

## High-level lifecycle

### Process entry and window class

Windows enters the executable through the Microsoft Visual C++ runtime stub. The stub initializes the heap, multithreading runtime, standard I/O, command-line arguments, environment, and C initializers. It derives the command tail and the initial show command from `STARTUPINFOA`, obtains the module handle, and calls `app_win_main`. It always passes a null `previous_instance`, so the window-class registration path is taken in normal execution.

The client registers the ANSI window class `DAClass` with these established properties:

| Property | Value |
|---|---|
| Window procedure | `win_main_window_proc` |
| Class style | `0` |
| Extra class and window bytes | `0` |
| Icon | Resource identifier `126` |
| Cursor | `IDC_ARROW` |
| Background brush | `BLACK_BRUSH` |
| Menu | None |

If the first `RegisterClassA` call fails, the code calls `RegisterClassA` a second time while forming a diagnostic assertion, sets `app_error_code` to 1, and returns 0 from `app_win_main`. The second call does not recover startup.

### Preflight checks and single-instance enforcement

`app_win_main` performs the following checks in order before it creates the window:

1. It examines three values under `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\RunServices`: the unnamed default value, `" .EXE"` with a leading space, and `"UMGR32.EXE"`. The key is requested with `KEY_ALL_ACCESS`. For the first value whose returned data begins with a nonzero byte, the client displays a message box with type `0x44`. Selecting `IDNO` deletes that value and stops startup. Any other response permits startup. Failure to open the key permits startup.
2. It checks a process-name list and two files. A match displays a yes-or-no prompt. Selecting `IDNO` returns 0 and stops startup; the other response continues the scan.
3. It calls `FindWindowExA(NULL, NULL, "#32770", "Brothers Speeder")`. Finding that top-level dialog stops startup without creating the client window.
4. It creates the initially owned mutex `Nexon.SingleInstance`. `ERROR_ALREADY_EXISTS` causes an immediate `_exit(0)`. A mutex creation failure with another error is not treated as fatal, so this check fails open.

The process scan dynamically resolves `CreateToolhelp32Snapshot`, `Process32First`, and `Process32Next` from `Kernel32.DLL`. It runs on Windows 9x, or on the NT platform when the major version is at least 5. Executable paths are compared case-insensitively by suffix against this exact list:

| Entry | Compared process suffix |
|---:|---|
| 0 | `Patch.exe` |
| 1 | `boserve.exe` |
| 2 | `winhwak.exe` |
| 3 | `grcframe.exe` |

The implementation passes a local `PROCESSENTRY32A` to `Process32First` without first assigning its required `dwSize` field. It also does not close the process snapshot handle. Failure to load the library, resolve an entry point, create the snapshot, or begin enumeration skips to the file checks and therefore permits startup unless a later check rejects it.

The file checks use `fopen(..., "rb")` on `<system-directory>\grcframe.exe` and `<windows-directory>\KeyHook.dll`. Each existing file produces its own yes-or-no prompt.

These checks are documented as observed control flow. The runtime strings used for several prompts do not establish a broader intent for the checks.

### Window creation

After the checks pass, `app_win_main` creates a topmost `Dark Ages` window of class `DAClass`. The extended style is `WS_EX_TOPMOST`. The style value `0x90880000` combines `WS_POPUP`, `WS_VISIBLE`, `WS_BORDER`, and `WS_SYSMENU`. Both coordinates are `CW_USEDEFAULT`, and the requested outer size is 642 by 482 pixels. The client later creates a 640 by 480 screen pane for presentation.

The return from `CreateWindowExA` is stored in `app_main_window` but is not validated. The code still calls `SetForegroundWindow`, `ShowWindow`, `UpdateWindow`, and `app_initialize` if the handle is null.

### Subsystem initialization

`app_initialize` runs only after the window creation attempt. Its established setup order is:

1. Fill a 1024-byte global buffer with ASCII spaces.
2. Read physical-memory status and request a process working-set range from half of physical memory to all physical memory. Return values are ignored.
3. Remove the IME context from the main window with `ImmAssociateContext(window, NULL)`.
4. Set the current directory from the executable path parsed out of the raw command line.
5. Set the process priority class to `HIGH_PRIORITY_CLASS`, create the client memory manager, and seed `rand` with `time(NULL)`.
6. Open `Legend.dat`, `seo.dat`, and `khan.dat`. All three must open for that archive set to be accepted. Otherwise the client tries the single fallback archive `DarkAges.dat`.
7. Create the rendering and media singletons, initialize rendering support, and load `Skill.tbl`.
8. Construct the remaining service objects, the sound manager, and configuration. The sound path opened here is `.\Music\1.mp3`.
9. If `English.nfo` exists, construct the configuration from `Darkages.cfg`; otherwise use `Legend.cfg`.
10. Create the deferred-deletion queue, the 640 by 480 8-bit screen pane using `Legend.pal`, the internal event dispatcher, and the separate `event_manager_instance`. Both workers are resumed during this phase.
11. Register the screen pane with the dispatcher, establish the 640 by 480 screen rectangle, load `Darkages.prf` or `Legend.prf` according to configuration byte `+0x8D`, and create the first terminal pane.

Most constructors communicate failure through the process-wide `app_error_code`. `app_initialize` contains staged cleanup branches for partial construction. When it returns, `app_win_main` checks the error code again and returns 0 instead of entering the message loop if it is nonzero.

The missing-archive path is different. It deletes the two early rendering or media singletons, shows two error boxes, posts `WM_QUIT`, and calls `_exit(0)`. The first box labels `GetCommandLineA` output as the current directory, even though it did not call `GetCurrentDirectoryA`. The direct `_exit` bypasses `app_shutdown`.

### Command-line directory parsing

`app_set_working_directory_from_command_line` does not use the `command_line` argument supplied to `app_win_main`. It calls `GetCommandLineA` again and applies this parser:

- If any double quote exists, copy from the second command-line byte through the byte before the last quote.
- Otherwise, copy up to the first space.
- If there is no space, copy the complete command line.
- Find the last backslash in the copied path, terminate the string there, and pass the directory to `SetCurrentDirectoryA`.

The helper ignores the `SetCurrentDirectoryA` result. Its local path buffer is fixed at approximately 256 bytes and the copies have no length check. Because it uses the last quote anywhere in the raw command line and always starts the quoted case at byte 1, quoted arguments after the executable can confuse the result.

### Message pump and activation

Immediately before the first `GetMessageA`, `app_win_main` marks the window active and calls `ui_screen_pane_activate` if the screen pane exists. The message loop is blocking and unfiltered. It does not render or advance timers directly:

```c
for (;;) {
    int get_message_result;

    get_message_result = GetMessageA(&message, NULL, 0, 0);
    if (get_message_result == 0) {
        break;
    }

    if (message.message == WM_QUIT || app_shutdown_requested == 1) {
        app_shutdown_requested = 1;
        break;
    }

    TranslateMessage(&message);
    DispatchMessageA(&message);
}
```

The explicit `WM_QUIT` comparison is normally redundant because `GetMessageA` returns zero for `WM_QUIT`. The code tests only for zero, so a `GetMessageA` error return of `-1` is treated as a message and passed to `TranslateMessage` and `DispatchMessageA`. The shutdown flag is examined only after `GetMessageA` returns, so changing the flag alone does not wake a blocked loop.

`win_main_window_proc` connects the pump to client behavior. Relevant lifecycle cases include:

- `WM_DESTROY` calls `PostQuitMessage(0)` and returns 0.
- An inactive `WM_ACTIVATEAPP` path minimizes the window, beeps, and clears `app_window_active`.
- The active `WM_ACTIVATEAPP` path repeats the `Brothers Speeder` window check. A match sets the shutdown flag and posts quit or close messages. Otherwise it restores the window, sets the active flag, and calls `ui_screen_pane_activate`.
- `WM_USER + 2` routes asynchronous Winsock notifications, as described in [Networking](../subsystems/networking.md#connection-and-windows-notification).
- Keyboard, mouse, palette, and client-specific messages are dispatched in the same procedure. The complete message table and both application-defined messages are documented in [Input and Windows Events](../subsystems/input-and-events.md#complete-handled-message-table). Unknown messages go to `DefWindowProcA`.

Periodic timer processing is independent of this pump. The event worker waits with a 1 millisecond timeout and invokes the dispatcher tick. See [Worker and timer lifecycle](../subsystems/internal-events.md#worker-and-timer-lifecycle).

### Normal and abnormal shutdown

Normal shutdown begins when `GetMessageA` returns zero or the loop observes `app_shutdown_requested`. `app_win_main` waits for a fixed 200 milliseconds, calls `app_shutdown`, closes and clears `app_single_instance_mutex`, and returns `message.wParam`. The CRT entry then passes that result to `_exit`.

For a normal `WM_QUIT`, `message.wParam` is the quit code. If the shutdown flag stops the loop while a different message is current, the function returns that message's `wParam` instead.

`app_shutdown` is an ordered, null-safe deletion sweep. It closes the archive managers first, then destroys the constructed service singletons and global objects. Confirmed objects in the sweep include media and audio support, screen and UI state, configuration and preferences, the event dispatcher, the deferred-deletion queue, network-owned state, and the memory manager. Each owned global is set to null after deletion.

Deleting the event dispatcher releases its pane and timer containers, calls `timeEndPeriod`, and then invokes the worker base destructor. That base destructor uses `TerminateThread`, closes the worker and wait handles, and destroys queued synchronization state. It does not request a cooperative worker exit or join the thread. The full timing teardown is documented in [Internal Event Routing](../subsystems/internal-events.md#shutdown-and-ownership).

Several exits do not run this sweep:

| Exit path | Behavior |
|---|---|
| Window-class or preflight failure | Return 0 from `app_win_main`; no subsystem graph was constructed. |
| Existing `Nexon.SingleInstance` mutex | Call `_exit(0)` immediately. |
| Missing required data archives | Perform limited early cleanup, display errors, post `WM_QUIT`, then call `_exit(0)`. |
| Nonzero initialization error | Use staged cleanup inside `app_initialize`, then return 0 from `app_win_main` without calling the full `app_shutdown`. |
| CRT exception path | Filter the exception and call the CRT exit path with the exception code. |

## Code-level flow

### CRT handoff

1. `Darkages.exe:0x004C62A5` `app_crt_entry` installs the runtime exception frame and initializes the heap, thread runtime, standard I/O, arguments, environment, and C initializers. Heap initialization failure exits with code `0x1C`; multithreading initialization failure exits with code `0x10`.
2. It obtains the command tail from the CRT helper, derives `show_command` from `STARTUPINFOA` when `STARTF_USESHOWWINDOW` is present or uses `SW_SHOWDEFAULT`, and calls `app_win_main(GetModuleHandleA(NULL), NULL, command_tail, show_command)`.
3. A normal return calls `_exit(win_main_result)`. The exception branch calls `__XcptFilter` and then the CRT exception exit routine.

### `app_win_main`

1. Clear `app_main_window`.
2. When `previous_instance == NULL`, assemble and register `DAClass`. A registration failure sets `app_error_code = 1` and returns 0.
3. Save `instance` in `app_instance`.
4. Call `app_check_runservices_entries`. Return 0 from `app_win_main` if it returns 1.
5. Call `app_check_conflicting_software`. Return 0 if it returns 0.
6. Call `win_top_level_window_exists("#32770", "Brothers Speeder")`. Return 0 on a match.
7. Create `Nexon.SingleInstance`. Call `_exit(0)` when `GetLastError()` is `ERROR_ALREADY_EXISTS`.
8. Create and show the main window, set `app_window_active = 1`, and call `app_initialize`.
9. If `app_error_code != 0`, emit the diagnostic and return 0.
10. Activate the screen pane and enter the `GetMessageA` loop.
11. On loop exit, wait 200 milliseconds, call `app_shutdown`, close the mutex, and return the current `MSG.wParam`.

The `command_line` parameter is not read by this function. Directory setup re-reads the complete process command line.

### Initialization and failure ownership

`Darkages.exe:0x0045CCA0` `app_initialize` has no meaningful return value. Construction success is represented by nonnull globals and `app_error_code == 0`. Each major allocation is followed by an error check. Later failure labels delete the objects that were already created, proceeding backward through the partial dependency graph.

The successful path creates `event_dispatcher` with `event_dispatcher_ctor`, calls `util_thread_queue_start` to resume its suspended worker, then constructs `event_manager_instance` with `event_manager_ctor` and starts that worker. It registers `ui_screen_pane` with the dispatcher only after both exist.

The archive failure branch at `Darkages.exe:0x0045CE6C` does not join the common staged cleanup. It destroys the two early graphics or media objects, displays the archive errors, posts a quit message, and reaches `_exit(0)` at `Darkages.exe:0x0045CF31`.

### Window messages and loop termination

`Darkages.exe:0x0045BDC0` `win_main_window_proc` is the only window procedure installed for `DAClass`. `DispatchMessageA` reaches it for queued messages, while some client paths also post messages directly to `app_main_window`. Its `WM_DESTROY` case supplies the normal `WM_QUIT`. Other fatal paths set `app_shutdown_requested` and post a message so the blocking loop can observe the flag.

`Darkages.exe:0x0049D790` `ui_screen_pane_activate` sets a screen-pane active field at `+0x568`, resets a presentation counter to 10, restores the pane's render object through a virtual call, and invokes two screen update helpers. It is called once before the first `GetMessageA` and again after an active `WM_ACTIVATEAPP` transition.

### Teardown

`Darkages.exe:0x0045B8F0` `app_shutdown` checks each singleton or global before calling its deleting destructor. It then clears the corresponding global. The event dispatcher is deleted after the main screen, configuration, preference, and several UI service objects, but before the deferred-deletion queue and final media and memory singletons. No return code is propagated from teardown.

## Function table

| Address | Current IDA name | Prototype | Purpose | Call relationships and notes |
|---:|---|---|---|---|
| `Darkages.exe:0x0041D390` | `app_check_conflicting_software` | `int __cdecl(void)` | Prompt for four process suffixes and two files. | Called by `app_win_main`; 0 stops startup. Toolhelp failures are fail-open. The local `PROCESSENTRY32A.dwSize` is not initialized and the snapshot is not closed. |
| `Darkages.exe:0x0041D690` | `win_top_level_window_exists` | `BOOL __cdecl(LPCSTR class_name, LPCSTR window_title)` | Test for a matching top-level window. | Thin `FindWindowExA(NULL, NULL, ...)` wrapper. Used during initial startup and active-window restoration. |
| `Darkages.exe:0x0041D6C0` | `app_check_runservices_entries` | `int __cdecl(void)` | Examine and optionally delete three `RunServices` values. | Called first by `app_win_main`; 1 means a value was deleted after `IDNO` and startup must stop. |
| `Darkages.exe:0x0045B5F0` | `app_win_main` | `int __stdcall(HINSTANCE instance, HINSTANCE previous_instance, LPSTR command_line, int show_command)` | Own the Win32 lifecycle. | Called only by `app_crt_entry`; calls the checks, creates the mutex and window, initializes subsystems, pumps messages, and shuts down. |
| `Darkages.exe:0x0045B8F0` | `app_shutdown` | `void __cdecl(void)` | Delete the initialized subsystem graph in fixed order. | Called only on the normal message-loop exit path. Null-checks and clears owned globals. |
| `Darkages.exe:0x0045BDC0` | `win_main_window_proc` | `LRESULT __stdcall(HWND window, UINT message, WPARAM wparam, LPARAM lparam)` | Dispatch all `DAClass` window messages. | Registered by `app_win_main`; `WM_DESTROY` posts quit and `WM_ACTIVATEAPP` coordinates active state and screen restoration. See [Input and Windows Events](../subsystems/input-and-events.md). |
| `Darkages.exe:0x0045C840` | `app_set_working_directory_from_command_line` | `void __cdecl(void)` | Set the current directory to the parsed executable directory. | Called near the start of `app_initialize`; re-reads the raw command line and ignores API failure. |
| `Darkages.exe:0x0045CCA0` | `app_initialize` | `void __cdecl(void)` | Construct resources, rendering, audio, configuration, event workers, screen state, and the initial pane. | Called after window creation. Uses `app_error_code` and staged cleanup; the missing-archive branch calls `_exit(0)`. |
| `Darkages.exe:0x0049D790` | `ui_screen_pane_activate` | `void __thiscall(void *screen_pane)` | Restore active screen and presentation state. | Called before the message loop and by the active `WM_ACTIVATEAPP` path. |
| `Darkages.exe:0x004C62A5` | `app_crt_entry` | `void __cdecl(void)` | Initialize the Microsoft C runtime and hand off to `app_win_main`. | PE process entry; passes the result to `_exit` and contains the CRT exception path. |
