# UI Panes and Registries

The client represents screens, dialogs, controls, and many temporary overlays as `Pane`-derived objects. A pane participates in two independent hierarchies. The Screen composition tree records placement and parent-child drawing relationships. The event-dispatch tree records which panes receive mouse, keyboard, socket-packet, and timer callbacks. A pane may appear in both trees, but the trees have different nodes, payloads, and ownership.

The executable does not expose usable Microsoft C++ class RTTI for these pane types. Enumeration instead starts from the common `Pane` virtual layout. Every confirmed pane-compatible virtual table contains `ui_pane_register_events` at slot `+0x30` and `ui_pane_unregister_events` at `+0x34`. Cross-references to the registration slot identify 173 compatible tables. These include controls and construction-state tables as well as full visible screens, so the count is not a count of 173 independently visible windows. The complete address and handler inventory is in [Pane Virtual Table Inventory](../appendices/ui-pane-vtables.md).

## High-level operation

### Two pane hierarchies

The Screen composition tree answers where a pane is placed and which panes are below it. `ui_screen_registry` is a static pointer to its heap registry. The root node holds `ui_screen_pane`, the persistent 640 by 480 surface pane. Adding a pane supplies a rectangle, an optional pane to place it in front of, and an optional parent pane. Removing a node also removes its child hierarchy.

The event-dispatch tree answers which panes are eligible for input and internal events. It is reached through `event_dispatcher + 0x68`. Mouse events use virtual slot `+0x38`, keyboard events use `+0x3C`, socket-packet events use `+0x40`, and due pane timers use `+0x44`. Event traversal visits a node's children before the node itself. Mouse coordinates are adjusted into pane-relative coordinates during traversal.

The two registrations are deliberately separate. `ui_pane_add_to_screen` does not register event handling, and `ui_pane_register_events` does not add anything to the drawing tree. Constructors and transitions normally perform both operations explicitly.

### Pane lifecycle

`app_initialize` first creates the Screen registry, constructs `ui_screen_pane`, and makes that pane the Screen root. It then creates and starts the event dispatcher and registers the screen pane as the initial event receiver.

The first substantial screen is `TerminalPane`. Its constructor publishes `ui_terminal_pane`, adds the pane below `ui_screen_pane`, registers it in the event hierarchy, and adds its controls. A successful terminal bootstrap packet causes `ui_terminal_handle_server_packet` to remove the terminal child, unregister and remove the terminal pane, create `MainMenuPane`, add and register the main menu, send the bootstrap/version traffic, and delete the terminal object. The terminal destructor clears its static pointer.

`MainMenuPane` publishes `ui_main_menu_pane`. Its server handler can create `ServerSelectDialogPane` as a registered child dialog. The dialog destructor explicitly unregisters the pane and removes it from Screen before destroying its dialog state.

The transition into the game reaches `ui_main_menu_create_game_panes`. It creates the persistent in-game `BackgroundPane` and `MapPane`, publishes `ui_background_pane` and `ui_map_pane`, and builds the larger child-pane graph below them. The map pane becomes the principal socket-packet receiver for in-game server actions.

Temporary controls follow the same rule. The game-buttons owner stores each allocated skill or spell pane in an indexed heap-pointer array, adds it below the owner in Screen, and registers it below the owner in the event tree. Removal deletes the node from both trees, deletes the object, and clears the pointer slot. `TextEdit` Escape and Enter paths also remove both registrations before queueing deferred deletion. Enter calls the class-specific virtual slot `+0x4C` before removal.

The base `ui_pane_dtor` only cancels timers and destroys base members. It does not unregister the object from either tree. Every owning path must therefore remove both registrations before immediate or deferred deletion. The observed order varies by owner, but both removals precede destruction.

### Named persistent panes and handlers

The following panes have established construction, virtual-table, and handler evidence. `M`, `K`, `S`, and `T` mean mouse, keyboard, socket packet, and timer.

