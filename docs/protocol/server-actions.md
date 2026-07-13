# Server Actions

This index records every fixed action accepted by at least one pane's socket-event handler. Each class name links to a dedicated page containing its payload format, handler functions, and current schema status. Message-page offsets are payload-relative and exclude framing, action, sequence, and the trailing zero.

The `S...` class names identify the 59 fixed server-to-client actions accepted by the traced 4.21 handlers. No additional fixed server action was found.

All entries use the server-to-client direction. Actions `0x00`, `0x03`, and `0x40` bypass the receive sequence and XOR decoder. Other entries are decoded before the Event type 9 packet is dispatched through pane virtual slot `+0x40`.

## Fixed action index

| Action | Class name | Encrypted | 4.21 handlers and notes |
|---:|---|---|---|
| `0x00` | [`SVersionCheck`](server/000-00-version-check.md) | No | `Darkages.exe:0x0045F780` `ui_main_menu_handle_server_packet`. Subtype zero reads the server version, selects table function 0 through 9, requires a nine-byte key, and queues both transformation updates. XOR bypass. Used during login or lobby. |
| `0x01` | [`SNewUserCheck`](server/001-01-new-user-check.md) | Yes | `Darkages.exe:0x00461080` `sub_461080`. Used during login or lobby. |
| `0x02` | [`SLoginCheck`](server/002-02-login-check.md) | Yes | `Darkages.exe:0x00461080` `sub_461080`, `Darkages.exe:0x0045F780` `ui_main_menu_handle_server_packet`, `Darkages.exe:0x00464300` `sub_464300`, `Darkages.exe:0x00463A60` `sub_463A60`. Used during login or lobby. |
| `0x03` | [`STransferServer`](server/003-03-transfer-server.md) | No | `Darkages.exe:0x0045F780` `ui_main_menu_handle_server_packet` and `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. The `MapPane` branch calls transfer-server processing and can send `kClientTransferServer`. XOR bypass. |
| `0x04` | [`SUserPosition`](server/004-04-user-position.md) | Yes | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, `Darkages.exe:0x004889F0` `sub_4889F0`. |
| `0x05` | [`SUserAppearance`](server/005-05-user-appearance.md) | Yes | `Darkages.exe:0x004889F0` `sub_4889F0`. |
| `0x06` | [`SMap`](server/006-06-map.md) | Yes | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x07` | [`SDrawObjects`](server/007-07-draw-objects.md) | Yes | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x08` | [`SStatus`](server/008-08-status.md) | Yes | `Darkages.exe:0x0043F420` `sub_43F420`, `Darkages.exe:0x00441850` `sub_441850`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, `Darkages.exe:0x004889F0` `sub_4889F0`, `Darkages.exe:0x004927F0` `sub_4927F0`, `Darkages.exe:0x00494C60` `sub_494C60`. |
| `0x0A` | [`SMessage`](server/010-0a-message.md) | Yes | `Darkages.exe:0x0041CCA0` `sub_41CCA0`, `Darkages.exe:0x00444690` `sub_444690`, `Darkages.exe:0x0045F780` `ui_main_menu_handle_server_packet`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, `Darkages.exe:0x00491520` `sub_491520`. |
| `0x0B` | [`SMove`](server/011-0b-move.md) | Yes | `Darkages.exe:0x004889F0` `sub_4889F0`. |
| `0x0C` | [`SMoveObject`](server/012-0c-move-object.md) | Yes | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x0D` | [`SSay`](server/013-0d-say.md) | Yes | `Darkages.exe:0x0041CCA0` `sub_41CCA0`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x0E` | [`SRemoveObjects`](server/014-0e-remove-objects.md) | Yes | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. The mapped branch removes an object from map state. |
| `0x0F` | [`SAddItem`](server/015-0f-add-item.md) | Yes | `Darkages.exe:0x00441850` `sub_441850`, `Darkages.exe:0x004889F0` `sub_4889F0`. |
| `0x10` | [`SRemoveItem`](server/016-10-remove-item.md) | Yes | `Darkages.exe:0x00441850` `sub_441850`, `Darkages.exe:0x004889F0` `sub_4889F0`. |
| `0x11` | [`SChangeDirection`](server/017-11-change-direction.md) | Yes | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, which calls `Darkages.exe:0x0046B574` `ui_map_handle_object_direction`. Offset `0x01` is a big-endian object ID and offset `0x05` is a direction value from 0 through 3. |
| `0x13` | [`SDamageEffect`](server/019-13-damage-effect.md) | Yes | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x15` | [`SMapInfo`](server/021-15-map-info.md) | Yes | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x17` | [`SAddSpell`](server/023-17-add-spell.md) | Yes | `Darkages.exe:0x00443B10` `ui_spell_inventory_handle_server_packet`, `Darkages.exe:0x004889F0` `sub_4889F0`. |
| `0x18` | [`SRemoveSpell`](server/024-18-remove-spell.md) | Yes | `Darkages.exe:0x00443B10` `ui_spell_inventory_handle_server_packet`, `Darkages.exe:0x004889F0` `sub_4889F0`. |
| `0x19` | [`SSoundEffect`](server/025-19-sound-effect.md) | Yes | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x1A` | [`SMotion`](server/026-1a-motion.md) | Yes | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x1B` | [`SEnterEditingMode`](server/027-1b-enter-editing-mode.md) | Yes | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. Used for paper editing. |
| `0x1F` | [`SChangeWeather`](server/031-1f-change-weather.md) | Yes | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x20` | [`SChangeHour`](server/032-20-change-hour.md) | Yes | `Darkages.exe:0x0043F420` `sub_43F420`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x21` | [`SSelfSaveOk`](server/033-21-self-save-ok.md) | Yes | `Darkages.exe:0x004909E0` `ui_option_pane_handle_server_packet`. |
| `0x26` | [`SActionChange`](server/038-26-action-change.md) | Yes | `Darkages.exe:0x004234E0` `sub_4234E0`. |
| `0x29` | [`SEffectLayer`](server/041-29-effect-layer.md) | Yes | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x2A` | [`SAddContainer`](server/042-2a-add-container.md) | Yes | `Darkages.exe:0x00425370` `sub_425370`, `Darkages.exe:0x00441850` `sub_441850`. |
| `0x2B` | [`SRemoveContainer`](server/043-2b-remove-container.md) | Yes | `Darkages.exe:0x00425370` `sub_425370`. |
| `0x2C` | [`SAddSkill`](server/044-2c-add-skill.md) | Yes | `Darkages.exe:0x00442990` `ui_skill_inventory_handle_server_packet`. |
| `0x2D` | [`SRemoveSkill`](server/045-2d-remove-skill.md) | Yes | `Darkages.exe:0x00442990` `ui_skill_inventory_handle_server_packet`. |
| `0x2E` | [`SFieldMap`](server/046-2e-field-map.md) | Yes | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x2F` | [`SScreenMenu`](server/047-2f-screen-menu.md) | Yes | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, `Darkages.exe:0x00479480` `sub_479480`, `Darkages.exe:0x0047AFD0` `sub_47AFD0`. |
| `0x30` | [`SPursuitMenu`](server/048-30-pursuit-menu.md) | Yes | `Darkages.exe:0x00461080` `sub_461080`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, `Darkages.exe:0x00479480` `sub_479480`. |
| `0x31` | [`SBulletin`](server/049-31-bulletin.md) | Yes | `Darkages.exe:0x00413CE0` `ui_bulletin_session_handle_server_packet`, `Darkages.exe:0x00414260` `ui_bulletin_dialog_handle_server_packet`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x32` | [`SStateObjectState`](server/050-32-state-object-state.md) | Yes | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. The mapped branch performs inline tile or object updates. |
| `0x33` | [`SDrawHumanObject`](server/051-33-draw-human-object.md) | Yes | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x34` | [`SObjectInfo`](server/052-34-object-info.md) | Yes | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x35` | [`SShowPaper`](server/053-35-show-paper.md) | Yes | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x36` | [`SShowUsers`](server/054-36-show-users.md) | Yes | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x37` | [`SAddEquip`](server/055-37-add-equip.md) | Yes | `Darkages.exe:0x0042EAF0` `ui_equip_pane_handle_server_packet`. |
| `0x38` | [`SRemoveEquip`](server/056-38-remove-equip.md) | Yes | `Darkages.exe:0x0042EAF0` `ui_equip_pane_handle_server_packet`. |
| `0x39` | [`SSelfLook`](server/057-39-self-look.md) | Yes | `Darkages.exe:0x0042EAF0` `ui_equip_pane_handle_server_packet`. |
| `0x3A` | [`SSpelled`](server/058-3a-spelled.md) | Yes | `Darkages.exe:0x0043E8B0` `sub_43E8B0`. |
| `0x3B` | [`SRequestCRC`](server/059-3b-request-crc.md) | Yes | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x3C` | [`SMapPart`](server/060-3c-map-part.md) | Yes | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x3D` | [`SLevelPoint`](server/061-3d-level-point.md) | Yes | `Darkages.exe:0x0043F420` `sub_43F420`. |
| `0x3E` | [`SWindowChange`](server/062-3e-window-change.md) | Yes | `Darkages.exe:0x0043D650` `sub_43D650`. |
| `0x3F` | [`SActionCoolTime`](server/063-3f-action-cool-time.md) | Yes | `Darkages.exe:0x00442990` `ui_skill_inventory_handle_server_packet`, `Darkages.exe:0x00443B10` `ui_spell_inventory_handle_server_packet`. |
| `0x40` | [`SSendPatch`](server/064-40-send-patch.md) | No | `Darkages.exe:0x004955C0` `sub_4955C0`. XOR bypass. May be used during login or lobby. |
| `0x42` | [`SExchange`](server/066-42-exchange.md) | Yes | `Darkages.exe:0x00435EA0` `sub_435EA0`, `Darkages.exe:0x004374F0` `sub_4374F0`, `Darkages.exe:0x00437C70` `sub_437C70`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. Exchange-session handlers dispatch on a second byte with established subtypes 1 through 5. |
| `0x48` | [`SSpellDelayCancel`](server/072-48-spell-delay-cancel.md) | Yes | `Darkages.exe:0x004568B0` `sub_4568B0`. |
| `0x49` | [`SRequestPortrait`](server/073-49-request-portrait.md) | Yes | `Darkages.exe:0x0040B3A0` `ui_handle_server_request_portrait`. An embedded diagnostic names `kServerRequestPortrait` and states decimal value 73. The handler initiates client action `0x4F`. |
| `0x4B` | [`SBounce`](server/075-4b-bounce.md) | Yes | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, which calls `Darkages.exe:0x0046CA54` `net_forward_embedded_client_packet`. Offset `0x01` is a big-endian embedded length and offset `0x03` begins the logical client packet. |
| `0x4C` | [`SReconnect`](server/076-4c-reconnect.md) | Yes | `Darkages.exe:0x004921F0` `ui_exit_wait_handle_server_packet`. The Stone handler consumes all `0x4C` packets. Subtype `0x01` sends `CQuit` subtype `0x00` and changes an exit-wait pane to the safe-exit message; it does not call the Socket reconnect path. |
| `0x51` | [`SBlockInput`](server/081-51-block-input.md) | Yes | `Darkages.exe:0x004889F0` `sub_4889F0`. |
| `0x56` | [`SMulti`](server/086-56-multi.md) | Yes | `Darkages.exe:0x004A2ED0` `ui_server_select_handle_server_packet`. `ServerSelectDialogPane` reads a count, allocates replacement entry storage, and deserializes fixed-width integers and colon-delimited strings before loading the table. |

## `MapPane` switch coverage

`Darkages.exe:0x00468A90` accepts these 31 values:

```text
03 04 06 07 08 0A 0C 0D 0E 11 13 15 19 1A 1B 1F
20 29 2E 2F 30 31 32 33 34 35 36 3B 3C 42 4B
```

Within its explicit `0x03` through `0x4B` switch range, the other values return unhandled. A value can still be supported by another pane, as shown in the full index above.
