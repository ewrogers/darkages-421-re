# C Structure Layouts

This appendix presents the established 4.21 runtime layouts as C99-compatible structures. It is a programmer-oriented companion to the exact offset tables in [Data Map](data-map.md) and [Runtime UI Memory Map](runtime-ui-memory-map.md). Those tables remain authoritative for field evidence, ownership, lifetime, and safe-reading guidance.

These declarations describe bytes in a 32-bit remote process. `da_ptr32` is an address-sized value in the client, not a native pointer in a 64-bit inspector. Read a remote object into a byte buffer, validate its source pointer and vtable, and copy unaligned scalar fields with `memcpy`. Do not cast an arbitrary `ReadProcessMemory` destination to one of these packed types and then assume that every host permits unaligned access.

Unknown ranges are explicit so that each published field lands at its documented offset. A layout ending at the last known field is a prefix or view, not proof that the original C++ class ended there.

```c
#include <stdint.h>

typedef uint32_t da_ptr32;

typedef struct da_client_point {
    int32_t top;
    int32_t left;
} da_client_point;                      /* 0x0008 bytes */

typedef struct da_client_rect {
    int32_t top;
    int32_t left;
    int32_t bottom;
    int32_t right;
} da_client_rect;                       /* 0x0010 bytes */

#pragma pack(push, 1)
```

## Static root and vtable RVAs

These constants are RVAs, not absolute runtime addresses. Add each value to the loaded `Darkages.exe` module base. A root constant identifies four bytes of static pointer storage. A vtable constant identifies the relocated value expected in the first dword of a compatible live object.

```c
#define DA_RVA_UI_BULLETIN_SESSION       0x000e2d70u
#define DA_RVA_UI_LAG_INDICATOR_PANE     0x000e2e40u
#define DA_RVA_UI_EQUIP_PANE             0x000e32a4u
#define DA_RVA_EVENT_MANAGER             0x000e32c0u
#define DA_RVA_UI_USERS_DIALOG_PANE      0x000e3564u
#define DA_RVA_UI_LOCAL_USER             0x000f5198u
#define DA_RVA_UI_MAIN_MENU_PANE         0x000f51acu
#define DA_RVA_UI_MAP_PANE               0x000f51b0u
#define DA_RVA_UI_BACKGROUND_PANE        0x000f51b4u
#define DA_RVA_NET_SOCKET                0x000f51bcu
#define DA_RVA_UI_SCREEN_PANE            0x000f51c8u
#define DA_RVA_UI_SCREEN_REGISTRY        0x000f51ccu
#define DA_RVA_EVENT_DISPATCHER          0x000f51d0u
#define DA_RVA_MEMORY_MANAGER            0x000fa160u
#define DA_RVA_UI_LOCAL_USER_ALIAS       0x000fc244u
#define DA_RVA_UI_TERMINAL_PANE          0x000fd640u

#define DA_RVA_VTABLE_BULLETIN_SESSION   0x00101e60u
#define DA_RVA_VTABLE_LAG_INDICATOR      0x00109c20u
#define DA_RVA_VTABLE_EQUIP_PANE         0x0010c6c0u
#define DA_RVA_VTABLE_EVENT_MANAGER      0x0010d720u
#define DA_RVA_VTABLE_GAME_BUTTONS       0x00110300u
#define DA_RVA_VTABLE_SKILL_INVENTORY    0x00110420u
#define DA_RVA_VTABLE_SPELL_INVENTORY    0x00110480u
#define DA_RVA_VTABLE_USERS_DIALOG       0x001106e0u
#define DA_RVA_VTABLE_SKILL_SLOT         0x00114820u
#define DA_RVA_VTABLE_SPELL_SLOT         0x00114880u
#define DA_RVA_VTABLE_LOCAL_USER         0x0011e980u
#define DA_RVA_VTABLE_NET_SOCKET         0x00126940u
```

For example, reading the lag value takes two remote reads. First read a `da_ptr32` from `module_base + DA_RVA_UI_LAG_INDICATOR_PANE`, then read a `da_lag_indicator_pane` from the resulting heap address. Validate its first dword against `module_base + DA_RVA_VTABLE_LAG_INDICATOR` before using `smoothed_move_rtt_ms`.

## Pane and hierarchy layouts

The common Pane prefix is `0xF4` bytes. The derived layouts below embed it so that a programmer can follow a Screen node to a pane, identify the vtable, and then select the matching derived view.

