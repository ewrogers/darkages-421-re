# Client Message Directory

This directory expands the compact [client action index](client-actions.md) into one section per fixed client-to-server action. It records the exact `net_c_queue_send` call sites and their containing IDA functions even when the function still has a neutral `sub_...` name.

The `C...` identifiers are later-client reference names. They are not RTTI recovered from Stone. Placeholder fields identify only established offsets and ownership boundaries; they do not assign unverified payload meanings.

## Common shape conventions

A packet builder submits `logical_length` bytes beginning with the action. `net_c_queue_send` copies those bytes and appends the zero sentinel. Ordinary packets then acquire a sequence byte and XOR transformation. Actions `0x00`, `0x10`, and `0x48` remain cleartext and do not consume the sequence.

```text
Transformed: [action] [sequence] [XOR(payload || 00)]
Cleartext:   [action] [payload] [00]
```

The placeholder name `unknown_01` means that bytes beginning at logical offset `0x01` are present or possible but have not yet been divided into verified fields. The appended sentinel is not part of the builder-supplied `logical_length`.

The per-message tables account for 118 direct fixed-action calls to `net_c_queue_send`. The remaining direct caller is `Darkages.exe:0x0046CA9C` in `net_forward_embedded_client_packet`. Server action `0x4B` supplies that packet's first byte dynamically, so the call cannot be assigned to one fixed client-message section. See [Server-directed dynamic send](client-actions.md#server-directed-dynamic-send).

## Directory