| Pane | Static root and object size | Vtable | M handler | K handler | S handler | T handler |
|---|---|---:|---|---|---|---|
| `ScreenPane` | `ui_screen_pane` at `Darkages.exe:0x004F51C8`, `0x580` bytes | `ui_screen_pane_vtable` at `Darkages.exe:0x00524CE0` | `ui_pane_default_mouse_handler` | `ui_screen_pane_handle_key_event` | `ui_pane_default_socket_handler` | `ui_screen_pane_handle_timer` |
| `TerminalPane` | `ui_terminal_pane` at `Darkages.exe:0x004FD640`, `0xF78` bytes | `ui_terminal_pane_vtable` at `Darkages.exe:0x00529000` | `ui_terminal_handle_mouse_event` | `ui_terminal_handle_key_event` | `ui_terminal_handle_server_packet` | `ui_pane_default_timer_handler` |
| `MainMenuPane` | `ui_main_menu_pane` at `Darkages.exe:0x004F51AC`, `0x124` bytes | `ui_main_menu_pane_vtable` at `Darkages.exe:0x005177C0` | `ui_main_menu_handle_mouse_event` | `ui_main_menu_handle_key_event` | `ui_main_menu_handle_server_packet` | `ui_pane_default_timer_handler` |
| `BackgroundPane` | `ui_background_pane` at `Darkages.exe:0x004F51B4`, `0x144` bytes | `ui_background_pane_vtable` at `Darkages.exe:0x00500700` | `ui_background_pane_handle_mouse_event` | `ui_pane_default_key_handler` | `ui_handle_server_request_portrait` | `ui_background_pane_handle_timer` |
| `MapPane` | `ui_map_pane` at `Darkages.exe:0x004F51B0`, `0xF38` bytes | `ui_map_pane_vtable` at `Darkages.exe:0x00519280` | `ui_map_handle_mouse_event` | `ui_map_handle_key_event` | `ui_map_dispatch_server_packet` | `ui_map_handle_timer` |
| `ServerSelectDialogPane` | Dynamic, `0x554` bytes | `ui_server_select_dialog_pane_vtable` at `Darkages.exe:0x00526140` | `ui_server_select_handle_mouse_event` | `Darkages.exe:0x0042AED0` | `ui_server_select_handle_server_packet` | `Darkages.exe:0x0040A0B0` |
| Exit-wait pane | Dynamic | `ui_exit_wait_pane_vtable` at `Darkages.exe:0x005207E0` | `Darkages.exe:0x0042A190` | `Darkages.exe:0x0042AED0` | `ui_exit_wait_handle_server_packet` | `Darkages.exe:0x0040A0B0` |