```c
typedef struct da_pane_prefix {
    da_ptr32 vtable;                    /* +0x0000 */
    uint8_t unknown_0004[0x04];         /* +0x0004 */
    int32_t error_code;                 /* +0x0008 */
    uint8_t unknown_000c[0x2c];         /* +0x000C */
    da_client_rect graphic_bounds;      /* +0x0038 */
    uint8_t unknown_0048[0x60];         /* +0x0048 */
    da_client_point relative_origin;    /* +0x00A8 */
    uint8_t visible;                    /* +0x00B0 */
    uint8_t unknown_b1;                 /* +0x00B1 */
    uint8_t unknown_b2;                 /* +0x00B2 */
    uint8_t screen_payload_flag;        /* +0x00B3 */
    uint8_t unknown_00b4[0x3c];         /* +0x00B4 */
    uint8_t capture_related_state;      /* +0x00F0 */
    uint8_t unknown_00f1[0x03];         /* +0x00F1 */
} da_pane_prefix;                       /* 0x00F4 bytes */

typedef struct da_hierarchy_list {
    uint8_t unknown_0000[0x0c];         /* +0x0000 */
    uint32_t node_stride;               /* +0x000C */
    uint8_t unknown_0010[0x04];         /* +0x0010 */
    uint32_t node_count;                /* +0x0014 */
    da_ptr32 node_bytes;                /* +0x0018 */
    da_ptr32 root_or_parent_node;       /* +0x001C */
} da_hierarchy_list;                    /* 0x0020 bytes */

typedef struct da_event_hierarchy_node {
    da_ptr32 parent_node;               /* +0x00 */
    da_ptr32 child_list;                /* +0x04 */
    da_ptr32 pane;                      /* +0x08 */
    uint8_t unknown_0c[0x03];           /* +0x0C */
} da_event_hierarchy_node;              /* 0x0F-byte packed stride */

typedef struct da_screen_hierarchy_node {
    da_ptr32 parent_node;               /* +0x00 */
    da_ptr32 child_list;                /* +0x04 */
    da_ptr32 pane;                      /* +0x08 */
    da_client_rect screen_region;       /* +0x0C */
    uint8_t visible;                    /* +0x1C */
    uint8_t pane_screen_payload_flag;   /* +0x1D */
    uint8_t insertion_state;            /* +0x1E */
    uint8_t unknown_1f[0x20];           /* +0x1F */
} da_screen_hierarchy_node;             /* 0x3F-byte packed stride */
```

The hierarchy array itself is packed. Record two begins at `array + stride`, which is unaligned for both known stride values.

## Event and socket layouts