| Action | Reference name | Wire treatment | Direct queue calls |
|---:|---|---|---:|
| `0x00` | [`CVersion`](#0x00-cversion) | Cleartext, no sequence consumed | 1 |
| `0x02` | [`CNewUser`](#0x02-cnewuser) | XOR-transformed, sequence consumed | 1 |
| `0x03` | [`CLogin`](#0x03-clogin) | XOR-transformed, sequence consumed | 1 |
| `0x04` | [`CNewUserApperance`](#0x04-cnewuserapperance) | XOR-transformed, sequence consumed | 1 |
| `0x05` | [`CMapRequest`](#0x05-cmaprequest) | XOR-transformed, sequence consumed | 1 |
| `0x06` | [`CMove`](#0x06-cmove) | XOR-transformed, sequence consumed | 1 |
| `0x07` | [`CGet`](#0x07-cget) | XOR-transformed, sequence consumed | 1 |
| `0x08` | [`CDrop`](#0x08-cdrop) | XOR-transformed, sequence consumed | 1 |
| `0x0B` | [`CQuit`](#0x0b-cquit) | XOR-transformed, sequence consumed | 6 |
| `0x0C` | [`CPutGround`](#0x0c-cputground) | XOR-transformed, sequence consumed | 1 |
| `0x0D` | [`CBlockListen`](#0x0d-cblocklisten) | XOR-transformed, sequence consumed | 3 |
| `0x0E` | [`CSay`](#0x0e-csay) | XOR-transformed, sequence consumed | 4 |
| `0x10` | [`CTransferServer`](#0x10-ctransferserver) | Cleartext, no sequence consumed | 3 |
| `0x11` | [`CChangeDirection`](#0x11-cchangedirection) | XOR-transformed, sequence consumed | 2 |
| `0x13` | [`CAttack`](#0x13-cattack) | XOR-transformed, sequence consumed | 1 |
| `0x18` | [`CWho`](#0x18-cwho) | XOR-transformed, sequence consumed | 1 |
| `0x19` | [`CWhisper`](#0x19-cwhisper) | XOR-transformed, sequence consumed | 1 |
| `0x1B` | [`CUserSetting`](#0x1b-cusersetting) | XOR-transformed, sequence consumed | 1 |
| `0x1C` | [`CUse`](#0x1c-cuse) | XOR-transformed, sequence consumed | 1 |
| `0x1D` | [`CEmotion`](#0x1d-cemotion) | XOR-transformed, sequence consumed | 1 |
| `0x23` | [`CExitEditingMode`](#0x23-cexiteditingmode) | XOR-transformed, sequence consumed | 2 |
| `0x24` | [`CDropGold`](#0x24-cdropgold) | XOR-transformed, sequence consumed | 1 |
| `0x26` | [`CChangePassword`](#0x26-cchangepassword) | XOR-transformed, sequence consumed | 2 |
| `0x29` | [`CGive`](#0x29-cgive) | XOR-transformed, sequence consumed | 1 |
| `0x2A` | [`CGiveGold`](#0x2a-cgivegold) | XOR-transformed, sequence consumed | 1 |
| `0x2D` | [`CSelfLook`](#0x2d-cselflook) | XOR-transformed, sequence consumed | 1 |
| `0x2E` | [`CGroup`](#0x2e-cgroup) | XOR-transformed, sequence consumed | 1 |
| `0x2F` | [`CGroupToggle`](#0x2f-cgrouptoggle) | XOR-transformed, sequence consumed | 1 |
| `0x30` | [`CChangeSlot`](#0x30-cchangeslot) | XOR-transformed, sequence consumed | 4 |
| `0x38` | [`CRefreshUser`](#0x38-crefreshuser) | XOR-transformed, sequence consumed | 1 |
| `0x39` | [`CMenuCode`](#0x39-cmenucode) | XOR-transformed, sequence consumed | 9 |
| `0x3A` | [`CMessage`](#0x3a-cmessage) | XOR-transformed, sequence consumed | 11 |
| `0x3B` | [`CBulletin`](#0x3b-cbulletin) | XOR-transformed, sequence consumed | 15 |
| `0x3C` | [`CPutToContainer`](#0x3c-cputtocontainer) | XOR-transformed, sequence consumed | 1 |
| `0x3D` | [`CGetFromContainer`](#0x3d-cgetfromcontainer) | XOR-transformed, sequence consumed | 2 |
| `0x3E` | [`CUseSkill`](#0x3e-cuseskill) | XOR-transformed, sequence consumed | 1 |
| `0x3F` | [`CFieldMap`](#0x3f-cfieldmap) | XOR-transformed, sequence consumed | 1 |
| `0x41` | [`CGetParcel`](#0x41-cgetparcel) | XOR-transformed, sequence consumed | 1 |
| `0x42` | [`CException`](#0x42-cexception) | XOR-transformed, sequence consumed | 1 |
| `0x43` | [`CInteract`](#0x43-cinteract) | XOR-transformed, sequence consumed | 4 |
| `0x44` | [`CRemoveEquip`](#0x44-cremoveequip) | XOR-transformed, sequence consumed | 1 |
| `0x45` | [`CReplyCRC`](#0x45-creplycrc) | XOR-transformed, sequence consumed | 1 |
| `0x46` | [`CGroupView`](#0x46-cgroupview) | XOR-transformed, sequence consumed | 1 |
| `0x47` | [`CAddStat`](#0x47-caddstat) | XOR-transformed, sequence consumed | 1 |
| `0x48` | [`CRequestPatch`](#0x48-crequestpatch) | Cleartext, no sequence consumed | 4 |
| `0x4A` | [`CExchange`](#0x4a-cexchange) | XOR-transformed, sequence consumed | 7 |
| `0x4D` | [`CSpellDelayRequest`](#0x4d-cspelldelayrequest) | XOR-transformed, sequence consumed | 1 |
| `0x4E` | [`CSpellDelaySay`](#0x4e-cspelldelaysay) | XOR-transformed, sequence consumed | 4 |
| `0x4F` | [`CSendPortrait`](#0x4f-csendportrait) | XOR-transformed, sequence consumed | 1 |
| `0x57` | [`CMulti`](#0x57-cmulti) | XOR-transformed, sequence consumed | 2 |
| `0x62` | [`baram`](#0x62-baram) | XOR-transformed, sequence consumed | 1 |

## `0x00` `CVersion`

**Direction:** client to server

**Wire treatment:** Cleartext, no sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x00`. |
| `0x01` | 2 | `data_version` | Unsigned big-endian value selected by the lobby startup code. |
| `0x03` | 2 | `client_tag` | Literal ASCII `LK`. |
| `0x05` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not included in the builder's submitted length of 5. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x004AD6A3` | `sub_4AD140` | `Darkages.exe:0x004AD140` |

### Mapping status

The Stone builder establishes the complete five-byte submitted shape. The later-client name and lobby or login scope agree with the observed startup path.

## `0x02` `CNewUser`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x02`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x00462730` | `sub_4625C4` | `Darkages.exe:0x004625C4` |

### Mapping status

The `CNewUser` name is a later-client correlation. Stone emits action `0x02` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x03` `CLogin`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x03`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x00462D2F` | `sub_462BF4` | `Darkages.exe:0x00462BF4` |

### Mapping status

The `CLogin` name is a later-client correlation. Stone emits action `0x03` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x04` `CNewUserApperance`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x04`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x00461725` | `sub_4616A4` | `Darkages.exe:0x004616A4` |

### Mapping status

The `CNewUserApperance` name is a later-client correlation. Stone emits action `0x04` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x05` `CMapRequest`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x05`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x00465D7A` | `sub_465CC4` | `Darkages.exe:0x00465CC4` |

### Mapping status

The `CMapRequest` name is a later-client correlation. Stone emits action `0x05` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x06` `CMove`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x06`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x0048853A` | `sub_4884E4` | `Darkages.exe:0x004884E4` |

### Mapping status

The `CMove` name is a later-client correlation. Stone emits action `0x06` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x07` `CGet`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x07`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x00454213` | `sub_4541B4` | `Darkages.exe:0x004541B4` |

### Mapping status

The `CGet` name is a later-client correlation. Stone emits action `0x07` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x08` `CDrop`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x08`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x004530AE` | `sub_453054` | `Darkages.exe:0x00453054` |

### Mapping status

The `CDrop` name is a later-client correlation. Stone emits action `0x08` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x0B` `CQuit`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x0B`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x0046471C` | `sub_4646F0` | `Darkages.exe:0x004646F0` |
| `Darkages.exe:0x004889DC` | `sub_488974` | `Darkages.exe:0x00488974` |
| `Darkages.exe:0x00488E2E` | `sub_488B44` | `Darkages.exe:0x00488B44` |
| `Darkages.exe:0x004922B1` | `sub_4921F0` | `Darkages.exe:0x004921F0` |
| `Darkages.exe:0x0049236B` | `sub_492310` | `Darkages.exe:0x00492310` |
| `Darkages.exe:0x0049544D` | `sub_495414` | `Darkages.exe:0x00495414` |

### Mapping status

The `CQuit` name is a later-client correlation. Stone emits action `0x0B` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x0C` `CPutGround`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x0C`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x0046B1C2` | `sub_46B194` | `Darkages.exe:0x0046B194` |

### Mapping status

The `CPutGround` name is a later-client correlation. Stone emits action `0x0C` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x0D` `CBlockListen`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x0D`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x004C1691` | `sub_4C1654` | `Darkages.exe:0x004C1654` |
| `Darkages.exe:0x004C1956` | `sub_4C18A4` | `Darkages.exe:0x004C18A4` |
| `Darkages.exe:0x004C1A66` | `sub_4C19B4` | `Darkages.exe:0x004C19B4` |

### Mapping status

The `CBlockListen` name is a later-client correlation. Stone emits action `0x0D` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x0E` `CSay`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x0E`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x004811F1` | `sub_481150` | `Darkages.exe:0x00481150` |
| `Darkages.exe:0x00482DA9` | `sub_482B70` | `Darkages.exe:0x00482B70` |
| `Darkages.exe:0x0048865F` | `sub_4885D4` | `Darkages.exe:0x004885D4` |
| `Darkages.exe:0x004C0987` | `sub_4C0920` | `Darkages.exe:0x004C0920` |

### Mapping status

The `CSay` name is a later-client correlation. Stone emits action `0x0E` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x10` `CTransferServer`

**Direction:** client to server

**Wire treatment:** Cleartext, no sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x10`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x00460617` | `sub_460584` | `Darkages.exe:0x00460584` |
| `Darkages.exe:0x00463119` | `sub_463010` | `Darkages.exe:0x00463010` |
| `Darkages.exe:0x0046C901` | `sub_46C7D4` | `Darkages.exe:0x0046C7D4` |

### Mapping status

The payload fields remain placeholders. Stone diagnostics independently name `kClientTransferServer`, and the Socket transfer gate permits only this action while a server transfer is active.

## `0x11` `CChangeDirection`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x11`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x0046822F` | `sub_4681E4` | `Darkages.exe:0x004681E4` |
| `Darkages.exe:0x004884D4` | `sub_4884B4` | `Darkages.exe:0x004884B4` |

### Mapping status

The `CChangeDirection` name is a later-client correlation. Stone emits action `0x11` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x13` `CAttack`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x13`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x0048895C` | `sub_488944` | `Darkages.exe:0x00488944` |

### Mapping status

The `CAttack` name is a later-client correlation. Stone emits action `0x13` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x18` `CWho`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x18`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x0043E39C` | `sub_43E384` | `Darkages.exe:0x0043E384` |

### Mapping status

The `CWho` name is a later-client correlation. Stone emits action `0x18` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x19` `CWhisper`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x19`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x004C1182` | `sub_4C10D0` | `Darkages.exe:0x004C10D0` |

### Mapping status

The `CWhisper` name is a later-client correlation. Stone emits action `0x19` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x1B` `CUserSetting`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x1B`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x00491AC4` | `sub_491AA4` | `Darkages.exe:0x00491AA4` |

### Mapping status

The `CUserSetting` name is a later-client correlation. Stone emits action `0x1B` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x1C` `CUse`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x1C`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x004533C6` | `sub_453394` | `Darkages.exe:0x00453394` |

### Mapping status

The `CUse` name is a later-client correlation. Stone emits action `0x1C` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x1D` `CEmotion`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x1D`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x00488694` | `sub_488674` | `Darkages.exe:0x00488674` |

### Mapping status

The `CEmotion` name is a later-client correlation. Stone emits action `0x1D` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x23` `CExitEditingMode`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x23`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x00493C99` | `sub_493B60` | `Darkages.exe:0x00493B60` |
| `Darkages.exe:0x0049421C` | `sub_494100` | `Darkages.exe:0x00494100` |

### Mapping status

The `CExitEditingMode` name is a later-client correlation. Stone emits action `0x23` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x24` `CDropGold`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x24`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x004534E1` | `sub_453424` | `Darkages.exe:0x00453424` |

### Mapping status

The `CDropGold` name is a later-client correlation. Stone emits action `0x24` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x26` `CChangePassword`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x26`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x00463A45` | `sub_4638C4` | `Darkages.exe:0x004638C4` |
| `Darkages.exe:0x004642F0` | `sub_464174` | `Darkages.exe:0x00464174` |

### Mapping status

The `CChangePassword` name is a later-client correlation. Stone emits action `0x26` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x29` `CGive`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x29`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x0045336A` | `sub_453324` | `Darkages.exe:0x00453324` |

### Mapping status

The `CGive` name is a later-client correlation. Stone emits action `0x29` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x2A` `CGiveGold`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x2A`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x0045360B` | `sub_453564` | `Darkages.exe:0x00453564` |

### Mapping status

The `CGiveGold` name is a later-client correlation. Stone emits action `0x2A` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x2D` `CSelfLook`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x2D`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x0043D4DC` | `sub_43D4C4` | `Darkages.exe:0x0043D4C4` |

### Mapping status

The `CSelfLook` name is a later-client correlation. Stone emits action `0x2D` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x2E` `CGroup`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x2E`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x004306AE` | `sub_430644` | `Darkages.exe:0x00430644` |

### Mapping status

The `CGroup` name is a later-client correlation. Stone emits action `0x2E` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x2F` `CGroupToggle`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x2F`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x0042F1D0` | `sub_42F1A4` | `Darkages.exe:0x0042F1A4` |

### Mapping status

The `CGroupToggle` name is a later-client correlation. Stone emits action `0x2F` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x30` `CChangeSlot`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x30`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x0042499F` | `sub_424914` | `Darkages.exe:0x00424914` |
| `Darkages.exe:0x00441216` | `sub_441164` | `Darkages.exe:0x00441164` |
| `Darkages.exe:0x004427DF` | `sub_442684` | `Darkages.exe:0x00442684` |
| `Darkages.exe:0x0044395F` | `sub_443804` | `Darkages.exe:0x00443804` |

### Mapping status

The `CChangeSlot` name is a later-client correlation. Stone emits action `0x30` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x38` `CRefreshUser`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x38`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x00488110` | `sub_4880E4` | `Darkages.exe:0x004880E4` |

### Mapping status

The `CRefreshUser` name is a later-client correlation. Stone emits action `0x38` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x39` `CMenuCode`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x39`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x0047B8CD` | `sub_47B850` | `Darkages.exe:0x0047B850` |
| `Darkages.exe:0x0047BBB7` | `sub_47BB00` | `Darkages.exe:0x0047BB00` |
| `Darkages.exe:0x0047C175` | `sub_47C100` | `Darkages.exe:0x0047C100` |
| `Darkages.exe:0x0047C2CF` | `sub_47C220` | `Darkages.exe:0x0047C220` |
| `Darkages.exe:0x0047C60F` | `sub_47C560` | `Darkages.exe:0x0047C560` |
| `Darkages.exe:0x0047C8C8` | `sub_47C7C0` | `Darkages.exe:0x0047C7C0` |
| `Darkages.exe:0x0047D3BF` | `sub_47D310` | `Darkages.exe:0x0047D310` |
| `Darkages.exe:0x0047D46D` | `sub_47D3E0` | `Darkages.exe:0x0047D3E0` |
| `Darkages.exe:0x0047D7DC` | `sub_47D6E4` | `Darkages.exe:0x0047D6E4` |

### Mapping status

The `CMenuCode` name is a later-client correlation. Stone emits action `0x39` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x3A` `CMessage`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x3A`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x0047E85C` | `sub_47E7D0` | `Darkages.exe:0x0047E7D0` |
| `Darkages.exe:0x0047F9AC` | `sub_47F920` | `Darkages.exe:0x0047F920` |
| `Darkages.exe:0x0047FA89` | `sub_47F9C0` | `Darkages.exe:0x0047F9C0` |
| `Darkages.exe:0x004805CC` | `sub_480540` | `Darkages.exe:0x00480540` |
| `Darkages.exe:0x004806A9` | `sub_4805E0` | `Darkages.exe:0x004805E0` |
| `Darkages.exe:0x0048113C` | `sub_4810B0` | `Darkages.exe:0x004810B0` |
| `Darkages.exe:0x004812AE` | `sub_481150` | `Darkages.exe:0x00481150` |
| `Darkages.exe:0x00481EEC` | `sub_481E60` | `Darkages.exe:0x00481E60` |
| `Darkages.exe:0x0048205B` | `sub_481F00` | `Darkages.exe:0x00481F00` |
| `Darkages.exe:0x00482B5C` | `sub_482AD0` | `Darkages.exe:0x00482AD0` |
| `Darkages.exe:0x00482F1C` | `sub_482B70` | `Darkages.exe:0x00482B70` |

### Mapping status

The `CMessage` name is a later-client correlation. Stone emits action `0x3A` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x3B` `CBulletin`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x3B`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x0040ECE1` | `sub_40ECA4` | `Darkages.exe:0x0040ECA4` |
| `Darkages.exe:0x004109EC` | `sub_410964` | `Darkages.exe:0x00410964` |
| `Darkages.exe:0x0041294C` | `sub_4128C4` | `Darkages.exe:0x004128C4` |
| `Darkages.exe:0x004145B4` | `sub_414534` | `Darkages.exe:0x00414534` |
| `Darkages.exe:0x00415738` | `sub_4156B4` | `Darkages.exe:0x004156B4` |
| `Darkages.exe:0x00415934` | `sub_4158C4` | `Darkages.exe:0x004158C4` |
| `Darkages.exe:0x0041715C` | `sub_4170D4` | `Darkages.exe:0x004170D4` |
| `Darkages.exe:0x0041817E` | `sub_417F94` | `Darkages.exe:0x00417F94` |
| `Darkages.exe:0x004187E8` | `sub_418764` | `Darkages.exe:0x00418764` |
| `Darkages.exe:0x0041A73C` | `sub_41A6B4` | `Darkages.exe:0x0041A6B4` |
| `Darkages.exe:0x0041AE45` | `sub_41AB84` | `Darkages.exe:0x0041AB84` |
| `Darkages.exe:0x0041B4E4` | `sub_41B474` | `Darkages.exe:0x0041B474` |
| `Darkages.exe:0x0041B60B` | `sub_41B5A4` | `Darkages.exe:0x0041B5A4` |
| `Darkages.exe:0x0041B8A8` | `sub_41B824` | `Darkages.exe:0x0041B824` |
| `Darkages.exe:0x0041BA1B` | `sub_41B9B4` | `Darkages.exe:0x0041B9B4` |

### Mapping status

The `CBulletin` name is a later-client correlation. Stone emits action `0x3B` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x3C` `CPutToContainer`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x3C`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x00424B24` | `sub_424A74` | `Darkages.exe:0x00424A74` |

### Mapping status

The `CPutToContainer` name is a later-client correlation. Stone emits action `0x3C` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x3D` `CGetFromContainer`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x3D`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x004412D8` | `sub_441234` | `Darkages.exe:0x00441234` |
| `Darkages.exe:0x00453A15` | `sub_4539B4` | `Darkages.exe:0x004539B4` |

### Mapping status

The `CGetFromContainer` name is a later-client correlation. Stone emits action `0x3D` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x3E` `CUseSkill`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x3E`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x00454C96` | `sub_454C64` | `Darkages.exe:0x00454C64` |

### Mapping status

The `CUseSkill` name is a later-client correlation. Stone emits action `0x3E` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x3F` `CFieldMap`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x3F`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x0043953A` | `sub_4394B4` | `Darkages.exe:0x004394B4` |

### Mapping status

The `CFieldMap` name is a later-client correlation. Stone emits action `0x3F` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x41` `CGetParcel`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x41`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x00494E2C` | `sub_494E14` | `Darkages.exe:0x00494E14` |

### Mapping status

The `CGetParcel` name is a later-client correlation. Stone emits action `0x41` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x42` `CException`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x42`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x00434B43` | `sub_434A40` | `Darkages.exe:0x00434A40` |

### Mapping status

The `CException` name is a later-client correlation. Stone emits action `0x42` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x43` `CInteract`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x43`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x0047B755` | `sub_47B704` | `Darkages.exe:0x0047B704` |
| `Darkages.exe:0x00488723` | `sub_4886A0` | `Darkages.exe:0x004886A0` |
| `Darkages.exe:0x0048C820` | `sub_48C774` | `Darkages.exe:0x0048C774` |
| `Darkages.exe:0x0048F17F` | `sub_48F114` | `Darkages.exe:0x0048F114` |

### Mapping status

The `CInteract` name is a later-client correlation. Stone emits action `0x43` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x44` `CRemoveEquip`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x44`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x0042F8E2` | `sub_42F8A4` | `Darkages.exe:0x0042F8A4` |

### Mapping status

The `CRemoveEquip` name is a later-client correlation. Stone emits action `0x44` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x45` `CReplyCRC`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x45`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x00465522` | `sub_4654E4` | `Darkages.exe:0x004654E4` |

### Mapping status

The `CReplyCRC` name is a later-client correlation. Stone emits action `0x45` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x46` `CGroupView`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x46`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x0042F919` | `sub_42F8F0` | `Darkages.exe:0x0042F8F0` |

### Mapping status

The `CGroupView` name is a later-client correlation. Stone emits action `0x46` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x47` `CAddStat`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x47`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x0044028E` | `sub_440234` | `Darkages.exe:0x00440234` |

### Mapping status

The `CAddStat` name is a later-client correlation. Stone emits action `0x47` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x48` `CRequestPatch`

**Direction:** client to server

**Wire treatment:** Cleartext, no sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x48`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x0048763C` | `sub_487604` | `Darkages.exe:0x00487604` |
| `Darkages.exe:0x004953EC` | `sub_495380` | `Darkages.exe:0x00495380` |
| `Darkages.exe:0x00495F55` | `sub_495E80` | `Darkages.exe:0x00495E80` |
| `Darkages.exe:0x0049609D` | `sub_496054` | `Darkages.exe:0x00496054` |

### Mapping status

The `CRequestPatch` name is a later-client correlation. Stone emits action `0x48` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x4A` `CExchange`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x4A`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x0043071A` | `sub_4306C4` | `Darkages.exe:0x004306C4` |
| `Darkages.exe:0x0043596E` | `sub_435904` | `Darkages.exe:0x00435904` |
| `Darkages.exe:0x00435A5A` | `sub_435A04` | `Darkages.exe:0x00435A04` |
| `Darkages.exe:0x00435ACA` | `sub_435A74` | `Darkages.exe:0x00435A74` |
| `Darkages.exe:0x00435E90` | `sub_435E24` | `Darkages.exe:0x00435E24` |
| `Darkages.exe:0x00437310` | `sub_4372A4` | `Darkages.exe:0x004372A4` |
| `Darkages.exe:0x00437AE8` | `sub_437A64` | `Darkages.exe:0x00437A64` |

### Mapping status

The `CExchange` name is a later-client correlation. Stone emits action `0x4A` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x4D` `CSpellDelayRequest`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x4D`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x00455CDF` | `sub_455CA4` | `Darkages.exe:0x00455CA4` |

### Mapping status

The `CSpellDelayRequest` name is a later-client correlation. Stone emits action `0x4D` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x4E` `CSpellDelaySay`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x4E`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x004548F1` | `sub_454884` | `Darkages.exe:0x00454884` |
| `Darkages.exe:0x00455921` | `sub_455834` | `Darkages.exe:0x00455834` |
| `Darkages.exe:0x00455815` | `sub_4556F4` | `Darkages.exe:0x004556F4` |
| `Darkages.exe:0x00456997` | `sub_456940` | `Darkages.exe:0x00456940` |

### Mapping status

The `CSpellDelaySay` name is a later-client correlation. Stone emits action `0x4E` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x4F` `CSendPortrait`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x4F`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x0040A7E3` | `net_c_build_portrait_response` | `Darkages.exe:0x0040A664` |

### Mapping status

The action purpose is independently established: `net_c_build_portrait_response` validates a local portrait and sends it after server action `0x49`. Individual payload fields remain to be mapped.

## `0x57` `CMulti`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | Constant `0x57`. |
| `0x01` | `logical_length - 1` | `unknown_01` | Zero or more builder-supplied payload bytes; field boundaries are not yet mapped. |
| `logical_length` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not part of the builder-supplied length. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x004A29D7` | `sub_4A2994` | `Darkages.exe:0x004A2994` |
| `Darkages.exe:0x004A2EB0` | `sub_4A2E54` | `Darkages.exe:0x004A2E54` |

### Mapping status

The `CMulti` name is a later-client correlation. Stone emits action `0x57` at the listed sites, but the payload fields and client-side trigger still require current-version tracing.

## `0x62` `baram`

**Direction:** client to server

**Wire treatment:** XOR-transformed, sequence consumed.

### Logical shape

| Offset | Width | Field | Established meaning |
|---:|---:|---|---|
| `0x00` | 1 | `action` | ASCII `b`, numeric action `0x62`. |
| `0x01` | 4 | `bootstrap_text` | Literal ASCII `aram`, completing `baram`. |
| `0x05` | 1 | `queue_sentinel` | Zero appended by `net_c_queue_send`; not included in the builder's submitted length of 5. |

### Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x004AD50B` | `sub_4AD140` | `Darkages.exe:0x004AD140` |

### Mapping status

The complete five-byte submitted value is established. On initial process startup it carries sequence `0x00` and advances `net_c_send_sequence` to `0x01`. Any reset or synchronization effect in the server is not visible in this client.
