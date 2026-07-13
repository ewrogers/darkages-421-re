# Server Message Directory

This directory expands the compact [server action index](server-actions.md) into one section per fixed server-to-client action. It records the Stone functions that accept each action even when a function still has a neutral `sub_...` IDA name.

The `S...` identifiers are message names supplied from later-client dumps. They are not RTTI or symbols recovered from Stone. Placeholder fields identify only established offsets and ownership boundaries; they do not assign later-version payload meanings to the 4.21 client.

## Common shape conventions

In this directory, `Encrypted: Yes` is protocol shorthand for a wire body that carries a sequence byte and whose payload and sentinel are XOR-transformed. The Stone receive decoder removes that sequence byte and reverses the XOR passes before Event dispatch. `Encrypted: No` means the sequence byte and XOR passes are both omitted. The label describes the protocol branch and does not claim cryptographic security.

```text
Encrypted: [action] [sequence] [XOR(payload || 00)]
Not encrypted: [action] [payload] [00]
After receive decoding: [action] [payload] [00]
```

The Event packet size covers the decoded application packet, including its protocol sentinel. The decoder also writes a local zero guard immediately after the reported packet bytes. Generic `unknown_01` rows keep the payload opaque until its readers establish field boundaries, widths, byte order, string encoding, and validation.

## Directory