```c
typedef struct da_worker_record {
    uint32_t code;                      /* +0x00 */
    da_ptr32 data;                      /* +0x04 */
    int32_t value;                      /* +0x08 */
    da_ptr32 completion_event;          /* +0x0C */
    da_ptr32 result_buffer;             /* +0x10 */
    int32_t result_buffer_size;         /* +0x14 */
} da_worker_record;                     /* 0x18 bytes */

typedef struct da_worker_queue_base {
    da_ptr32 vtable;                    /* +0x00 */
    uint8_t unknown_04[0x08];           /* +0x04 */
    uint32_t wait_timeout_ms;           /* +0x0C */
    uint8_t wait_handle_count;          /* +0x10 */
    uint8_t unknown_11[0x03];           /* +0x11 */
    da_ptr32 wait_handles[16];          /* +0x14, index 0 is work */
    da_ptr32 work_queue;                /* +0x54 */
    da_ptr32 completion_monitor;        /* +0x58 */
    da_ptr32 completion_waiters;        /* +0x5C */
    da_ptr32 worker_thread;             /* +0x60 */
    uint32_t worker_thread_id;          /* +0x64 */
} da_worker_queue_base;                 /* 0x68 bytes */

typedef struct da_bounded_ring {
    da_ptr32 vtable;                    /* +0x00 */
    uint8_t unknown_04[0x08];           /* +0x04 */
    da_ptr32 monitor;                   /* +0x0C */
    da_ptr32 not_full_condition;        /* +0x10 */
    da_ptr32 not_empty_condition;       /* +0x14 */
    int32_t element_size;               /* +0x18 */
    int32_t capacity;                   /* +0x1C */
    da_ptr32 buffer;                    /* +0x20 */
    int32_t element_count;              /* +0x24 */
    int32_t head_index;                 /* +0x28 */
    int32_t tail_index;                 /* +0x2C */
} da_bounded_ring;                      /* 0x30 bytes */

typedef struct da_event_dispatcher_prefix {
    da_worker_queue_base worker;        /* +0x0000 */
    da_ptr32 pane_registry;             /* +0x0068 */
    uint32_t multimedia_period_ms;      /* +0x006C */
    uint32_t current_tick;              /* +0x0070 */
    uint32_t next_deadline;             /* +0x0074 */
    da_ptr32 timer_list;                /* +0x0078 */
} da_event_dispatcher_prefix;           /* known prefix through 0x007C */

typedef struct da_event_timer_record {
    da_ptr32 receiver_pane;             /* +0x00 */
    uint32_t callback_identifier;       /* +0x04 */
    uint32_t absolute_deadline;         /* +0x08 */
    uint32_t payload_first;             /* +0x0C */
    uint32_t payload_second;            /* +0x10 */
} da_event_timer_record;                /* 0x14 bytes */

typedef struct da_event_manager_input_view {
    da_worker_queue_base worker;        /* +0x0000 */
    da_ptr32 socket_object;             /* +0x0068 */
    int32_t mouse_y;                    /* +0x006C */
    int32_t mouse_x;                    /* +0x0070 */
    uint8_t unknown_0074[0x08];         /* +0x0074 */
    uint32_t double_click_time_limit;   /* +0x007C */
    uint8_t previous_click_button;      /* +0x0080 */
    uint8_t unknown_0081[0x03];         /* +0x0081 */
    int32_t previous_click_y;           /* +0x0084 */
    int32_t previous_click_x;           /* +0x0088 */
    uint32_t previous_click_time;       /* +0x008C */
    uint8_t unknown_0090[0x300];        /* +0x0090 */
    uint8_t scan_code_pressed[256];     /* +0x0390 */
    uint8_t input_modifier_state;       /* +0x0490 */
    uint8_t unknown_0491[0x1f];         /* +0x0491 */
    uint32_t input_block_flags;         /* +0x04B0 */
    uint32_t mouse_button_state;        /* +0x04B4 */
} da_event_manager_input_view;          /* known prefix through 0x04B8 */

typedef struct da_socket_421 {
    da_worker_queue_base worker;        /* +0x00000 */
    da_ptr32 event_sink;                /* +0x00068 */
    da_ptr32 receive_cache;             /* +0x0006C */
    uint8_t unknown_00070[0x48004];     /* +0x00070 */
    uint32_t receive_cursor;            /* +0x48074 */
    uint32_t receive_remaining;         /* +0x48078 */
    uint32_t current_body_length;       /* +0x4807C */
    uint8_t unknown_48080[0x04];        /* +0x48080 */
    da_ptr32 serial_handle;             /* +0x48084 */
    uint8_t unknown_48088[0x34];        /* +0x48088 */
    uint8_t socket_connected;           /* +0x480BC */
    uint8_t unknown_480bd[0x03];        /* +0x480BD */
    uint32_t socket;                    /* +0x480C0 */
    uint8_t unknown_480c4;              /* +0x480C4 */
    uint8_t wire_body_buffer[0x10000];  /* +0x480C5 */
    uint8_t unknown_580c5[0x03];        /* +0x580C5 */
    uint32_t frame_offset;              /* +0x580C8 */
    uint8_t unknown_580cc[0x10000];     /* +0x580CC */
    uint8_t decoded_or_encoded_buffer[0x10000]; /* +0x680CC */
    uint8_t transport_mode;             /* +0x780CC */
    uint8_t frame_open;                 /* +0x780CD */
    uint8_t unknown_780ce;              /* +0x780CE */
    uint8_t legacy_send_sequence;       /* +0x780CF */
    uint8_t unknown_780d0[0x02];        /* +0x780D0 */
    uint8_t transfer_gate;              /* +0x780D2 */
    uint8_t unknown_780d3;              /* +0x780D3 */
} da_socket_421;                        /* 0x780D4 bytes */

typedef struct da_socket_event {
    da_ptr32 vtable;                    /* +0x00 */
    uint8_t unknown_04[0x08];           /* +0x04 */
    uint8_t event_type;                 /* +0x0C, value 9 */
    uint8_t unknown_0d[0x07];           /* +0x0D */
    da_ptr32 decoded_packet;            /* +0x14 */
    uint32_t decoded_packet_size;       /* +0x18 */
    uint8_t unknown_1c[0x08];           /* +0x1C */
} da_socket_event;                      /* 0x24 bytes */
```

`da_worker_queue_base.wait_handles[0]` is the native work semaphore. The worker tolerates that semaphore being signaled while `work_queue` is empty and still invokes its periodic vtable slot. This is the established wake primitive for the proposed worker-affine Event Proxy pumps. Do not modify `da_bounded_ring` indices or counts from an inspector. Socket and EventMan rings have capacity `0x80`; the dispatcher ring has capacity `0x400`.

