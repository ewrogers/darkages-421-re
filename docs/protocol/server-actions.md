# Server Actions

This index records every fixed action accepted by at least one pane's socket-event handler. A neutral name means the action comparison and handler are established but the packet's game meaning has not yet reached the documentation threshold.

All entries use the server-to-client direction. Actions `0x00`, `0x40`, and `0x03` bypass the receive sequence and XOR decoder. Other entries are decoded before the Event type 9 packet is dispatched through pane virtual slot `+0x40`.

## Fixed action index

| Action | Working name | Accepting handlers and established notes |
|---:|---|---|
| `0x00` | `server_version_and_key_setup` | `Darkages.exe:0x0045F780` `ui_main_menu_handle_server_packet`. Subtype zero reads server-version data, requires a nine-byte key, and updates the runtime XOR key. XOR bypass. |
| `0x01` | `server_action_01` | `Darkages.exe:0x00461080` `sub_461080`. |
| `0x02` | `server_action_02` | `Darkages.exe:0x00461080` `sub_461080`, `Darkages.exe:0x0045F780` `ui_main_menu_handle_server_packet`, `Darkages.exe:0x00464300` `sub_464300`, `Darkages.exe:0x00463A60` `sub_463A60`. |
| `0x03` | `server_transfer_server` | `Darkages.exe:0x0045F780` `ui_main_menu_handle_server_packet` and `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. The `MapPane` branch calls transfer-server processing and can send `kClientTransferServer`. XOR bypass. |
| `0x04` | `server_action_04` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, `Darkages.exe:0x004889F0` `sub_4889F0`. |
| `0x05` | `server_action_05` | `Darkages.exe:0x004889F0` `sub_4889F0`. |
| `0x06` | `server_action_06` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x07` | `server_action_07` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x08` | `server_action_08` | `Darkages.exe:0x0043F420` `sub_43F420`, `Darkages.exe:0x00441850` `sub_441850`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, `Darkages.exe:0x004889F0` `sub_4889F0`, `Darkages.exe:0x004927F0` `sub_4927F0`, `Darkages.exe:0x00494C60` `sub_494C60`. |
| `0x0A` | `server_action_0a` | `Darkages.exe:0x0041CCA0` `sub_41CCA0`, `Darkages.exe:0x00444690` `sub_444690`, `Darkages.exe:0x0045F780` `ui_main_menu_handle_server_packet`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, `Darkages.exe:0x00491520` `sub_491520`. |
| `0x0B` | `server_action_0b` | `Darkages.exe:0x004889F0` `sub_4889F0`. |
| `0x0C` | `server_action_0c` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x0D` | `server_action_0d` | `Darkages.exe:0x0041CCA0` `sub_41CCA0`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x0E` | `server_action_0e` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. The mapped branch removes an object from map state. |
| `0x0F` | `server_action_0f` | `Darkages.exe:0x00441850` `sub_441850`, `Darkages.exe:0x004889F0` `sub_4889F0`. |
| `0x10` | `server_action_10` | `Darkages.exe:0x00441850` `sub_441850`, `Darkages.exe:0x004889F0` `sub_4889F0`. |
| `0x11` | `server_action_11` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x13` | `server_action_13` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x15` | `server_action_15` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x17` | `server_action_17` | `Darkages.exe:0x00443B10` `sub_443B10`, `Darkages.exe:0x004889F0` `sub_4889F0`. |
| `0x18` | `server_action_18` | `Darkages.exe:0x00443B10` `sub_443B10`, `Darkages.exe:0x004889F0` `sub_4889F0`. |
| `0x19` | `server_action_19` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x1A` | `server_action_1a` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x1B` | `server_action_1b` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x1F` | `server_action_1f` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x20` | `server_action_20` | `Darkages.exe:0x0043F420` `sub_43F420`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x21` | `server_action_21` | `Darkages.exe:0x004909E0` `sub_4909E0`. |
| `0x26` | `server_action_26` | `Darkages.exe:0x004234E0` `sub_4234E0`. |
| `0x29` | `server_action_29` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x2A` | `server_action_2a` | `Darkages.exe:0x00425370` `sub_425370`, `Darkages.exe:0x00441850` `sub_441850`. |
| `0x2B` | `server_action_2b` | `Darkages.exe:0x00425370` `sub_425370`. |
| `0x2C` | `server_action_2c` | `Darkages.exe:0x00442990` `sub_442990`. |
| `0x2D` | `server_action_2d` | `Darkages.exe:0x00442990` `sub_442990`. |
| `0x2E` | `server_action_2e` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x2F` | `server_action_2f` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, `Darkages.exe:0x00479480` `sub_479480`, `Darkages.exe:0x0047AFD0` `sub_47AFD0`. |
| `0x30` | `server_action_30` | `Darkages.exe:0x00461080` `sub_461080`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, `Darkages.exe:0x00479480` `sub_479480`. |
| `0x31` | `server_action_31` | `Darkages.exe:0x00413CE0` `sub_413CE0`, `Darkages.exe:0x00414260` `sub_414260`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x32` | `server_action_32` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. The mapped branch performs inline tile or object updates. |
| `0x33` | `server_action_33` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x34` | `server_action_34` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x35` | `server_action_35` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x36` | `server_action_36` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x37` | `server_action_37` | `Darkages.exe:0x0042EAF0` `sub_42EAF0`. |
| `0x38` | `server_action_38` | `Darkages.exe:0x0042EAF0` `sub_42EAF0`. |
| `0x39` | `server_action_39` | `Darkages.exe:0x0042EAF0` `sub_42EAF0`. |
| `0x3A` | `server_action_3a` | `Darkages.exe:0x0043E8B0` `sub_43E8B0`. |
| `0x3B` | `server_action_3b` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x3C` | `server_action_3c` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. |
| `0x3D` | `server_action_3d` | `Darkages.exe:0x0043F420` `sub_43F420`. |
| `0x3E` | `server_action_3e` | `Darkages.exe:0x0043D650` `sub_43D650`. |
| `0x3F` | `server_action_3f` | `Darkages.exe:0x00442990` `sub_442990`, `Darkages.exe:0x00443B10` `sub_443B10`. |
| `0x40` | `server_action_40` | `Darkages.exe:0x004955C0` `sub_4955C0`. XOR bypass. |
| `0x42` | `server_exchange_session` | `Darkages.exe:0x00435EA0` `sub_435EA0`, `Darkages.exe:0x004374F0` `sub_4374F0`, `Darkages.exe:0x00437C70` `sub_437C70`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. Exchange-session handlers dispatch on a second byte with established subtypes 1 through 5. |
| `0x48` | `server_action_48` | `Darkages.exe:0x004568B0` `sub_4568B0`. |
| `0x49` | `server_request_portrait` | `Darkages.exe:0x0040B3A0` `ui_handle_server_request_portrait`. An embedded diagnostic names `kServerRequestPortrait` and states decimal value 73. The handler initiates client action `0x4F`. |
| `0x4B` | `server_forward_client_packet` | `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, which calls `Darkages.exe:0x0046CA54` `net_forward_embedded_client_packet`. Offset `0x01` is a big-endian embedded length and offset `0x03` begins the logical client packet. |
| `0x4C` | `server_action_4c` | `Darkages.exe:0x004921F0` `sub_4921F0`. |
| `0x51` | `server_action_51` | `Darkages.exe:0x004889F0` `sub_4889F0`. |
| `0x56` | `server_list` | `Darkages.exe:0x004A2ED0` `ui_server_select_handle_server_packet`. `ServerSelectDialogPane` reads a count, allocates replacement entry storage, and deserializes fixed-width integers and colon-delimited strings before loading the table. |

## `MapPane` switch coverage

`Darkages.exe:0x00468A90` accepts these 31 values:

```text
03 04 06 07 08 0A 0C 0D 0E 11 13 15 19 1A 1B 1F
20 29 2E 2F 30 31 32 33 34 35 36 3B 3C 42 4B
```

Within its explicit `0x03` through `0x4B` switch range, the other values return unhandled. A value can still be supported by another pane, as shown in the full index above.