| Action | Message name | Encrypted | Mapped functions |
|---:|---|---|---:|
| `0x00` | [`SVersionCheck`](#0x00-sversioncheck) | No | 1 |
| `0x01` | [`SNewUserCheck`](#0x01-snewusercheck) | Yes | 1 |
| `0x02` | [`SLoginCheck`](#0x02-slogincheck) | Yes | 4 |
| `0x03` | [`STransferServer`](#0x03-stransferserver) | No | 2 |
| `0x04` | [`SUserPosition`](#0x04-suserposition) | Yes | 2 |
| `0x05` | [`SUserAppearance`](#0x05-suserappearance) | Yes | 1 |
| `0x06` | [`SMap`](#0x06-smap) | Yes | 1 |
| `0x07` | [`SDrawObjects`](#0x07-sdrawobjects) | Yes | 1 |
| `0x08` | [`SStatus`](#0x08-sstatus) | Yes | 6 |
| `0x0A` | [`SMessage`](#0x0a-smessage) | Yes | 5 |
| `0x0B` | [`SMove`](#0x0b-smove) | Yes | 1 |
| `0x0C` | [`SMoveObject`](#0x0c-smoveobject) | Yes | 1 |
| `0x0D` | [`SSay`](#0x0d-ssay) | Yes | 2 |
| `0x0E` | [`SRemoveObjects`](#0x0e-sremoveobjects) | Yes | 1 |
| `0x0F` | [`SAddItem`](#0x0f-sadditem) | Yes | 2 |
| `0x10` | [`SRemoveItem`](#0x10-sremoveitem) | Yes | 2 |
| `0x11` | [`SChangeDirection`](#0x11-schangedirection) | Yes | 2 |
| `0x13` | [`SDamageEffect`](#0x13-sdamageeffect) | Yes | 1 |
| `0x15` | [`SMapInfo`](#0x15-smapinfo) | Yes | 1 |
| `0x17` | [`SAddSpell`](#0x17-saddspell) | Yes | 2 |
| `0x18` | [`SRemoveSpell`](#0x18-sremovespell) | Yes | 2 |
| `0x19` | [`SSoundEffect`](#0x19-ssoundeffect) | Yes | 1 |
| `0x1A` | [`SMotion`](#0x1a-smotion) | Yes | 1 |
| `0x1B` | [`SEnterEditingMode`](#0x1b-sentereditingmode) | Yes | 1 |
| `0x1F` | [`SChangeWeather`](#0x1f-schangeweather) | Yes | 1 |
| `0x20` | [`SChangeHour`](#0x20-schangehour) | Yes | 2 |
| `0x21` | [`SSelfSaveOk`](#0x21-sselfsaveok) | Yes | 1 |
| `0x26` | [`SActionChange`](#0x26-sactionchange) | Yes | 1 |
| `0x29` | [`SEffectLayer`](#0x29-seffectlayer) | Yes | 1 |
| `0x2A` | [`SAddContainer`](#0x2a-saddcontainer) | Yes | 2 |
| `0x2B` | [`SRemoveContainer`](#0x2b-sremovecontainer) | Yes | 1 |
| `0x2C` | [`SAddSkill`](#0x2c-saddskill) | Yes | 1 |
| `0x2D` | [`SRemoveSkill`](#0x2d-sremoveskill) | Yes | 1 |
| `0x2E` | [`SFieldMap`](#0x2e-sfieldmap) | Yes | 1 |
| `0x2F` | [`SScreenMenu`](#0x2f-sscreenmenu) | Yes | 3 |
| `0x30` | [`SPursuitMenu`](#0x30-spursuitmenu) | Yes | 3 |
| `0x31` | [`SBulletin`](#0x31-sbulletin) | Yes | 3 |
| `0x32` | [`SStateObjectState`](#0x32-sstateobjectstate) | Yes | 1 |
| `0x33` | [`SDrawHumanObject`](#0x33-sdrawhumanobject) | Yes | 1 |
| `0x34` | [`SObjectInfo`](#0x34-sobjectinfo) | Yes | 1 |
| `0x35` | [`SShowPaper`](#0x35-sshowpaper) | Yes | 1 |
| `0x36` | [`SShowUsers`](#0x36-sshowusers) | Yes | 1 |
| `0x37` | [`SAddEquip`](#0x37-saddequip) | Yes | 1 |
| `0x38` | [`SRemoveEquip`](#0x38-sremoveequip) | Yes | 1 |
| `0x39` | [`SSelfLook`](#0x39-sselflook) | Yes | 1 |
| `0x3A` | [`SSpelled`](#0x3a-sspelled) | Yes | 1 |
| `0x3B` | [`SRequestCRC`](#0x3b-srequestcrc) | Yes | 1 |
| `0x3C` | [`SMapPart`](#0x3c-smappart) | Yes | 1 |
| `0x3D` | [`SLevelPoint`](#0x3d-slevelpoint) | Yes | 1 |
| `0x3E` | [`SWindowChange`](#0x3e-swindowchange) | Yes | 1 |
| `0x3F` | [`SActionCoolTime`](#0x3f-sactioncooltime) | Yes | 2 |
| `0x40` | [`SSendPatch`](#0x40-ssendpatch) | No | 1 |
| `0x42` | [`SExchange`](#0x42-sexchange) | Yes | 4 |
| `0x48` | [`SSpellDelayCancel`](#0x48-sspelldelaycancel) | Yes | 1 |
| `0x49` | [`SRequestPortrait`](#0x49-srequestportrait) | Yes | 1 |
| `0x4B` | [`SBounce`](#0x4b-sbounce) | Yes | 2 |
| `0x4C` | [`SReconnect`](#0x4c-sreconnect) | Yes | 2 |
| `0x51` | [`SBlockInput`](#0x51-sblockinput) | Yes | 1 |
| `0x56` | [`SMulti`](#0x56-smulti) | Yes | 1 |

## `0x00` `SVersionCheck`

**Direction:** server to client

**Encrypted:** No.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x00`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x0045F780` | `ui_main_menu_handle_server_packet` | Accepts the action in the main-menu packet handler. |

### Stone evidence

`Darkages.exe:0x0045F780` `ui_main_menu_handle_server_packet`. Subtype zero reads server-version data, requires a nine-byte key, and updates the runtime XOR key. XOR bypass. Later-client scope: login or lobby.

### Mapping status

The later-client `SVersionCheck` name and login or lobby scope agree with the Stone path. The handler's subtype-zero branch and key-update behavior are established, but the complete payload schema is not yet divided into fields.

## `0x01` `SNewUserCheck`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x01`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00461080` | `sub_461080` | Accepts the action in its pane's Event type 9 packet handler. |

### Stone evidence

`Darkages.exe:0x00461080` `sub_461080`. Later-client scope: login or lobby.

### Mapping status

The `SNewUserCheck` name is a later-client cross-version reference. Stone accepts action `0x01` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x02` `SLoginCheck`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x02`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00461080` | `sub_461080` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x0045F780` | `ui_main_menu_handle_server_packet` | Accepts the action in the main-menu packet handler. |
| `Darkages.exe:0x00464300` | `sub_464300` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x00463A60` | `sub_463A60` | Accepts the action in its pane's Event type 9 packet handler. |

### Stone evidence

`Darkages.exe:0x00461080` `sub_461080`, `Darkages.exe:0x0045F780` `ui_main_menu_handle_server_packet`, `Darkages.exe:0x00464300` `sub_464300`, `Darkages.exe:0x00463A60` `sub_463A60`. Later-client scope: login or lobby.

### Mapping status

The `SLoginCheck` name is a later-client cross-version reference. Stone accepts action `0x02` in the listed functions. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x03` `STransferServer`

**Direction:** server to client

**Encrypted:** No.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x03`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x0045F780` | `ui_main_menu_handle_server_packet` | Accepts the action in the main-menu packet handler. |
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

### Stone evidence

`Darkages.exe:0x0045F780` `ui_main_menu_handle_server_packet` and `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. The `MapPane` branch calls transfer-server processing and can send `kClientTransferServer`. XOR bypass.

### Mapping status

The `STransferServer` name is a later-client cross-version reference. Stone accepts action `0x03` in the listed functions. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x04` `SUserPosition`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x04`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |
| `Darkages.exe:0x004889F0` | `sub_4889F0` | Accepts the action in its pane's Event type 9 packet handler. |

### Stone evidence

`Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, `Darkages.exe:0x004889F0` `sub_4889F0`.

### Mapping status

The `SUserPosition` name is a later-client cross-version reference. Stone accepts action `0x04` in the listed functions. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x05` `SUserAppearance`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x05`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x004889F0` | `sub_4889F0` | Accepts the action in its pane's Event type 9 packet handler. |

### Stone evidence

`Darkages.exe:0x004889F0` `sub_4889F0`.

### Mapping status

The `SUserAppearance` name is a later-client cross-version reference. Stone accepts action `0x05` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x06` `SMap`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x06`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

### Stone evidence

`Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`.

### Mapping status

The `SMap` name is a later-client cross-version reference. Stone accepts action `0x06` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x07` `SDrawObjects`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x07`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

### Stone evidence

`Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`.

### Mapping status

The `SDrawObjects` name is a later-client cross-version reference. Stone accepts action `0x07` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x08` `SStatus`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x08`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x0043F420` | `sub_43F420` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x00441850` | `sub_441850` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |
| `Darkages.exe:0x004889F0` | `sub_4889F0` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x004927F0` | `sub_4927F0` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x00494C60` | `sub_494C60` | Accepts the action in its pane's Event type 9 packet handler. |

### Stone evidence

`Darkages.exe:0x0043F420` `sub_43F420`, `Darkages.exe:0x00441850` `sub_441850`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, `Darkages.exe:0x004889F0` `sub_4889F0`, `Darkages.exe:0x004927F0` `sub_4927F0`, `Darkages.exe:0x00494C60` `sub_494C60`.

### Mapping status

The `SStatus` name is a later-client cross-version reference. Stone accepts action `0x08` in the listed functions. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x0A` `SMessage`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x0A`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x0041CCA0` | `sub_41CCA0` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x00444690` | `sub_444690` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x0045F780` | `ui_main_menu_handle_server_packet` | Accepts the action in the main-menu packet handler. |
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |
| `Darkages.exe:0x00491520` | `sub_491520` | Accepts the action in its pane's Event type 9 packet handler. |

### Stone evidence

`Darkages.exe:0x0041CCA0` `sub_41CCA0`, `Darkages.exe:0x00444690` `sub_444690`, `Darkages.exe:0x0045F780` `ui_main_menu_handle_server_packet`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, `Darkages.exe:0x00491520` `sub_491520`.

### Mapping status

The `SMessage` name is a later-client cross-version reference. Stone accepts action `0x0A` in the listed functions. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x0B` `SMove`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x0B`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x004889F0` | `sub_4889F0` | Accepts the action in its pane's Event type 9 packet handler. |

### Stone evidence

`Darkages.exe:0x004889F0` `sub_4889F0`.

### Mapping status

The `SMove` name is a later-client cross-version reference. Stone accepts action `0x0B` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x0C` `SMoveObject`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x0C`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

### Stone evidence

`Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`.

### Mapping status

The `SMoveObject` name is a later-client cross-version reference. Stone accepts action `0x0C` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x0D` `SSay`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x0D`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x0041CCA0` | `sub_41CCA0` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

### Stone evidence

`Darkages.exe:0x0041CCA0` `sub_41CCA0`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`.

### Mapping status

The `SSay` name is a later-client cross-version reference. Stone accepts action `0x0D` in the listed functions. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x0E` `SRemoveObjects`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x0E`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

### Stone evidence

`Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. The mapped branch removes an object from map state.

### Mapping status

The `SRemoveObjects` name is a later-client cross-version reference. Stone accepts action `0x0E` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x0F` `SAddItem`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x0F`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00441850` | `sub_441850` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x004889F0` | `sub_4889F0` | Accepts the action in its pane's Event type 9 packet handler. |

### Stone evidence

`Darkages.exe:0x00441850` `sub_441850`, `Darkages.exe:0x004889F0` `sub_4889F0`.

### Mapping status

The `SAddItem` name is a later-client cross-version reference. Stone accepts action `0x0F` in the listed functions. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x10` `SRemoveItem`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x10`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00441850` | `sub_441850` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x004889F0` | `sub_4889F0` | Accepts the action in its pane's Event type 9 packet handler. |

### Stone evidence

`Darkages.exe:0x00441850` `sub_441850`, `Darkages.exe:0x004889F0` `sub_4889F0`.

### Mapping status

The `SRemoveItem` name is a later-client cross-version reference. Stone accepts action `0x10` in the listed functions. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x11` `SChangeDirection`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x11`. |
| `0x01` | 4 | `object_id` | Unsigned big-endian object identifier. |
| `0x05` | 1 | `direction` | Direction value. The mapped handler applies values 0 through 3. |
| `0x06` | `packet_size - 6` | `unknown_06` | Any trailing bytes and the protocol sentinel; this boundary is not yet divided further. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |
| `Darkages.exe:0x0046B574` | `ui_map_handle_object_direction` | Reads the object ID and direction and updates the matching map object. |

### Stone evidence

`Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, which calls `Darkages.exe:0x0046B574` `ui_map_handle_object_direction`. Offset `0x01` is a big-endian object ID and offset `0x05` is a direction value from 0 through 3.

### Mapping status

The later-client `SChangeDirection` name agrees with the Stone data flow. The handler reads `object_id`, locates the map object, and applies `direction` only when its value is from 0 through 3. This payload prefix is established directly from the Stone instructions.

## `0x13` `SDamageEffect`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x13`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

### Stone evidence

`Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`.

### Mapping status

The `SDamageEffect` name is a later-client cross-version reference. Stone accepts action `0x13` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x15` `SMapInfo`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x15`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

### Stone evidence

`Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`.

### Mapping status

The `SMapInfo` name is a later-client cross-version reference. Stone accepts action `0x15` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x17` `SAddSpell`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x17`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00443B10` | `sub_443B10` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x004889F0` | `sub_4889F0` | Accepts the action in its pane's Event type 9 packet handler. |

### Stone evidence

`Darkages.exe:0x00443B10` `sub_443B10`, `Darkages.exe:0x004889F0` `sub_4889F0`.

### Mapping status

The `SAddSpell` name is a later-client cross-version reference. Stone accepts action `0x17` in the listed functions. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x18` `SRemoveSpell`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x18`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00443B10` | `sub_443B10` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x004889F0` | `sub_4889F0` | Accepts the action in its pane's Event type 9 packet handler. |

### Stone evidence

`Darkages.exe:0x00443B10` `sub_443B10`, `Darkages.exe:0x004889F0` `sub_4889F0`.

### Mapping status

The `SRemoveSpell` name is a later-client cross-version reference. Stone accepts action `0x18` in the listed functions. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x19` `SSoundEffect`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x19`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

### Stone evidence

`Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`.

### Mapping status

The `SSoundEffect` name is a later-client cross-version reference. Stone accepts action `0x19` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x1A` `SMotion`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x1A`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

### Stone evidence

`Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`.

### Mapping status

The `SMotion` name is a later-client cross-version reference. Stone accepts action `0x1A` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x1B` `SEnterEditingMode`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x1B`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

### Stone evidence

`Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. Later-client context: paper editing.

### Mapping status

The `SEnterEditingMode` name is a later-client cross-version reference. Stone accepts action `0x1B` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x1F` `SChangeWeather`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x1F`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

### Stone evidence

`Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`.

### Mapping status

The `SChangeWeather` name is a later-client cross-version reference. Stone accepts action `0x1F` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x20` `SChangeHour`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x20`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x0043F420` | `sub_43F420` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

### Stone evidence

`Darkages.exe:0x0043F420` `sub_43F420`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`.

### Mapping status

The `SChangeHour` name is a later-client cross-version reference. Stone accepts action `0x20` in the listed functions. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x21` `SSelfSaveOk`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x21`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x004909E0` | `sub_4909E0` | Accepts the action in its pane's Event type 9 packet handler. |

### Stone evidence

`Darkages.exe:0x004909E0` `sub_4909E0`.

### Mapping status

The `SSelfSaveOk` name is a later-client cross-version reference. Stone accepts action `0x21` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x26` `SActionChange`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x26`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x004234E0` | `sub_4234E0` | Accepts the action in its pane's Event type 9 packet handler. |

### Stone evidence

`Darkages.exe:0x004234E0` `sub_4234E0`.

### Mapping status

The `SActionChange` name is a later-client cross-version reference. Stone accepts action `0x26` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x29` `SEffectLayer`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x29`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

### Stone evidence

`Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`.

### Mapping status

The `SEffectLayer` name is a later-client cross-version reference. Stone accepts action `0x29` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x2A` `SAddContainer`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x2A`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00425370` | `sub_425370` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x00441850` | `sub_441850` | Accepts the action in its pane's Event type 9 packet handler. |

### Stone evidence

`Darkages.exe:0x00425370` `sub_425370`, `Darkages.exe:0x00441850` `sub_441850`.

### Mapping status

The `SAddContainer` name is a later-client cross-version reference. Stone accepts action `0x2A` in the listed functions. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x2B` `SRemoveContainer`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x2B`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00425370` | `sub_425370` | Accepts the action in its pane's Event type 9 packet handler. |

### Stone evidence

`Darkages.exe:0x00425370` `sub_425370`.

### Mapping status

The `SRemoveContainer` name is a later-client cross-version reference. Stone accepts action `0x2B` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x2C` `SAddSkill`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x2C`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00442990` | `sub_442990` | Accepts the action in its pane's Event type 9 packet handler. |

### Stone evidence

`Darkages.exe:0x00442990` `sub_442990`.

### Mapping status

The `SAddSkill` name is a later-client cross-version reference. Stone accepts action `0x2C` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x2D` `SRemoveSkill`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x2D`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00442990` | `sub_442990` | Accepts the action in its pane's Event type 9 packet handler. |

### Stone evidence

`Darkages.exe:0x00442990` `sub_442990`.

### Mapping status

The `SRemoveSkill` name is a later-client cross-version reference. Stone accepts action `0x2D` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x2E` `SFieldMap`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x2E`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

### Stone evidence

`Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`.

### Mapping status

The `SFieldMap` name is a later-client cross-version reference. Stone accepts action `0x2E` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x2F` `SScreenMenu`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x2F`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |
| `Darkages.exe:0x00479480` | `sub_479480` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x0047AFD0` | `sub_47AFD0` | Accepts the action in its pane's Event type 9 packet handler. |

### Stone evidence

`Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, `Darkages.exe:0x00479480` `sub_479480`, `Darkages.exe:0x0047AFD0` `sub_47AFD0`.

### Mapping status

The `SScreenMenu` name is a later-client cross-version reference. Stone accepts action `0x2F` in the listed functions. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x30` `SPursuitMenu`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x30`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00461080` | `sub_461080` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |
| `Darkages.exe:0x00479480` | `sub_479480` | Accepts the action in its pane's Event type 9 packet handler. |

### Stone evidence

`Darkages.exe:0x00461080` `sub_461080`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, `Darkages.exe:0x00479480` `sub_479480`.

### Mapping status

The `SPursuitMenu` name is a later-client cross-version reference. Stone accepts action `0x30` in the listed functions. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x31` `SBulletin`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x31`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00413CE0` | `sub_413CE0` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x00414260` | `sub_414260` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

### Stone evidence

`Darkages.exe:0x00413CE0` `sub_413CE0`, `Darkages.exe:0x00414260` `sub_414260`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`.

### Mapping status

The `SBulletin` name is a later-client cross-version reference. Stone accepts action `0x31` in the listed functions. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x32` `SStateObjectState`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x32`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

### Stone evidence

`Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. The mapped branch performs inline tile or object updates.

### Mapping status

The `SStateObjectState` name is a later-client cross-version reference. Stone accepts action `0x32` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x33` `SDrawHumanObject`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x33`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

### Stone evidence

`Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`.

### Mapping status

The `SDrawHumanObject` name is a later-client cross-version reference. Stone accepts action `0x33` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x34` `SObjectInfo`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x34`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

### Stone evidence

`Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`.

### Mapping status

The `SObjectInfo` name is a later-client cross-version reference. Stone accepts action `0x34` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x35` `SShowPaper`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x35`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

### Stone evidence

`Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`.

### Mapping status

The `SShowPaper` name is a later-client cross-version reference. Stone accepts action `0x35` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x36` `SShowUsers`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x36`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

### Stone evidence

`Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`.

### Mapping status

The `SShowUsers` name is a later-client cross-version reference. Stone accepts action `0x36` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x37` `SAddEquip`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x37`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x0042EAF0` | `sub_42EAF0` | Accepts the action in its pane's Event type 9 packet handler. |

### Stone evidence

`Darkages.exe:0x0042EAF0` `sub_42EAF0`.

### Mapping status

The `SAddEquip` name is a later-client cross-version reference. Stone accepts action `0x37` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x38` `SRemoveEquip`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x38`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x0042EAF0` | `sub_42EAF0` | Accepts the action in its pane's Event type 9 packet handler. |

### Stone evidence

`Darkages.exe:0x0042EAF0` `sub_42EAF0`.

### Mapping status

The `SRemoveEquip` name is a later-client cross-version reference. Stone accepts action `0x38` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x39` `SSelfLook`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x39`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x0042EAF0` | `sub_42EAF0` | Accepts the action in its pane's Event type 9 packet handler. |

### Stone evidence

`Darkages.exe:0x0042EAF0` `sub_42EAF0`.

### Mapping status

The `SSelfLook` name is a later-client cross-version reference. Stone accepts action `0x39` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x3A` `SSpelled`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x3A`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x0043E8B0` | `sub_43E8B0` | Accepts the action in its pane's Event type 9 packet handler. |

### Stone evidence

`Darkages.exe:0x0043E8B0` `sub_43E8B0`.

### Mapping status

The `SSpelled` name is a later-client cross-version reference. Stone accepts action `0x3A` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x3B` `SRequestCRC`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x3B`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

### Stone evidence

`Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`.

### Mapping status

The `SRequestCRC` name is a later-client cross-version reference. Stone accepts action `0x3B` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x3C` `SMapPart`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x3C`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

### Stone evidence

`Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`.

### Mapping status

The `SMapPart` name is a later-client cross-version reference. Stone accepts action `0x3C` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x3D` `SLevelPoint`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x3D`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x0043F420` | `sub_43F420` | Accepts the action in its pane's Event type 9 packet handler. |

### Stone evidence

`Darkages.exe:0x0043F420` `sub_43F420`.

### Mapping status

The `SLevelPoint` name is a later-client cross-version reference. Stone accepts action `0x3D` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x3E` `SWindowChange`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x3E`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x0043D650` | `sub_43D650` | Accepts the action in its pane's Event type 9 packet handler. |

### Stone evidence

`Darkages.exe:0x0043D650` `sub_43D650`.

### Mapping status

The `SWindowChange` name is a later-client cross-version reference. Stone accepts action `0x3E` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x3F` `SActionCoolTime`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x3F`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00442990` | `sub_442990` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x00443B10` | `sub_443B10` | Accepts the action in its pane's Event type 9 packet handler. |

### Stone evidence

`Darkages.exe:0x00442990` `sub_442990`, `Darkages.exe:0x00443B10` `sub_443B10`.

### Mapping status

The `SActionCoolTime` name is a later-client cross-version reference. Stone accepts action `0x3F` in the listed functions. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x40` `SSendPatch`

**Direction:** server to client

**Encrypted:** No.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x40`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x004955C0` | `sub_4955C0` | Accepts the action in its pane's Event type 9 packet handler. |

### Stone evidence

`Darkages.exe:0x004955C0` `sub_4955C0`. XOR bypass. The later-client scope may be login or lobby.

### Mapping status

The `SSendPatch` name is a later-client cross-version reference. Stone accepts action `0x40` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x42` `SExchange`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x42`. |
| `0x01` | 1 | `subtype` | Exchange operation. Values 1 through 5 have mapped Stone branches. |
| `0x02` | `packet_size - 2` | `unknown_02` | Subtype-dependent payload and protocol sentinel; field boundaries remain to be mapped. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00435EA0` | `sub_435EA0` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x004374F0` | `sub_4374F0` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x00437C70` | `sub_437C70` | Accepts the action in its pane's Event type 9 packet handler. |
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |

### Stone evidence

`Darkages.exe:0x00435EA0` `sub_435EA0`, `Darkages.exe:0x004374F0` `sub_4374F0`, `Darkages.exe:0x00437C70` `sub_437C70`, `Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`. Exchange-session handlers dispatch on a second byte with established subtypes 1 through 5.

### Mapping status

The later-client `SExchange` name agrees with the Stone exchange-pane handlers. The subtype byte and branches 1 through 5 are established. Their individual payload schemas remain to be mapped.

## `0x48` `SSpellDelayCancel`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x48`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x004568B0` | `sub_4568B0` | Accepts the action in its pane's Event type 9 packet handler. |

### Stone evidence

`Darkages.exe:0x004568B0` `sub_4568B0`.

### Mapping status

The `SSpellDelayCancel` name is a later-client cross-version reference. Stone accepts action `0x48` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x49` `SRequestPortrait`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x49`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x0040B3A0` | `ui_handle_server_request_portrait` | Handles the portrait request and starts client portrait serialization. |

### Stone evidence

`Darkages.exe:0x0040B3A0` `ui_handle_server_request_portrait`. An embedded diagnostic names `kServerRequestPortrait` and states decimal value 73. The handler initiates client action `0x4F`.

### Mapping status

Both the later-client `SRequestPortrait` reference and the Stone diagnostic `kServerRequestPortrait` identify this action. The handler initiates client action `0x4F`; its request payload, if any beyond the sentinel, remains to be mapped.

## `0x4B` `SBounce`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x4B`. |
| `0x01` | 2 | `embedded_length` | Unsigned big-endian length of the embedded logical client packet. |
| `0x03` | `embedded_length` | `embedded_client_packet` | Begins with a client-direction action byte and is submitted through `net_c_queue_send`. |
| `0x03 + embedded_length` | variable | `unknown_tail` | Any remaining bytes, including the outer protocol sentinel; the exact trailing boundary is not yet mapped. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x00468A90` | `ui_map_dispatch_server_packet` | Accepts and dispatches the action in `MapPane`. |
| `Darkages.exe:0x0046CA54` | `net_forward_embedded_client_packet` | Reads the embedded length and submits the embedded client packet. |

### Stone evidence

`Darkages.exe:0x00468A90` `ui_map_dispatch_server_packet`, which calls `Darkages.exe:0x0046CA54` `net_forward_embedded_client_packet`. Offset `0x01` is a big-endian embedded length and offset `0x03` begins the logical client packet.

### Mapping status

The later-client `SBounce` name is a cross-version reference. Stone directly establishes the embedded length and forwarding behavior. The server can therefore select a client action dynamically rather than using one of the fixed action constants found in client builders.

## `0x4C` `SReconnect`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x4C`. |
| `0x01` | 1 | `subtype` | Value `0x01` completes the Stone safe-exit exchange. Other values are consumed without the mapped side effects. |
| `0x02` | `packet_size - 2` | `unknown_02` | Any trailing bytes and the protocol sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x004921F0` | `ui_exit_wait_handle_server_packet` | Consumes `0x4C`; subtype `0x01` completes the safe-exit exchange. |
| `Darkages.exe:0x00492310` | `ui_exit_wait_pane_ctor` | Creates the exit-wait pane, shows the warning, and sends `CQuit` subtype `0x01`. |

### Stone evidence

`Darkages.exe:0x004921F0` `ui_exit_wait_handle_server_packet`. The Stone handler consumes all `0x4C` packets. Subtype `0x01` sends `CQuit` subtype `0x00` and changes an exit-wait pane to the safe-exit message; it does not call the Socket reconnect path.

### Mapping status

The `SReconnect` name is retained as the supplied later-client reference, but it does not describe the mapped Stone behavior by itself. `ui_exit_wait_handle_server_packet` claims every `0x4C` packet. For subtype `0x01`, it sets the pane completion flag, sends `CQuit` subtype `0x00`, and replaces the warning with the safe-to-exit text. The paired `ui_exit_wait_pane_ctor` sends `CQuit` subtype `0x01` when the wait pane is created. No Socket connection or reconnect function is called by this handler.

## `0x51` `SBlockInput`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x51`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x004889F0` | `sub_4889F0` | Accepts the action in its pane's Event type 9 packet handler. |

### Stone evidence

`Darkages.exe:0x004889F0` `sub_4889F0`.

### Mapping status

The `SBlockInput` name is a later-client cross-version reference. Stone accepts action `0x51` in the listed function. Payload field division remains a placeholder until its readers and client-side effects are traced end to end.

## `0x56` `SMulti`

**Direction:** server to client

**Encrypted:** Yes.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x56`. |
| `0x01` | `packet_size - 2` | `unknown_01` | Decoded payload bytes whose field boundaries are not yet mapped. |
| `packet_size - 1` | 1 | `packet_sentinel` | Protocol zero sentinel. |

### Mapped Stone functions

| Function address | Current IDA name | Role |
|---:|---|---|
| `Darkages.exe:0x004A2ED0` | `ui_server_select_handle_server_packet` | Deserializes the server-selection table. |

### Stone evidence

`Darkages.exe:0x004A2ED0` `ui_server_select_handle_server_packet`. `ServerSelectDialogPane` reads a count, allocates replacement entry storage, and deserializes fixed-width integers and colon-delimited strings before loading the table.

### Mapping status

The later-client `SMulti` name is a cross-version reference. Stone independently establishes that this action replaces and deserializes the server-selection table. The exact packet fields will be documented after the reader helpers are traced end to end.