## Persistent game UI layouts

```c
typedef struct da_lag_indicator_pane {
    da_pane_prefix pane;                /* +0x0000 */
    uint32_t smoothed_move_rtt_ms;      /* +0x00F4 */
} da_lag_indicator_pane;                /* 0x00F8 bytes */

typedef struct da_game_buttons_pane {
    da_pane_prefix pane;                /* +0x0000 */
    da_ptr32 map_pane;                  /* +0x00F4 */
    uint8_t unknown_00f8[0x08];         /* +0x00F8 */
    uint8_t equipment_selected;         /* +0x0100 */
    uint8_t unknown_0101[0x07];         /* +0x0101 */
    uint8_t skill_selected;             /* +0x0108 */
    uint8_t unknown_0109[0x07];         /* +0x0109 */
    uint8_t spell_selected;             /* +0x0110 */
    uint8_t unknown_0111[0x07];         /* +0x0111 */
    uint8_t chat_selected;              /* +0x0118 */
    uint8_t unknown_0119[0x07];         /* +0x0119 */
    uint8_t status_selected;            /* +0x0120 */
    uint8_t unknown_0121[0x03];         /* +0x0121 */
    da_ptr32 current_content_pane;      /* +0x0124 */
    uint8_t unknown_0128[0x08];         /* +0x0128 */
    da_ptr32 chat_pane;                 /* +0x0130 */
    da_ptr32 status_pane;               /* +0x0134 */
    da_ptr32 equip_pane;                /* +0x0138 */
    da_ptr32 skill_inventory_pane;      /* +0x013C */
    da_ptr32 spell_inventory_pane;      /* +0x0140 */
    da_ptr32 system_message_pane;       /* +0x0144 */
    uint8_t unknown_0148[0x10];         /* +0x0148 */
} da_game_buttons_pane;                 /* 0x0158 bytes */

typedef struct da_inventory_parent_pane {
    da_pane_prefix pane;                /* +0x0000 */
    da_ptr32 slot_panes[36];            /* +0x00F4, entry 0 is slot 1 */
    uint8_t state_184;                  /* +0x0184 */
    uint8_t unknown_0185[0x03];         /* +0x0185 */
} da_inventory_parent_pane;             /* 0x0188 bytes */

typedef struct da_skill_slot_pane {
    da_pane_prefix pane;                /* +0x0000 */
    uint16_t display_id;                /* +0x00F4 */
    char name[0x80];                    /* +0x00F6 */
    uint8_t unknown_0176[0x100];        /* +0x0176 */
    uint8_t slot;                       /* +0x0276, 1 through 36 */
    uint8_t pointer_capture_state;      /* +0x0277 */
    uint8_t cooldown_active;            /* +0x0278 */
    uint8_t unknown_0279[0x03];         /* +0x0279 */
    da_client_point pointer_down_point; /* +0x027C */
    da_client_rect tracking_rect;       /* +0x0284 */
} da_skill_slot_pane;                   /* 0x0294 bytes */

typedef struct da_spell_slot_pane {
    da_pane_prefix pane;                /* +0x0000 */
    uint8_t slot;                       /* +0x00F4, 1 through 36 */
    uint8_t unknown_00f5;               /* +0x00F5 */
    uint16_t display_id;                /* +0x00F6 */
    int8_t spell_type;                  /* +0x00F8 */
    char name[0x80];                    /* +0x00F9 */
    char comment[0x80];                 /* +0x0179 */
    uint8_t packet_flag;                /* +0x01F9 */
    uint8_t pointer_capture_state;      /* +0x01FA */
    uint8_t cooldown_active;            /* +0x01FB */
    da_client_point pointer_down_point; /* +0x01FC */
    da_client_rect tracking_rect;       /* +0x0204 */
} da_spell_slot_pane;                   /* 0x0214 bytes */
```

The skill and spell parent layouts are byte-identical in the published range. Select the child structure only after validating the parent and child vtable RVAs.

## Character, bulletin, and users layouts

