# Server Actions

This index records every fixed action accepted by at least one pane's socket-event handler. The [server message directory](server-messages.md) expands each row into a dedicated message section with its transform status, placeholder wire shape, and known handler functions.

The `S...` names are direction-prefixed reference names supplied from later-client dumps. They are not RTTI or symbols recovered from the Stone executable, and a matching action value does not by itself establish that the later payload schema is identical. Each row separately records what is established in the 4.21 code. The supplied list plus `SChangeDirection` at `0x11` matches all 59 fixed server action values found in Stone, with no additional fixed server action found.

All entries use the server-to-client direction. Actions `0x00`, `0x03`, and `0x40` bypass the receive sequence and XOR decoder. Other entries are decoded before the Event type 9 packet is dispatched through pane virtual slot `+0x40`.

## Fixed action index

| Action | Message name | Stone accepting handlers and established notes |
|---:|---|---|
| `0x00` | `SVersionCheck` | `Darkages.exe:0x0045F780` `ui_main_menu_handle_server_packet`. Subtype zero reads server-version data, requires a nine-byte key, and updates the runtime XOR key. XOR bypass. Later-client scope: login or lobby. |
| `0x01` | `SNewUserCheck` | `Darkages.exe:0x00461080` `sub_461080`. Later-client scope: login or lobby. |
| `0x02` | `SLoginCheck` | `Darkages.exe:0x00461080` `sub_461080`, `Darkages.exe:0x0045F780` `ui_main_menu_handle_server_packet`, `Darkages.exe:0x00464300` `sub_464300`, `Darkages.exe:0x00463A60` `sub_463A60`. Later-client scope: login or lobby. |
| `0x03` | `STransferServer` | `Darkages.exe:0x0045F780` `ui_main_menu_handle_server_packet` and `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. The `MapPane` branch calls transfer-server processing and can send `kClientTransferServer`. XOR bypass. |
| `0x04` | `SUserPosition` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, `Darkages.exe:0x004889F0` `sub_4889F0`. |
| `0x05` | `SUserAppearance` | `Darkages.exe:0x004889F0` `sub_4889F0`. |
| `0x06` | `SMap` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x07` | `SDrawObjects` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x08` | `SStatus` | `Darkages.exe:0x0043F420` `sub_43F420`, `Darkages.exe:0x00441850` `sub_441850`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, `Darkages.exe:0x004889F0` `sub_4889F0`, `Darkages.exe:0x004927F0` `sub_4927F0`, `Darkages.exe:0x00494C60` `sub_494C60`. |
| `0x0A` | `SMessage` | `Darkages.exe:0x0041CCA0` `sub_41CCA0`, `Darkages.exe:0x00444690` `sub_444690`, `Darkages.exe:0x0045F780` `ui_main_menu_handle_server_packet`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, `Darkages.exe:0x00491520` `sub_491520`. |
| `0x0B` | `SMove` | `Darkages.exe:0x004889F0` `sub_4889F0`. |
| `0x0C` | `SMoveObject` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x0D` | `SSay` | `Darkages.exe:0x0041CCA0` `sub_41CCA0`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x0E` | `SRemoveObjects` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. The mapped branch removes an object from map state. |
| `0x0F` | `SAddItem` | `Darkages.exe:0x00441850` `sub_441850`, `Darkages.exe:0x004889F0` `sub_4889F0`. |
| `0x10` | `SRemoveItem` | `Darkages.exe:0x00441850` `sub_441850`, `Darkages.exe:0x004889F0` `sub_4889F0`. |
| `0x11` | `SChangeDirection` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, which calls `Darkages.exe:0x0046B574` `ui_map_handle_object_direction`. Offset `0x01` is a big-endian object ID and offset `0x05` is a direction value from 0 through 3. |
| `0x13` | `SDamageEffect` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x15` | `SMapInfo` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x17` | `SAddSpell` | `Darkages.exe:0x00443B10` `sub_443B10`, `Darkages.exe:0x004889F0` `sub_4889F0`. |
| `0x18` | `SRemoveSpell` | `Darkages.exe:0x00443B10` `sub_443B10`, `Darkages.exe:0x004889F0` `sub_4889F0`. |
| `0x19` | `SSoundEffect` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x1A` | `SMotion` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x1B` | `SEnterEditingMode` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. Later-client context: paper editing. |
| `0x1F` | `SChangeWeather` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x20` | `SChangeHour` | `Darkages.exe:0x0043F420` `sub_43F420`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x21` | `SSelfSaveOk` | `Darkages.exe:0x004909E0` `sub_4909E0`. |
| `0x26` | `SActionChange` | `Darkages.exe:0x004234E0` `sub_4234E0`. |
| `0x29` | `SEffectLayer` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x2A` | `SAddContainer` | `Darkages.exe:0x00425370` `sub_425370`, `Darkages.exe:0x00441850` `sub_441850`. |
| `0x2B` | `SRemoveContainer` | `Darkages.exe:0x00425370` `sub_425370`. |
| `0x2C` | `SAddSkill` | `Darkages.exe:0x00442990` `sub_442990`. |
| `0x2D` | `SRemoveSkill` | `Darkages.exe:0x00442990` `sub_442990`. |
| `0x2E` | `SFieldMap` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x2F` | `SScreenMenu` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, `Darkages.exe:0x00479480` `sub_479480`, `Darkages.exe:0x0047AFD0` `sub_47AFD0`. |
| `0x30` | `SPursuitMenu` | `Darkages.exe:0x00461080` `sub_461080`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, `Darkages.exe:0x00479480` `sub_479480`. |
| `0x31` | `SBulletin` | `Darkages.exe:0x00413CE0` `sub_413CE0`, `Darkages.exe:0x00414260` `sub_414260`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x32` | `SStateObjectState` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. The mapped branch performs inline tile or object updates. |
| `0x33` | `SDrawHumanObject` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x34` | `SObjectInfo` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x35` | `SShowPaper` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x36` | `SShowUsers` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x37` | `SAddEquip` | `Darkages.exe:0x0042EAF0` `sub_42EAF0`. |
| `0x38` | `SRemoveEquip` | `Darkages.exe:0x0042EAF0` `sub_42EAF0`. |
| `0x39` | `SSelfLook` | `Darkages.exe:0x0042EAF0` `sub_42EAF0`. |
| `0x3A` | `SSpelled` | `Darkages.exe:0x0043E8B0` `sub_43E8B0`. |
| `0x3B` | `SRequestCRC` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x3C` | `SMapPart` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x3D` | `SLevelPoint` | `Darkages.exe:0x0043F420` `sub_43F420`. |
| `0x3E` | `SWindowChange` | `Darkages.exe:0x0043D650` `sub_43D650`. |
| `0x3F` | `SActionCoolTime` | `Darkages.exe:0x00442990` `sub_442990`, `Darkages.exe:0x00443B10` `sub_443B10`. |
| `0x40` | `SSendPatch` | `Darkages.exe:0x004955C0` `sub_4955C0`. XOR bypass. The later-client scope may be login or lobby. |
| `0x42` | `SExchange` | `Darkages.exe:0x00435EA0` `sub_435EA0`, `Darkages.exe:0x004374F0` `sub_4374F0`, `Darkages.exe:0x00437C70` `sub_437C70`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. Exchange-session handlers dispatch on a second byte with established subtypes 1 through 5. |
| `0x48` | `SSpellDelayCancel` | `Darkages.exe:0x004568B0` `sub_4568B0`. |
| `0x49` | `SRequestPortrait` | `Darkages.exe:0x0040B3A0` `ui_handle_server_request_portrait`. An embedded diagnostic names `kServerRequestPortrait` and states decimal value 73. The handler initiates client action `0x4F`. |
| `0x4B` | `SBounce` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, which calls `Darkages.exe:0x0046CA54` `net_forward_embedded_client_packet`. Offset `0x01` is a big-endian embedded length and offset `0x03` begins the logical client packet. |
| `0x4C` | `SReconnect` | `Darkages.exe:0x004921F0` `ui_exit_wait_handle_server_packet`. The Stone handler consumes all `0x4C` packets. Subtype `0x01` sends `CQuit` subtype `0x00` and changes an exit-wait pane to the safe-exit message; it does not call the Socket reconnect path. |
| `0x51` | `SBlockInput` | `Darkages.exe:0x004889F0` `sub_4889F0`. |
| `0x56` | `SMulti` | `Darkages.exe:0x004A2ED0` `ui_server_select_handle_server_packet`. `ServerSelectDialogPane` reads a count, allocates replacement entry storage, and deserializes fixed-width integers and colon-delimited strings before loading the table. |

## `MapPane` switch coverage

`Darkages.exe:0x00468A90` accepts these 31 values:

```text
03 04 06 07 08 0A 0C 0D 0E 11 13 15 19 1A 1B 1F
20 29 2E 2F 30 31 32 33 34 35 36 3B 3C 42 4B
```

Within its explicit `0x03` through `0x4B` switch range, the other values return unhandled. A value can still be supported by another pane, as shown in the full index above.
