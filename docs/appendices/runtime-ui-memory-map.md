# Runtime UI Memory Map

The 4.21 client exposes enough stable process state to build a useful read-only UI inspector. Static module globals lead to the Screen composition tree, the Event receiver tree, and several important persistent panes. Heap pane objects share a common base layout, and their first pointer identifies the concrete implementation through a module-relative virtual-table address.

This page is a runtime lookup reference. It describes addresses and fields confirmed by code paths that construct, read, update, or destroy them. The broader static global catalog is in [Data Map](data-map.md), packed programmer-facing declarations are in [C Structure Layouts](c-structure-layouts.md), and the complete pane-class discriminator list is in [Pane Virtual Table Inventory](ui-pane-vtables.md). [Event Proxy IPC, Rules, and State](../event-proxy/ipc-rules-and-state.md#runtime-state-and-memory-access) applies these paths to late-attach snapshots and guarded peek or poke requests.

## High-level operation

### Address model

`Darkages.exe` is a 32-bit image with preferred image base `0x00400000`. Every pointer and `uintptr_t` value discussed here is four bytes. The documented virtual addresses identify the analyzed image, but an external reader should resolve the loaded module base and add an RVA:

```c
uintptr_t runtime_address = loaded_module_base + documented_rva;
```

Do not assume that `Darkages.exe` loaded at its preferred base. A virtual-table address is also relocated with the module. For example, a live GameButtons pane is identified by:

```c
int is_game_buttons_pane =
    pane_vtable == loaded_module_base + 0x00110300;
```

### Static runtime roots

Each address in the table is storage in the image. Read one 32-bit pointer from `loaded_module_base + RVA` to obtain the heap object.

| Static VA | RVA | Current IDA name | Heap object reached | Useful next step |
|---:|---:|---|---|---|
| `Darkages.exe:0x004E2D70` | `0x000E2D70` | `ui_bulletin_session` | Active BulletinSession | Read active child at session `+0x120`. |
| `Darkages.exe:0x004E2E40` | `0x000E2E40` | `ui_lag_indicator_pane` | Persistent lag indicator pane | Read its smoothed movement round-trip value at `+0xF4`. |
| `Darkages.exe:0x004E32A4` | `0x000E32A4` | `ui_equip_pane` | Persistent User Equip pane | Read visibility, equipment slots, self-look strings, and legend child pointers. |
| `Darkages.exe:0x004E3564` | `0x000E3564` | `ui_users_dialog_pane` | Users Dialog Pane | Read visibility and its nine row/list objects. |
| `Darkages.exe:0x004F5198` | `0x000F5198` | `ui_local_user` | Active local user object | Read movement timing at `+0x6F2C/+0x6F34`. |
| `Darkages.exe:0x004F51AC` | `0x000F51AC` | `ui_main_menu_pane` | MainMenuPane | Distinguishes the main-menu phase. |
| `Darkages.exe:0x004F51B0` | `0x000F51B0` | `ui_map_pane` | MapPane | Distinguishes the in-game phase and leads to map-owned state. |
| `Darkages.exe:0x004F51B4` | `0x000F51B4` | `ui_background_pane` | BackgroundPane | Persistent parent of the in-game UI. |
| `Darkages.exe:0x004F51C8` | `0x000F51C8` | `ui_screen_pane` | Root ScreenPane | Persistent after successful initialization. |
| `Darkages.exe:0x004F51CC` | `0x000F51CC` | `ui_screen_registry` | Screen hierarchy list | Read root node pointer at list `+0x1C`. |
| `Darkages.exe:0x004F51D0` | `0x000F51D0` | `event_dispatcher` | Event dispatcher | Read Event pane hierarchy at dispatcher `+0x68`. |
| `Darkages.exe:0x004FD640` | `0x000FD640` | `ui_terminal_pane` | TerminalPane | Distinguishes terminal/bootstrap phase. |

These roots are nullable according to lifecycle. A null pointer is ordinarily a phase or ownership result, not an error in the reader.

### What the two pane trees reveal

The Screen tree is the primary inventory of composed UI. It contains persistent panes, hidden content panes that remain attached, dialogs, controls, and temporary slot panes. Its parent-child links also describe UI ownership and placement.

The Event tree is an independent inventory of panes registered for mouse, keyboard, socket-packet, or timer delivery. It is useful as a cross-check and can contain an object whose Screen membership is changing. Neither tree is an ownership-safe API for an external process. Both can change while they are read.

A practical read-only inspector can:

1. Snapshot the Screen tree.
2. Validate each pane pointer and vtable.
3. Read the common visibility byte and bounds.
4. Label known classes by vtable RVA.
5. Follow class-specific pointers only after confirming the expected vtable.
6. Optionally snapshot the Event tree and mark which visible or hidden panes currently receive Events.

### Useful observation recipes

#### List current panes

Read `ui_screen_registry`, then its root at `registry + 0x1C`. Observe the root pane at `root + 0x08`, and recursively snapshot the child list at `root + 0x04`. For every node, read the pane pointer at node `+0x08`. Read the live pane's vtable, local bound rectangle at `+0x38`, relative top/left origin at `+0xA8`, and visible byte at `+0xB0`.

The full vtable inventory contains construction and control variants as well as visible screens. A vtable match identifies the object's current implementation, but does not imply that the pane is visible or top-level.

#### Find the selected game content

There is no separate static root for GameButtonsPane. Find the pane whose vtable is `loaded_module_base + 0x00110300`. Its field at `+0x124` points to the currently selected content pane. Compare that pointer with fields `+0x130` through `+0x140` to label the selection as chat, status, equipment, skills, or spells.

This pointer records selection, while the selected pane's `+0xB0` byte records current visibility. Equipment can be the selected object while hidden because selecting equipment twice toggles its visibility.

#### Read skills and spells

From GameButtonsPane, read the skill inventory at `+0x13C` and the spell inventory at `+0x140`. Each parent owns a 36-entry pointer array beginning at `+0xF4`. Array entry zero represents protocol slot 1:

```c
uint32_t slot_pane =
    read_process_u32(inventory + 0x00f4 + 4 * (slot - 1));
```

A null entry is an empty slot. A non-null skill entry should use vtable RVA `0x00114820`; a spell entry should use `0x00114880`. The child objects retain the server-provided display ID and strings, their 1-based slot, pointer interaction state, and cooldown-active state.

#### Read equipment and self-look state

Read `ui_equip_pane` directly. The normal equipment range is 1 through 13. For a 1-based slot:

```c
uintptr_t item_id_address = equip_pane + 0x0b22 + 2 * slot;
uintptr_t item_name_address =
    equip_pane + 0x0b3e + 0x80 * (slot - 1);
```

The identifier is a native little-endian `uint16_t` in memory, even though the server packet supplied it in big-endian order. The name is a NUL-terminated byte string in a `0x80`-byte record. The pane also retains the most recent `SSelfLook` appearance bytes, strings, a 32-bit value, and two child-pane pointers. Several field meanings remain unknown, so the code-level table uses `unknown_XX` names.

#### Read bulletin, mail, and users state

`ui_bulletin_session` points to a specialized Pane with vtable RVA `0x00101E60`. Its current child pointer at `+0x120` leads to the active bulletin or mail dialog. Session `+0xF5` records its current child-history position and the one-byte field at `+0x124` records request-wait state.

`ui_users_dialog_pane` is a Pane with a direct static root. It owns nine heap row/list objects at `+0x550` through `+0x570`. The dialog remains allocated and is reused by later `SShowUsers` replies, so `+0xB0` is the useful shown/hidden test.

#### Read movement timing and the lag indicator

Read `ui_local_user` while the game UI is active. Its latest queued `CMove` timestamp is a 32-bit `timeGetTime` value at `+0x6F2C`, and the latest `SMove` response sample is at `+0x6F34`. The persistent lag pane has its own direct root, `ui_lag_indicator_pane`, and stores the displayed smoothed value at pane `+0xF4`.

The raw sample and displayed sample are intentionally different. On every accepted `SMove`, the pane replaces its value with `(old_value + latest_sample) / 2`. A reader that wants to reproduce the visible band should use the pane field. A reader that wants the latest unsmoothed movement response should use the local-user field.

### Peek versus poke

Read-only observation is practical with the layouts on this page. Raw writes are not equivalent to client UI operations.

Changing `Pane + 0xB0` does not perform invalidation, capture release, focus changes, or parent redraw. Replacing a child pointer does not update either packed hierarchy and can create a use-after-free during Event traversal. Editing a vtable or packed node can redirect a virtual call. Editing a fixed string without preserving its limit and NUL termination can overwrite adjacent state.

If controlled UI mutation is investigated later, the safer design is to invoke the established pane method on the client UI path and let it update both registries and rendering state. This page intentionally documents a read-only snapshot model.

## Code-level memory map

### Packed hierarchy list

Both Screen and Event registries use the same packed hierarchy list. Multi-byte fields in a node may be unaligned, so an external reader should copy bytes or use an unaligned-safe read helper instead of casting a local aligned structure over remote memory.

| List offset | Width | Meaning |
|---:|---:|---|
| `+0x0C` | 4 | Node stride. |
| `+0x14` | 4 | Node count. |
| `+0x18` | 4 | Pointer to contiguous packed node bytes. |
| `+0x1C` | 4 | Root or parent node pointer. Screen stores its separately allocated root here. |

| Common node offset | Width | Meaning |
|---:|---:|---|
| `+0x00` | 4 | Parent node pointer. |
| `+0x04` | 4 | Child hierarchy-list pointer, or null. |
| `+0x08` | 4 | Pane pointer, the first field of both known payloads. |

Screen requests a `0x34`-byte payload, so its node stride is `0x3F`. Event requests a four-byte payload, so its stride is `0x0F`.

| Screen node offset | Width | Meaning |
|---:|---:|---|
| `+0x08` | 4 | Pane pointer. |
| `+0x0C` | 16 | Screen-owned rectangle or region state, initialized to zero on insertion. This is not the Pane bounds field. |
| `+0x1C` | 1 | Visible state copied from `ui_pane_is_visible` during insertion and maintained by Screen operations. |
| `+0x1D` | 1 | Pane `+0xB3` flag copied during insertion. |
| `+0x1E` | 1 | Initialized to zero during insertion. |
| `+0x1F` | remaining payload | Additional Screen-owned state whose fields are not yet published. |

The runtime entry chains are:

```c
uintptr_t screen_list =
    read_process_u32(module_base + 0x000f51cc);
uintptr_t screen_root = read_process_u32(screen_list + 0x1c);
uintptr_t screen_children = read_process_u32(screen_root + 0x04);

uintptr_t dispatcher =
    read_process_u32(module_base + 0x000f51d0);
uintptr_t event_list = read_process_u32(dispatcher + 0x68);
```

### Common Pane fields

| Pane offset | Width | Meaning | Reader use |
|---:|---:|---|---|
| `+0x0000` | 4 | Current vtable pointer. | Convert to an RVA relative to the loaded module and match the inventory. |
| `+0x0008` | 4 | Last propagated Screen or dispatcher error. | Diagnostic state, not a class identifier. |
| `+0x0038` | 16 | Local graphic bound rectangle. | Four signed 32-bit values in client order: top, left, bottom, right. |
| `+0x00A8` | 8 | Relative origin point. | Signed 32-bit top at `+0xA8` and left at `+0xAC`. |
| `+0x00B0` | 1 | Visible or active byte. | Nonzero means shown according to `ui_pane_is_visible`. |
| `+0x00B1` | 1 | `unknown_B1` constructor flag. | Established location, semantics not established. |
| `+0x00B2` | 1 | `unknown_B2` constructor flag. | Established location, semantics not established. |
| `+0x00B3` | 1 | Screen payload flag. | Copied into Screen node `+0x1D`. |
| `+0x00F0` | 1 | Capture-related state. | Checked before Screen removal. |

`ui_pane_set_bound_rect` retains the input rectangle's top and left at `+0xA8/+0xAC`, translates the rectangle to a local origin, and applies it to the graphic bound rectangle at `+0x38`. `ui_pane_get_bound_rect` reverses that translation. The in-parent rectangle can therefore be reconstructed without a client call:

```c
int32_t relative_top = pane_bound_top + pane_origin_top;
int32_t relative_left = pane_bound_left + pane_origin_left;
int32_t relative_bottom = pane_bound_bottom + pane_origin_top;
int32_t relative_right = pane_bound_right + pane_origin_left;
```

These client rectangles use top, left, bottom, right field order, not the Win32 `RECT` memory order. `ui_screen_get_pane_origin` then finds the Screen node and accumulates parent placement. An external tree walker can preserve the parent chain and perform the same accumulation when absolute screen coordinates are needed.

### Runtime class discriminators

The following vtables are especially useful for state walking. The address to compare at runtime is `loaded_module_base + RVA`.

| Class or role | Static VA | RVA | Current IDA name |
|---|---:|---:|---|
| BulletinSession | `Darkages.exe:0x00501E60` | `0x00101E60` | `ui_bulletin_session_vtable` |
| Lag indicator pane | `Darkages.exe:0x00509C20` | `0x00109C20` | `ui_lag_indicator_pane_vtable` |
| GameButtonsPane | `Darkages.exe:0x00510300` | `0x00110300` | `ui_game_buttons_pane_vtable` |
| User Equip pane | `Darkages.exe:0x0050C6C0` | `0x0010C6C0` | `ui_equip_pane_vtable` |
| Users Dialog Pane | `Darkages.exe:0x005106E0` | `0x001106E0` | `ui_users_dialog_pane_vtable` |
| Skill inventory parent | `Darkages.exe:0x00510420` | `0x00110420` | `ui_skill_inventory_pane_vtable` |
| Spell inventory parent | `Darkages.exe:0x00510480` | `0x00110480` | `ui_spell_inventory_pane_vtable` |
| Skill slot child | `Darkages.exe:0x00514820` | `0x00114820` | `ui_skill_slot_pane_vtable` |
| Spell slot child | `Darkages.exe:0x00514880` | `0x00114880` | `ui_spell_slot_pane_vtable` |
| `NewUser` local-user pane | `Darkages.exe:0x0051E980` | `0x0011E980` | `ui_local_user_vtable` |

### Lag indicator and movement timing

The lag indicator is a `0xF8`-byte Pane created with the rest of the in-game pane graph. Its constructor publishes `ui_lag_indicator_pane`, initializes `+0xF4` to 300 milliseconds, sets a 32 by 32 local bound, and adds and registers the pane below `BackgroundPane`. The attach method passes `left = 537`, `top = 314`, `right = 549`, and `bottom = 328` to `ui_rect_init`. The helper stores the client-order words `(top = 314, left = 537, bottom = 328, right = 549)`, placing the visible indicator at `x = 537..549`, `y = 314..328`.

The value is not based on socket throughput, receive queue depth, or a periodic ping packet. `net_c_send_move` builds `CMove` action `0x06`, increments the one-byte movement sequence at local user `+0x6F28`, and stores `timeGetTime` at `+0x6F2C`. When the local user receives `SMove` action `0x0B`, `ui_local_user_handle_smove` stores the unsigned difference between the new clock sample and `+0x6F2C` at `+0x6F34`. It then calls `ui_lag_indicator_update_from_local_user`, which averages the new sample with pane `+0xF4` and invalidates the pane rectangle.

```c
uint32_t latest_sample = current_tick - last_move_send_tick;
uint32_t displayed_sample =
    (previous_displayed_sample + latest_sample) / 2;
```

The sender keeps only one timestamp. If another `CMove` is queued before an earlier `SMove` arrives, the newer send overwrites `+0x6F2C`. The packet byte read at decoded-packet offset `+0x0A` is also discarded rather than compared against the client movement sequence. The indicator is therefore a smoothed response time relative to the most recent move timestamp, not a rigorous per-sequence network RTT measurement.

`ui_lag_indicator_pane_draw` selects `statcon.epf` frames with unsigned comparisons:

| Smoothed value at pane `+0xF4` | Frame | Visual color in the checked version-family asset | Notes |
|---:|---:|---|---|
| `0` | 2 | Green | Zero bypasses the 1 through 249 fast band. Normal construction starts at 300, not zero. |
| `1` through `249` | 3 | Blue | Fastest band. |
| `250` through `349` | 2 | Green | Constructor default 300 starts here. |
| `350` through `449` | 1 | Orange | Intermediate warning band. |
| `450` or more | 0 | Red | Slowest band. |

The frame numbers and thresholds come directly from the 4.21 draw code. The color labels were checked by decoding the same-named `statcon.epf` against `legend.pal` from an available 4.51 data set: frame 3 is blue, frame 2 green, frame 1 orange, and frame 0 red. Exact 4.21 asset identity was not available in this workspace, so tools that require byte-identical 4.21 colors should verify those four asset frames while retaining the code-established frame mapping.

| Object offset | Width | Meaning |
|---:|---:|---|
| lag pane `+0x00F4` | 4 | Unsigned smoothed movement response value used for frame selection. |
| local user `+0x6F28` | 1 | Incrementing `CMove` sequence byte. |
| local user `+0x6F29` | 1 | Current movement sequence copied when an `SMove` is handled. It is not populated from the packet byte read at `+0x0A`. |
| local user `+0x6F2C` | 4 | `timeGetTime` value from the most recently queued `CMove`. |
| local user `+0x6F34` | 4 | Latest unsmoothed `SMove` minus latest-`CMove` clock difference. |

See [Pane Virtual Table Inventory](ui-pane-vtables.md) for dialogs, bulletin and mail panes, controls, MapPane, options, books, effects, and the remaining compatible tables.

### GameButtonsPane fields

GameButtonsPane is constructed by `ui_game_buttons_pane_ctor` and found dynamically by its vtable. Its direct links make it the most useful bridge from generic pane enumeration to persistent in-game UI state.

| Offset | Width | Meaning |
|---:|---:|---|
| `+0x00F4` | 4 | MapPane pointer passed to the constructor. It normally matches `ui_map_pane`. |
| `+0x0100` | 1 | Equipment button selected state. |
| `+0x0108` | 1 | Skill button selected state. |
| `+0x0110` | 1 | Spell button selected state. |
| `+0x0118` | 1 | Chat button selected state. |
| `+0x0120` | 1 | Status button selected state. |
| `+0x0124` | 4 | Current content-pane pointer. |
| `+0x0130` | 4 | Chat pane pointer. |
| `+0x0134` | 4 | Status pane pointer. |
| `+0x0138` | 4 | Equipment pane pointer. It normally matches `ui_equip_pane`. |
| `+0x013C` | 4 | Skill inventory parent pointer. |
| `+0x0140` | 4 | Spell inventory parent pointer. |
| `+0x0144` | 4 | System-message pointer passed to the constructor. |

The five selection methods hide the previous `+0x124` pane, show the chosen field, and replace `+0x124`. The button-state bytes are also updated so exactly one ordinary content button is selected.

### Skill inventory and slot panes

The skill inventory parent has vtable `ui_skill_inventory_pane_vtable`. It owns exactly 36 heap pointers.

| Parent offset | Width | Meaning |
|---:|---:|---|
| `+0x00F4` | `36 * 4` | Slot pane pointers for protocol slots 1 through 36. |

`SAddSkill` replaces the selected entry with a `0x294`-byte pane. `SRemoveSkill` unregisters, removes, deletes, and clears the entry. `SCooldown` marks the selected child and later clears it through a parent timer.

| Skill slot offset | Width | Meaning |
|---:|---:|---|
| `+0x0000` | 4 | `ui_skill_slot_pane_vtable`. |
| `+0x00F4` | 2 | Server-provided big-endian display ID, stored in native little-endian form. |
| `+0x00F6` | variable, maximum observed name length 127 | NUL-terminated skill name. |
| `+0x0276` | 1 | 1-based skill slot. This byte is sent by `CUseSkill`. |
| `+0x0277` | 1 | Pointer-capture or pressed state used by the mouse handler. |
| `+0x0278` | 1 | Cooldown-active flag. `ui_skill_slot_start_cooldown` sets it. |
| `+0x027C` | 8 | Stored pointer-down point, two signed 32-bit coordinates. |
| `+0x0284` | 16 | Pointer-tracking client rectangle. |

### Spell inventory and slot panes

The spell inventory parent mirrors the skill parent and also owns 36 heap pointers at `+0xF4`.

| Spell slot offset | Width | Meaning |
|---:|---:|---|
| `+0x0000` | 4 | `ui_spell_slot_pane_vtable`. |
| `+0x00F4` | 1 | 1-based spell slot. This byte is sent by the immediate `CUseSpell` path. |
| `+0x00F6` | 2 | Server-provided big-endian display ID, stored in native little-endian form. |
| `+0x00F8` | 1 | Signed spell type, established range 1 through 8. It selects the client activation path. |
| `+0x00F9` | `0x80` | NUL-terminated spell name buffer. |
| `+0x0179` | `0x80` | NUL-terminated spell comment buffer. |
| `+0x01F9` | 1 | `unknown_1F9`, copied from the trailing `SAddSpell` byte and passed to SpellBookDialog. |
| `+0x01FA` | 1 | Pointer-capture or pressed state used by the mouse handler. |
| `+0x01FB` | 1 | Cooldown-active flag. `ui_spell_slot_start_cooldown` sets it. |
| `+0x01FC` | 8 | Stored pointer-down point, two signed 32-bit coordinates. |
| `+0x0204` | 16 | Pointer-tracking client rectangle. |

### Equipment and self-look fields

The User Equip pane uses vtable `ui_equip_pane_vtable` and has a direct static root. The item fields use a 1-based protocol slot directly in the identifier expression, while names use a zero-based record index.

| Offset or expression | Width | Meaning |
|---:|---:|---|
| `+0x0550` through `+0x0553` | 4 | Four appearance bytes copied from the packet. |
| `+0x0554` | `0x100` | NUL-terminated `unknown_554` self-look string buffer. |
| `+0x0654` | `0x80` | NUL-terminated `unknown_654` self-look string buffer. |
| `+0x06D4` | `0x80` | NUL-terminated `unknown_6D4` self-look string buffer. |
| `+0x0754` | `0x80` | NUL-terminated `unknown_754` self-look string buffer. |
| `+0x07D4` | at least `0x100` | NUL-terminated `unknown_7D4` self-look string buffer. |
| `+0x0B22 + 2 * slot` | 2 | Item identifier for 1-based equipment slot 1 through 13. |
| `+0x0B3E + 0x80 * (slot - 1)` | `0x80` | NUL-terminated item name record for slot 1 through 13. |
| `+0x11C0` | 4 | `unknown_11C0` big-endian packet value stored in host order. |
| `+0x11C4` | 1 | `unknown_11C4` packet byte. |
| `+0x11C8` | 4 | Owned helper-pane pointer, allocated by the equipment constructor. |
| `+0x11CC` | 4 | Owned `LegendDialogPane` pointer. |
| `+0x11D0` | 1 | `unknown_11D0` state flag initialized to zero. |

The five strings are updated as one `SSelfLook` transaction. A snapshot reader should copy each bounded buffer and accept it only if the pane pointer and vtable remain unchanged after the copy.

### BulletinSession fields

| Session offset | Width | Meaning |
|---:|---:|---|
| `+0x0000` | 4 | `ui_bulletin_session_vtable`. |
| `+0x00F5` | 1 | Current child-history index. |
| `+0x00F8` | `10 * 4` | Ten child-dialog history pointers. |
| `+0x0120` | 4 | Active bulletin or mail child pane. |
| `+0x0124` | 1 | Request-wait state used to suppress overlapping operations. |

The session is a Pane and can be validated against vtable RVA `0x00101E60`. Its active child at `+0x120` is another Pane with its own bulletin or mail class vtable.

### Users Dialog Pane fields

| Pane offset | Width | Meaning |
|---:|---:|---|
| `+0x0000` | 4 | `ui_users_dialog_pane_vtable`. |
| `+0x0550` | `9 * 4` | Nine owned heap row/list object pointers. |
| `+0x0574` | 4 | Additional owned list-storage object. |
| `+0x0578` | 1 | `unknown_578` state flag. |
| `+0x0579` | 1 | `unknown_579` state flag. |
| `+0x057C` | 4 | Owned list or scrolling control pointer. |
| `+0x0581` | 1 | `unknown_581` state flag. |

Row record internals are not yet mapped to the documentation threshold. The direct pane root, visibility, common bounds, and nine owned objects are established.

### Stable external snapshot procedure

The client does not expose a lock for external readers. Screen insertion and removal can relocate a list's packed node array, and deferred deletion can free a pane after it disappears from a registry. Use retryable snapshots:

1. Read list stride, count, array pointer, and root pointer.
2. Reject an unexpected stride, an implausible count, integer overflow in `count * stride`, a null array with nonzero count, or an unreadable byte range.
3. Copy `count * stride` bytes with one `ReadProcessMemory` operation.
4. Re-read the four metadata fields and discard the copy if any changed.
5. Parse nodes from the local byte copy with unaligned-safe reads.
6. Snapshot each non-null child list recursively.
7. Before reading a pane, confirm that its first four bytes are readable and that its vtable is a known module address.
8. After copying class-specific state, re-read the pane's vtable and the owner pointer that led to it. Discard the object snapshot if either changed.

The core packed-node calculation is:

```c
static uintptr_t packed_node_address(
    uintptr_t node_array,
    uint32_t node_stride,
    uint32_t node_index)
{
    return node_array + (uintptr_t)node_stride * node_index;
}
```

Do not cast the remote `0x3F`- or `0x0F`-byte nodes to an ordinary local C structure. Their second and later records are not four-byte aligned.

## Function table

| Address | Current IDA name | Prototype | Purpose | Call relationships and notes |
|---:|---|---|---|---|
| `Darkages.exe:0x0040E660` | `ui_bulletin_session_deleting_dtor` | established `__thiscall` deleting destructor | Destroy the active BulletinSession and its child history. | Clears `ui_bulletin_session` and unregisters and removes the Pane. |
| `Darkages.exe:0x0040EB10` | `ui_bulletin_session_ctor` | `void *__thiscall(void *, int, const uint8_t *)` | Construct and publish a `0x12C`-byte BulletinSession Pane. | Initializes the ten-entry history and one-byte request-wait field. |
| `Darkages.exe:0x00423984` | `ui_lag_indicator_pane_dtor` | `void __thiscall(struct ui_lag_indicator_pane *)` | Destroy the lag indicator Pane. | Clears `ui_lag_indicator_pane` before base Pane cleanup. |
| `Darkages.exe:0x004239C0` | `ui_get_lag_indicator_pane` | `struct ui_lag_indicator_pane *__cdecl(void)` | Return the persistent lag indicator pointer. | Used by the SMove timing path and game-pane teardown. |
| `Darkages.exe:0x004239D0` | `ui_lag_indicator_pane_ctor` | `struct ui_lag_indicator_pane *__thiscall(struct ui_lag_indicator_pane *, void *)` | Construct and publish the `0xF8`-byte lag indicator Pane. | Initializes the displayed value to 300 and calls the attach method. |
| `Darkages.exe:0x00423A44` | `ui_lag_indicator_pane_attach` | `void __thiscall(struct ui_lag_indicator_pane *, void *)` | Add and register the lag indicator below BackgroundPane. | Uses left/top/right/bottom arguments `(537, 314, 549, 328)`. |
| `Darkages.exe:0x00423AA0` | `ui_lag_indicator_pane_draw` | `void __thiscall(struct ui_lag_indicator_pane *)` | Select and draw a `statcon.epf` frame. | Vtable slot `+0x48`; applies the five value bands documented above. |
| `Darkages.exe:0x00423B60` | `ui_lag_indicator_update_from_local_user` | `void __thiscall(struct ui_lag_indicator_pane *)` | Average the latest local-user timing sample into the displayed value. | Reads local user `+0x6F34`, writes pane `+0xF4`, and invalidates the Pane. |
| `Darkages.exe:0x00431150` | `event_dispatcher_register_pane` | `void __thiscall(void *, void *, int, int)` | Insert a pane into the Event hierarchy. | Event list is reached at dispatcher `+0x68`. |
| `Darkages.exe:0x0043A5F0` | `ui_game_buttons_pane_ctor` | `void *__thiscall(void *, void *, void *)` | Construct GameButtonsPane and its persistent content graph. | Installs `ui_game_buttons_pane_vtable`. |
| `Darkages.exe:0x0043CF20` | `ui_game_buttons_handle_key_event` | `int __thiscall(void *, void *)` | Select equipment, skills, spells, chat, or status. | Writes button state and calls `ui_game_buttons_select_content`. |
| `Darkages.exe:0x00442A34` | `ui_skill_inventory_apply_add_packet` | `int __thiscall(void *, const uint8_t *)` | Parse `SAddSkill` and replace a slot child. | Calls remove and create methods. |
| `Darkages.exe:0x00442C04` | `ui_skill_inventory_remove_slot_pane` | `void __thiscall(void *, uint8_t)` | Remove and delete one skill slot pane. | Clears one pointer at parent `+0xF4`. |
| `Darkages.exe:0x00442D14` | `ui_skill_inventory_create_slot_pane` | `void __thiscall(void *, uint8_t, uint16_t, const char *)` | Allocate and register one skill slot pane. | Object size `0x294`. |
| `Darkages.exe:0x00443BB4` | `ui_spell_inventory_apply_add_packet` | `int __thiscall(void *, const uint8_t *)` | Parse `SAddSpell` and replace a slot child. | Calls remove and create methods. |
| `Darkages.exe:0x00443E14` | `ui_spell_inventory_remove_slot_pane` | `void __thiscall(void *, uint8_t)` | Remove and delete one spell slot pane. | Clears one pointer at parent `+0xF4`. |
| `Darkages.exe:0x00443F24` | `ui_spell_inventory_create_slot_pane` | `void __thiscall(void *, uint8_t, uint16_t, int8_t, const char *, const char *, uint8_t)` | Allocate and register one spell slot pane. | Object size `0x214`. |
| `Darkages.exe:0x00446090` | `ui_point_init` | `void __cdecl(void *, int, int)` | Initialize an eight-byte client point. | Used for Pane relative-origin fields. |
| `Darkages.exe:0x004460B0` | `ui_rect_init` | `void __cdecl(void *, int, int, int, int)` | Initialize a client rectangle from left, top, right, and bottom. | Stores fields in top, left, bottom, right memory order. |
| `Darkages.exe:0x0044E4D0` | `ui_hierarchy_list_ctor` | `void *__thiscall(void *, int)` | Construct a packed hierarchy list. | Node stride is payload size plus `0x0B`. |
| `Darkages.exe:0x00454CA0` | `ui_skill_slot_finish_cooldown` | established `__thiscall` method | Clear a skill slot's cooldown-active byte. | Called by the parent inventory timer path. |
| `Darkages.exe:0x00454CC0` | `ui_skill_slot_start_cooldown` | established `__thiscall` method | Set a skill slot's cooldown-active byte. | Reached from `SCooldown` processing. |
| `Darkages.exe:0x00456220` | `ui_spell_slot_finish_cooldown` | established `__thiscall` method | Clear a spell slot's cooldown-active byte. | Called by the parent inventory timer path. |
| `Darkages.exe:0x00456240` | `ui_spell_slot_start_cooldown` | established `__thiscall` method | Set a spell slot's cooldown-active byte. | Reached from `SCooldown` processing. |
| `Darkages.exe:0x00456B90` | `ui_skill_slot_pane_ctor` | `void *__thiscall(void *, uint8_t, uint16_t, const char *)` | Initialize a skill slot's persistent fields. | Installs `ui_skill_slot_pane_vtable`. |
| `Darkages.exe:0x00456C70` | `ui_spell_slot_pane_ctor` | `void *__thiscall(void *, uint8_t, uint16_t, int8_t, const char *, const char *, uint8_t)` | Initialize a spell slot's persistent fields. | Installs `ui_spell_slot_pane_vtable`. |
| `Darkages.exe:0x004594A0` | `ui_hierarchy_get_node` | `void *__thiscall(void *, int)` | Return `array + index * stride` after bounds checks. | Confirms unaligned packed-node addressing. |
| `Darkages.exe:0x004876C4` | `ui_local_user_dtor` | `void __thiscall(struct ui_local_user *)` | Destroy the local-user Pane and owned state. | Clears `ui_local_user_singleton`; the game-pane teardown path clears the owning `ui_local_user` root. |
| `Darkages.exe:0x00487700` | `ui_local_user_ctor` | `struct ui_local_user *__thiscall(struct ui_local_user *, void *)` | Construct and publish the `0x776C`-byte local-user Pane. | Initializes the movement sequence and timing fields. |
| `Darkages.exe:0x00487860` | `ui_get_local_user` | `struct ui_local_user *__cdecl(void)` | Return the active local-user pointer. | Supplies the raw timing sample to the lag indicator update. |
| `Darkages.exe:0x004884E4` | `net_c_send_move` | `void __thiscall(struct ui_local_user *, uint8_t)` | Build and queue CMove while starting its timing sample. | Increments `+0x6F28` and overwrites `+0x6F2C`. |
| `Darkages.exe:0x004889F0` | `ui_local_user_handle_server_packet` | `int __thiscall(struct ui_local_user *, void *)` | Dispatch local-user server actions. | Routes action `0x0B` to `ui_local_user_handle_smove`. |
| `Darkages.exe:0x004897C4` | `ui_local_user_handle_smove` | `int __thiscall(struct ui_local_user *, const uint8_t *)` | Parse SMove, record a raw timing sample, and process movement acknowledgment or correction. | Calls the lag indicator update after `timeGetTime - last_move_send_tick`. |
| `Darkages.exe:0x00493240` | `ui_pane_get_bound_rect` | `void __thiscall(void *, int32_t *)` | Return Pane bounds translated by its relative origin. | Reads local rectangle `+0x38` and point `+0xA8`. |
| `Darkages.exe:0x004932B0` | `ui_pane_set_bound_rect` | `void __thiscall(void *, const int32_t *)` | Split input bounds into relative origin and local rectangle. | Vtable slot `+0x24`; called during Screen insertion. |
| `Darkages.exe:0x00498F40` | `ui_screen_add_pane` | established `__thiscall` add method | Insert a pane and `0x34`-byte payload into Screen. | Copies visibility and pane flags into the node. |
| `Darkages.exe:0x004993C4` | `ui_screen_find_pane_node` | established recursive `__thiscall` search | Find a pane in Screen. | Traverses nested packed child lists. |
| `Darkages.exe:0x00499ED0` | `ui_screen_get_pane_origin` | established `__thiscall` query | Accumulate a pane's screen origin. | Uses Screen parent relationships. |