```c
typedef struct da_equip_pane {
    da_pane_prefix pane;                /* +0x0000 */
    uint8_t unknown_00f4[0x45c];        /* +0x00F4 */
    uint8_t appearance[4];              /* +0x0550 */
    char self_look_554[0x100];          /* +0x0554 */
    char self_look_654[0x80];           /* +0x0654 */
    char self_look_6d4[0x80];           /* +0x06D4 */
    char self_look_754[0x80];           /* +0x0754 */
    char self_look_7d4[0x100];          /* +0x07D4, established minimum */
    uint8_t unknown_08d4[0x24e];        /* +0x08D4 */
    uint8_t unknown_0b22[0x02];         /* +0x0B22 */
    uint16_t item_id_by_slot[13];       /* +0x0B24, entry 0 is slot 1 */
    char item_name_by_slot[13][0x80];   /* +0x0B3E */
    uint8_t unknown_11be[0x02];         /* +0x11BE */
    uint32_t unknown_11c0;              /* +0x11C0 */
    uint8_t unknown_11c4;               /* +0x11C4 */
    uint8_t unknown_11c5[0x03];         /* +0x11C5 */
    da_ptr32 helper_pane;               /* +0x11C8 */
    da_ptr32 legend_dialog_pane;        /* +0x11CC */
    uint8_t unknown_11d0;               /* +0x11D0 */
    uint8_t unknown_11d1[0x03];         /* +0x11D1 */
} da_equip_pane;                        /* 0x11D4 bytes */

typedef struct da_bulletin_session {
    da_pane_prefix pane;                /* +0x0000 */
    uint8_t unknown_00f4;               /* +0x00F4 */
    uint8_t current_history_index;      /* +0x00F5 */
    uint8_t unknown_00f6[0x02];         /* +0x00F6 */
    da_ptr32 child_history[10];         /* +0x00F8 */
    da_ptr32 active_child_pane;         /* +0x0120 */
    uint8_t request_wait_state;         /* +0x0124 */
    uint8_t unknown_0125[0x07];         /* +0x0125 */
} da_bulletin_session;                  /* 0x012C bytes */

typedef struct da_users_dialog_pane {
    da_pane_prefix pane;                /* +0x0000 */
    uint8_t unknown_00f4[0x45c];        /* +0x00F4 */
    da_ptr32 row_or_list_objects[9];    /* +0x0550 */
    da_ptr32 list_storage_object;       /* +0x0574 */
    uint8_t unknown_0578;               /* +0x0578 */
    uint8_t unknown_0579;               /* +0x0579 */
    uint8_t unknown_057a[0x02];         /* +0x057A */
    da_ptr32 scrolling_control;         /* +0x057C */
    uint8_t unknown_0580;               /* +0x0580 */
    uint8_t unknown_0581;               /* +0x0581 */
    uint8_t unknown_0582[0x06];         /* +0x0582 */
} da_users_dialog_pane;                 /* 0x0588 bytes */
```

`self_look_7d4` is represented as the established minimum `0x100`-byte region. The following unknown range keeps later fields at their confirmed offsets without asserting that the original source declared exactly that array size.

## Local user movement timing

`ui_local_user` at `Darkages.exe:0x004F5198` points to a `0x776C`-byte object while the game UI is active. Only the timing slice needed for movement and the lag indicator is named here.

```c
typedef struct da_local_user_timing_view {
    uint8_t unknown_0000[0x6f28];       /* +0x0000 */
    uint8_t move_send_sequence;         /* +0x6F28 */
    uint8_t move_sequence_at_last_smove;/* +0x6F29 */
    uint8_t unknown_6f2a[0x02];         /* +0x6F2A */
    uint32_t last_move_send_tick;       /* +0x6F2C */
    uint8_t unknown_6f30[0x04];         /* +0x6F30 */
    uint32_t last_move_round_trip_ms;   /* +0x6F34 */
    uint8_t unknown_6f38[0x834];        /* +0x6F38 */
} da_local_user_timing_view;            /* 0x776C bytes */

#pragma pack(pop)
```

`last_move_send_tick` is overwritten on every `CMove`, so `last_move_round_trip_ms` is not a per-sequence latency table. It is the unsigned `timeGetTime` difference between an `SMove` and the most recently queued `CMove`. The lag pane stores a separate smoothed value at its own `+0xF4`.

## Validation checklist

Before interpreting any remote bytes:

1. Resolve the loaded module base and add the documented RVA.
2. Read the static pointer and reject null outside the expected lifecycle phase.
3. Confirm that the complete requested range is readable.
4. Validate the first dword against the expected relocated vtable where one is known.
5. Copy bounded strings and require a NUL terminator inside the published buffer.
6. Re-read the static pointer and vtable after the copy. Retry if either changed.
7. Treat packed hierarchy nodes and odd socket offsets as unaligned data.

Raw writes are not structure-aware UI operations. Use these layouts for observation first. A controlled writer should invoke the established client method on the client’s UI or event path so that registries, invalidation, focus, timers, and ownership remain consistent.