The socket handler is not a Winsock callback. Socket bytes are decoded and converted to internal Event type 9 before the dispatcher calls pane slot `+0x40`. The packet path is documented in [Networking](networking.md#receive-and-event-routing), while Win32 `WM_*` handling remains in [Input and Windows Events](input-and-events.md).

### Runtime observation roots

The documented virtual addresses use image base `0x00400000`. A runtime tool that resolves a loaded module should use `loaded_module_base + RVA` rather than assuming the preferred base.

| VA | RVA | Current IDA name | Runtime meaning | Lifetime and nullability |
|---:|---:|---|---|---|
| `Darkages.exe:0x004F51AC` | `0x000F51AC` | `ui_main_menu_pane` | Pointer to the heap `MainMenuPane`. | Non-null during the main-menu phase; cleared during transitions and teardown. |
| `Darkages.exe:0x004F51B0` | `0x000F51B0` | `ui_map_pane` | Pointer to the heap `MapPane`. | Published after game-pane creation; cleared during game teardown. |
| `Darkages.exe:0x004F51B4` | `0x000F51B4` | `ui_background_pane` | Pointer to the heap `BackgroundPane`. | Published with the game UI; cleared during game teardown. |
| `Darkages.exe:0x004F51C8` | `0x000F51C8` | `ui_screen_pane` | Pointer to the persistent heap `ScreenPane`. | Non-null after successful initialization until application shutdown. |
| `Darkages.exe:0x004F51CC` | `0x000F51CC` | `ui_screen_registry` | Pointer to the heap Screen hierarchy list. | Non-null after Screen initialization until application shutdown. |
| `Darkages.exe:0x004F51D0` | `0x000F51D0` | `event_dispatcher` | Pointer to the dispatcher; `+0x68` points to its pane hierarchy. | Non-null after event initialization until dispatcher shutdown. |
| `Darkages.exe:0x004FD640` | `0x000FD640` | `ui_terminal_pane` | Pointer to the heap `TerminalPane`. | Published by the constructor and cleared by the destructor. |

These roots are useful object discriminators as well as pointer chains. A live object begins with its current virtual-table pointer. Comparing that pointer against the [vtable inventory](../appendices/ui-pane-vtables.md) identifies compatible panes without relying on absent RTTI.

External readers can race Screen changes, event registration, and deferred deletion. The Screen add and remove paths do not acquire a visible lock around the packed list. A practical snapshot should read the list pointer, count, array pointer, and stride, copy the nodes, then re-read those four values and discard the snapshot if they changed. Each pane pointer should also be checked for readability and a known vtable before reading fields. Writes are more hazardous because pane fields, both registries, timers, and ownership must remain consistent.

## Code-level flow

### Common Pane layout and virtual slots

`ui_pane_ctor` constructs the common Pane base and installs `ui_pane_vtable`. The following fields and slots are directly established.

| Pane offset | Width | Established meaning |
|---:|---:|---|
| `+0x0000` | 4 | Current virtual-table pointer. |
| `+0x0008` | 4 | Last propagated Screen or dispatcher error. |
| `+0x00A8` | 16 | Pane bounds `RECT`. |
| `+0x00B0` | 1 | Enabled state; set and cleared by the Pane activation methods. |
| `+0x00B1` | 1 | Constructor flag copied from the first stack argument; exact meaning is not yet established. |
| `+0x00B2` | 1 | Constructor flag copied from the second stack argument; exact meaning is not yet established. |
| `+0x00B3` | 1 | Pane flag copied into the Screen node payload. |
| `+0x00F0` | 1 | Capture-related state checked before Screen removal. |

| Vtable offset | Role | Base implementation |
|---:|---|---|
| `+0x28` | Add pane to Screen composition | `ui_pane_add_to_screen` |
| `+0x2C` | Remove pane from Screen composition | `ui_pane_remove_from_screen` |
| `+0x30` | Register event receiver | `ui_pane_register_events` |
| `+0x34` | Unregister event receiver | `ui_pane_unregister_events` |
| `+0x38` | Mouse Event handler, Event types 0 through 7 | `ui_pane_default_mouse_handler` |
| `+0x3C` | Keyboard Event handler, Event type 8 | `ui_pane_default_key_handler` |
| `+0x40` | Socket-packet Event handler, Event type 9 | `ui_pane_default_socket_handler` |
| `+0x44` | Pane timer callback | `ui_pane_default_timer_handler` |
| `+0x48` | Additional class-specific handler | Base is `nullsub_6`; the generic role is not yet established. |
| `+0x4C` | Optional class-specific handler | Absent in the base table; `TextEdit` calls its override on Enter. |

### Packed hierarchy representation

Both registries use `ui_hierarchy_list_ctor`, which adds an 11-byte hierarchy header to the requested payload size. The result is a packed, commonly unaligned node array.

| List offset | Width | Meaning |
|---:|---:|---|
| `+0x0C` | 4 | Node stride. |
| `+0x14` | 4 | Node count. |
| `+0x18` | 4 | Pointer to the contiguous packed node array. |
| `+0x1C` | 4 | Root or parent node pointer. Screen stores its separately allocated root here. |

| Node offset | Width | Meaning |
|---:|---:|---|
| `+0x00` | 4 | Parent node pointer. |
| `+0x04` | 4 | Child hierarchy-list pointer, or null. |
| `+0x08` | variable | Registry-specific payload. Both mapped registries begin the payload with the pane pointer. |

Screen requests a `0x34`-byte payload, producing stride `0x3F`. The pane pointer is at node `+0x08`, its `RECT` is at `+0x0C`, and copied pane flags begin at `+0x1C`. The root pointer stored at `[ui_screen_registry + 0x1C]` addresses a node whose child-list pointer is at root `+0x04` and whose pane at root `+0x08` is `ui_screen_pane`.

The event dispatcher requests a four-byte payload, producing stride `0x0F`. Each event node therefore contains only its hierarchy links and the pane pointer at `+0x08`.

The exact external pointer chains are:

```c
screen_list = read_process_u32(module_base + 0x000f51cc);
screen_root = read_process_u32(screen_list + 0x1c);
screen_children = read_process_u32(screen_root + 0x04);

dispatcher_object = read_process_u32(module_base + 0x000f51d0);
event_list = read_process_u32(dispatcher_object + 0x68);
```

Walk `screen_children` for ordinary Screen descendants. The separately allocated Screen root itself can be observed directly at `screen_root`. Walk `event_list` for registered event receivers.

A raw external walk is equivalent to:

```c
static void walk_pane_list(uintptr_t list_address)
{
    uint32_t node_stride;
    uint32_t node_count;
    uintptr_t node_array;
    uint32_t node_index;

    node_stride = read_process_u32(list_address + 0x0c);
    node_count = read_process_u32(list_address + 0x14);
    node_array = read_process_u32(list_address + 0x18);

    for (node_index = 0; node_index < node_count; ++node_index) {
        uintptr_t node_address;
        uintptr_t child_list;
        uintptr_t pane_address;

        node_address = node_array + node_index * node_stride;
        child_list = read_process_u32(node_address + 0x04);
        pane_address = read_process_u32(node_address + 0x08);
        observe_pane(pane_address);

        if (child_list != 0) {
            walk_pane_list(child_list);
        }
    }
}
```

The helper reads 32-bit values from arbitrary byte addresses because the strides `0x3F` and `0x0F` do not preserve four-byte alignment.

### Creation and removal paths

The common add path is:

1. A constructor installs a concrete pane vtable after `ui_pane_ctor` initializes the base.
2. The owner calls vslot `+0x28`, reaching `ui_pane_add_to_screen` and then `ui_screen_add_pane`.
3. `ui_screen_add_pane` validates bounds, resolves optional parent and front panes with `ui_screen_find_pane_node`, copies the Screen payload, and inserts the packed node.
4. The owner calls vslot `+0x30`, reaching `ui_pane_register_events` and `event_dispatcher_register_pane`.

Removal reverses both memberships before destruction. `ui_pane_remove_from_screen` releases capture-related state when necessary and calls `ui_screen_remove_pane`. Event removal reaches `event_dispatcher_unregister_pane`. Representative dynamic paths are `ui_game_buttons_remove_skill_pane`, `ui_game_buttons_remove_spell_pane`, the server-select destructor, and `ui_text_edit_handle_key_event`.

`ui_screen_find_pane_node` recursively returns the containing list and node index. `ui_screen_get_pane_origin` walks parent links to accumulate screen coordinates. These functions make the Screen hierarchy useful for both rendering and runtime inspection.

## Function table

| Address | Current IDA name | Prototype | Purpose | Call relationships and notes |
|---:|---|---|---|---|
| `Darkages.exe:0x0040A5A0` | `ui_pane_default_key_handler` | established Pane virtual handler | Default keyboard Event handler. | Base vtable slot `+0x3C`; inherited by panes without a keyboard override. |
| `Darkages.exe:0x0040A5B0` | `ui_background_pane_ctor` | established `__thiscall` constructor | Construct the in-game background pane. | Called by `ui_main_menu_create_game_panes`; object size is `0x144`. |
| `Darkages.exe:0x0040B460` | `ui_pane_default_mouse_handler` | established Pane virtual handler | Default mouse Event handler. | Base vtable slot `+0x38`. |
| `Darkages.exe:0x0040B470` | `ui_pane_default_socket_handler` | established Pane virtual handler | Default socket Event handler. | Base vtable slot `+0x40`. |
| `Darkages.exe:0x0040EA50` | `ui_pane_default_timer_handler` | established Pane timer handler | Default timer callback. | Base vtable slot `+0x44`. |
| `Darkages.exe:0x00431150` | `event_dispatcher_register_pane` | `void __thiscall(void *, void *, int, int)` | Insert a pane into the event hierarchy. | Reached directly for the root and through Pane vslot `+0x30`. |
| `Darkages.exe:0x004311B0` | `event_dispatcher_unregister_pane` | `void __thiscall(void *, void *)` | Remove a pane from the event hierarchy. | Reached through Pane vslot `+0x34` and several explicit cleanup paths. |
| `Darkages.exe:0x00431D54` | `event_dispatch_hierarchy` | `int __thiscall(void *, void *, void *)` | Traverse child-first and call the event-specific pane slot. | Selects `+0x38`, `+0x3C`, or `+0x40` for Event types 0 through 9. |
| `Darkages.exe:0x004325D0` | `event_is_mouse` | `int __thiscall(void *event)` | Test Event type 0 through 7. | Used by `event_dispatch_hierarchy`. |
| `Darkages.exe:0x004325F0` | `event_is_keyboard` | `int __thiscall(void *event)` | Test Event type 8. | Used by `event_dispatch_hierarchy`. |
| `Darkages.exe:0x00442C04` | `ui_game_buttons_remove_skill_pane` | established `__thiscall` removal method | Remove, unregister, delete, and clear an indexed skill pane. | Counterpart to `ui_game_buttons_create_skill_pane`. |
| `Darkages.exe:0x00442D14` | `ui_game_buttons_create_skill_pane` | established `__thiscall` creation method | Allocate and register an indexed `0x294`-byte skill pane. | Stores the heap pointer in its owner. |
| `Darkages.exe:0x00443E14` | `ui_game_buttons_remove_spell_pane` | established `__thiscall` removal method | Remove, unregister, delete, and clear an indexed spell pane. | Counterpart to `ui_game_buttons_create_spell_pane`. |
| `Darkages.exe:0x00443F24` | `ui_game_buttons_create_spell_pane` | established `__thiscall` creation method | Allocate and register an indexed `0x214`-byte spell pane. | Stores the heap pointer in its owner. |
| `Darkages.exe:0x0044E4D0` | `ui_hierarchy_list_ctor` | `void *__thiscall(void *list, int payload_size)` | Construct a packed hierarchy list. | Produces stride `payload_size + 0x0B`. |
| `Darkages.exe:0x0044F4F0` | `ui_hierarchy_remove_node` | established `__thiscall` removal method | Remove one packed node and its child list. | Shared by Screen composition and event dispatch. |
| `Darkages.exe:0x004594A0` | `ui_hierarchy_get_node` | `void *__thiscall(void *list, int index)` | Return a checked packed-node address. | Computes `array + index * stride`. |
| `Darkages.exe:0x0045D930` | `ui_main_menu_pane_ctor` | established `__thiscall` constructor | Construct and publish `MainMenuPane`. | Called by terminal completion and the in-game return path. |
| `Darkages.exe:0x0045FA44` | `ui_main_menu_create_game_panes` | established `__thiscall` transition method | Build and publish the in-game pane graph. | Constructs BackgroundPane, MapPane, and child panes. |
| `Darkages.exe:0x004658C0` | `ui_map_pane_ctor` | established `__thiscall` constructor | Construct `MapPane`. | Called for a `0xF38`-byte allocation during game UI creation. |
| `Darkages.exe:0x00498F20` | `ui_screen_registry_ctor` | `void *__thiscall(void *registry)` | Construct the Screen composition registry. | Requests a `0x34`-byte hierarchy payload. |
| `Darkages.exe:0x00498F40` | `ui_screen_add_pane` | established `__thiscall` add method | Validate and insert a Screen pane node. | Called by `ui_pane_add_to_screen`. |
| `Darkages.exe:0x004993C4` | `ui_screen_find_pane_node` | established recursive `__thiscall` search | Find a pane and optionally return its list and index. | Used for parent, sibling, removal, and coordinate queries. |
| `Darkages.exe:0x00499530` | `ui_screen_remove_pane` | established `__thiscall` removal method | Remove a pane node and child hierarchy. | Called by `ui_pane_remove_from_screen`. |
| `Darkages.exe:0x00499ED0` | `ui_screen_get_pane_origin` | established `__thiscall` query | Accumulate a pane's screen-space origin. | Walks Screen hierarchy parent links. |
| `Darkages.exe:0x00492A90` | `ui_pane_dtor` | `void __thiscall(void *pane)` | Cancel timers and destroy the Pane base. | Does not remove either hierarchy registration. |
| `Darkages.exe:0x00492AD0` | `ui_pane_ctor` | `void *__thiscall(void *, unsigned char, unsigned char)` | Construct the common Pane base. | Installs `ui_pane_vtable` and initializes bounds and flags. |
| `Darkages.exe:0x004933C0` | `ui_pane_add_to_screen` | `void __thiscall(void *, const RECT *, void *, void *)` | Add this pane to Screen composition. | Pane vslot `+0x28`; delegates to `ui_screen_add_pane`. |
| `Darkages.exe:0x00493480` | `ui_pane_remove_from_screen` | `void __thiscall(void *)` | Remove this pane from Screen composition. | Pane vslot `+0x2C`; delegates to `ui_screen_remove_pane`. |
| `Darkages.exe:0x00493530` | `ui_pane_register_events` | `void __thiscall(void *, void *, int)` | Register this pane for event traversal. | Pane vslot `+0x30`; the signature xrefs enumerate compatible vtables. |
| `Darkages.exe:0x004935D0` | `ui_pane_unregister_events` | `void __thiscall(void *)` | Unregister this pane from event traversal. | Pane vslot `+0x34`. |
| `Darkages.exe:0x004A1B04` | `ui_server_select_dialog_pane_dtor` | established `__thiscall` destructor | Unregister and remove the server-select dialog. | Restores main-menu server-selection state before base destruction. |
| `Darkages.exe:0x004A2740` | `ui_server_select_dialog_pane_ctor` | established `__thiscall` constructor | Construct, add, and register the server-select dialog. | Called by `ui_main_menu_handle_server_packet`. |
| `Darkages.exe:0x004AC944` | `ui_terminal_pane_dtor` | established `__thiscall` destructor | Clear the terminal root and destroy its child editors. | Calls `ui_pane_dtor` after derived cleanup. |
| `Darkages.exe:0x004ACA00` | `ui_terminal_pane_ctor` | established `__thiscall` constructor | Construct, publish, add, and register `TerminalPane`. | Object size is `0xF78`. |
| `Darkages.exe:0x004AD130` | `ui_get_terminal_pane` | `void *__cdecl(void)` | Return `ui_terminal_pane`. | Direct static-root accessor. |
| `Darkages.exe:0x004AD140` | `ui_terminal_handle_server_packet` | established Pane socket handler | Handle terminal packets and transition to MainMenuPane. | Terminal vtable slot `+0x40`. |
| `Darkages.exe:0x004BE6F4` | `ui_text_edit_handle_key_event` | established Pane keyboard handler | Complete or cancel a temporary text editor. | Removes both registrations and queues deferred deletion. |
