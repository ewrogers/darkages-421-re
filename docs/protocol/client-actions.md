# Client Actions

This index records every fixed action byte placed in a logical client packet before `net_c_queue_send`. The send-site addresses are the calls that submit the completed byte array.

Each class name in the index links to a dedicated page containing its payload format, queue call sites, and current schema status. Message-page offsets are payload-relative and exclude framing, action, sequence, and the trailing zero.

The `C...` names identify 52 fixed client-to-server actions found in the traced 4.21 send path. Fifty-one appear directly at `net_c_queue_send` call sites. `CUseSpell` action `0x0F` is additionally established through the spell-delay controller, which stores the already-built logical packet and submits it immediately or on a later timer callback. The spelling `CNewUserApperance` is the 4.21 class name.

All entries use the client-to-server direction. The send queue appends a zero sentinel. Except for actions `0x00`, `0x48`, and `0x10`, `net_c_encode_packet_body` also inserts a sequence byte, applies the XOR transformation, and increments the send sequence before framing. The three cleartext bypass actions neither carry nor consume a sequence value.

## Fixed action index

| Action | Class name | Encrypted | 4.21 send sites and notes |
|---:|---|---|---|
| `0x00` | [`CVersion`](client/000-00-version.md) | No | `Darkages.exe:0x004AD6A3`. The four-byte payload is a big-endian data-version `uint16_t` followed by ASCII `LK`. XOR bypass. Used during login or lobby. |
| `0x02` | [`CNewUser`](client/002-02-new-user.md) | Yes | `Darkages.exe:0x00462730`. Used during login or lobby; the payload semantics are not yet established. |
| `0x03` | [`CLogin`](client/003-03-login.md) | Yes | `Darkages.exe:0x00462D2F`. Used during login or lobby; the payload semantics are not yet established. |
| `0x04` | [`CNewUserApperance`](client/004-04-new-user-apperance.md) | Yes | `Darkages.exe:0x00461725`. Used during login or lobby; the payload semantics are not yet established. |
| `0x05` | [`CMapRequest`](client/005-05-map-request.md) | Yes | `Darkages.exe:0x00465D7A`. Payload semantics are not yet established. |
| `0x06` | [`CMove`](client/006-06-move.md) | Yes | `Darkages.exe:0x0048853A`. Payload semantics are not yet established. |
| `0x07` | [`CGet`](client/007-07-get.md) | Yes | `Darkages.exe:0x00454213`. Payload semantics are not yet established. |
| `0x08` | [`CDrop`](client/008-08-drop.md) | Yes | `Darkages.exe:0x004530AE`. Payload semantics are not yet established. |
| `0x0B` | [`CQuit`](client/011-0b-quit.md) | Yes | `Darkages.exe:0x0046471C`, `Darkages.exe:0x004889DC`, `Darkages.exe:0x00488E2E`, `Darkages.exe:0x004922B1`, `Darkages.exe:0x0049236B`, `Darkages.exe:0x0049544D`. |
| `0x0C` | [`CPutGround`](client/012-0c-put-ground.md) | Yes | `Darkages.exe:0x0046B1C2`. Payload semantics are not yet established. |
| `0x0D` | [`CBlockListen`](client/013-0d-block-listen.md) | Yes | `Darkages.exe:0x004C1691`, `Darkages.exe:0x004C1956`, `Darkages.exe:0x004C1A66`. |
| `0x0E` | [`CSay`](client/014-0e-say.md) | Yes | `Darkages.exe:0x004811F1`, `Darkages.exe:0x00482DA9`, `Darkages.exe:0x0048865F`, `Darkages.exe:0x004C0987`. |
| `0x0F` | [`CUseSpell`](client/015-0f-use-spell.md) | Yes | `ui_spell_slot_begin_cast` builds the confirmed two-byte immediate form. `ui_spell_delay_begin` queues it directly or `ui_spell_delay_handle_timer` queues the held packet after delay traffic. |
| `0x10` | [`CTransferServer`](client/016-10-transfer-server.md) | No | `Darkages.exe:0x00460617`, `Darkages.exe:0x00463119`, `Darkages.exe:0x0046C901`. A `MapPane::ProcessTransferServer` diagnostic names `kClientTransferServer`; this is the only action allowed while the Socket transfer gate is active. XOR bypass. |
| `0x11` | [`CChangeDirection`](client/017-11-change-direction.md) | Yes | `Darkages.exe:0x0046822F`, `Darkages.exe:0x004884D4`. |
| `0x13` | [`CAttack`](client/019-13-attack.md) | Yes | `Darkages.exe:0x0048895C`. Payload semantics are not yet established. |
| `0x18` | [`CWho`](client/024-18-who.md) | Yes | `net_c_send_who_request` submits one action byte. The E window button requests the list; SShowUsers populates and shows the pane. |
| `0x19` | [`CWhisper`](client/025-19-whisper.md) | Yes | `Darkages.exe:0x004C1182`. Payload semantics are not yet established. |
| `0x1B` | [`CUserSetting`](client/027-1b-user-setting.md) | Yes | `net_c_send_user_setting` submits one setting-code byte from OptionPane-family controls. |
| `0x1C` | [`CUse`](client/028-1c-use.md) | Yes | `Darkages.exe:0x004533C6`. Payload semantics are not yet established. |
| `0x1D` | [`CEmotion`](client/029-1d-emotion.md) | Yes | `Darkages.exe:0x00488694`. Payload semantics are not yet established. |
| `0x23` | [`CExitEditingMode`](client/035-23-exit-editing-mode.md) | Yes | `ui_paper_dialog_handle_completion` and `net_c_send_exit_editing_mode` serialize Paper mode, text length, and normalized text. |
| `0x24` | [`CDropGold`](client/036-24-drop-gold.md) | Yes | `Darkages.exe:0x004534E1`. Payload semantics are not yet established. |
| `0x26` | [`CChangePassword`](client/038-26-change-password.md) | Yes | `Darkages.exe:0x00463A45`, `Darkages.exe:0x004642F0`. Used during login or lobby. |
| `0x29` | [`CGive`](client/041-29-give.md) | Yes | `Darkages.exe:0x0045336A`. Payload semantics are not yet established. |
| `0x2A` | [`CGiveGold`](client/042-2a-give-gold.md) | Yes | `Darkages.exe:0x0045360B`. Payload semantics are not yet established. |
| `0x2D` | [`CSelfLook`](client/045-2d-self-look.md) | Yes | `net_c_send_self_look` submits one action byte when a hidden current equipment pane is reopened. |
| `0x2E` | [`CGroup`](client/046-2e-group.md) | Yes | `Darkages.exe:0x004306AE`. Payload semantics are not yet established. |
| `0x2F` | [`CGroupToggle`](client/047-2f-group-toggle.md) | Yes | `Darkages.exe:0x0042F1D0`. Payload semantics are not yet established. |
| `0x30` | [`CChangeSlot`](client/048-30-change-slot.md) | Yes | `Darkages.exe:0x0042499F`, `Darkages.exe:0x00441216`, `Darkages.exe:0x004427DF`, `Darkages.exe:0x0044395F`. |
| `0x38` | [`CRefreshUser`](client/056-38-refresh-user.md) | Yes | `Darkages.exe:0x00488110`. Payload semantics are not yet established. |
| `0x39` | [`CMenuCode`](client/057-39-menu-code.md) | Yes | `Darkages.exe:0x0047B8CD`, `Darkages.exe:0x0047BBB7`, `Darkages.exe:0x0047C175`, `Darkages.exe:0x0047C2CF`, `Darkages.exe:0x0047C60F`, `Darkages.exe:0x0047C8C8`, `Darkages.exe:0x0047D3BF`, `Darkages.exe:0x0047D46D`, `Darkages.exe:0x0047D7DC`. |
| `0x3A` | [`CMessage`](client/058-3a-message.md) | Yes | `Darkages.exe:0x0047E85C`, `Darkages.exe:0x0047F9AC`, `Darkages.exe:0x0047FA89`, `Darkages.exe:0x004805CC`, `Darkages.exe:0x004806A9`, `Darkages.exe:0x0048113C`, `Darkages.exe:0x004812AE`, `Darkages.exe:0x00481EEC`, `Darkages.exe:0x0048205B`, `Darkages.exe:0x00482B5C`, `Darkages.exe:0x00482F1C`. |
| `0x3B` | [`CBulletin`](client/059-3b-bulletin.md) | Yes | `Darkages.exe:0x0040ECE1`, `Darkages.exe:0x004109EC`, `Darkages.exe:0x0041294C`, `Darkages.exe:0x004145B4`, `Darkages.exe:0x00415738`, `Darkages.exe:0x00415934`, `Darkages.exe:0x0041715C`, `Darkages.exe:0x0041817E`, `Darkages.exe:0x004187E8`, `Darkages.exe:0x0041A73C`, `Darkages.exe:0x0041AE45`, `Darkages.exe:0x0041B4E4`, `Darkages.exe:0x0041B60B`, `Darkages.exe:0x0041B8A8`, `Darkages.exe:0x0041BA1B`. |
| `0x3C` | [`CPutToContainer`](client/060-3c-put-to-container.md) | Yes | `Darkages.exe:0x00424B24`. Payload semantics are not yet established. |
| `0x3D` | [`CGetFromContainer`](client/061-3d-get-from-container.md) | Yes | `Darkages.exe:0x004412D8`, `Darkages.exe:0x00453A15`. |
| `0x3E` | [`CUseSkill`](client/062-3e-use-skill.md) | Yes | `net_c_send_use_skill` submits `[0x3E, skill_slot]`. |
| `0x3F` | [`CFieldMap`](client/063-3f-field-map.md) | Yes | `Darkages.exe:0x0043953A`. Payload semantics are not yet established. |
| `0x41` | [`CGetParcel`](client/065-41-get-parcel.md) | Yes | `Darkages.exe:0x00494E2C`. Payload semantics are not yet established. |
| `0x42` | [`CException`](client/066-42-exception.md) | Yes | `Darkages.exe:0x00434B43`. Payload semantics are not yet established. |
| `0x43` | [`CRequestObjectInfo`](client/067-43-request-object-info.md) | Yes | `Darkages.exe:0x0047B755`, `Darkages.exe:0x00488723`, `Darkages.exe:0x0048C820`, `Darkages.exe:0x0048F17F`. |
| `0x44` | [`CRemoveEquip`](client/068-44-remove-equip.md) | Yes | `Darkages.exe:0x0042F8E2`. Payload semantics are not yet established. |
| `0x45` | [`CReplyCRC`](client/069-45-reply-crc.md) | Yes | `Darkages.exe:0x00465522`. Payload semantics are not yet established. |
| `0x46` | [`CGroupView`](client/070-46-group-view.md) | Yes | `Darkages.exe:0x0042F919`. Payload semantics are not yet established. |
| `0x47` | [`CAddStat`](client/071-47-add-stat.md) | Yes | `Darkages.exe:0x0044028E`. Payload semantics are not yet established. |
| `0x48` | [`CRequestPatch`](client/072-48-request-patch.md) | No | `Darkages.exe:0x0048763C`, `Darkages.exe:0x004953EC`, `Darkages.exe:0x00495F55`, `Darkages.exe:0x0049609D`. XOR bypass. May be used during login or lobby. |
| `0x4A` | [`CExchange`](client/074-4a-exchange.md) | Yes | `Darkages.exe:0x0043071A`, `Darkages.exe:0x0043596E`, `Darkages.exe:0x00435A5A`, `Darkages.exe:0x00435ACA`, `Darkages.exe:0x00435E90`, `Darkages.exe:0x00437310`, `Darkages.exe:0x00437AE8`. |
| `0x4D` | [`CSpellDelayRequest`](client/077-4d-spell-delay-request.md) | Yes | `net_c_send_spell_delay_request` submits the configured delay-seconds byte. |
| `0x4E` | [`CSpellDelaySay`](client/078-4e-spell-delay-say.md) | Yes | Skill and spell paths submit a one-byte phrase length followed by phrase bytes; the delayed spell path can repeat it on 1000 ms timers. |
| `0x4F` | [`CSendPortrait`](client/079-4f-send-portrait.md) | Yes | `Darkages.exe:0x0040A7E3`. `net_c_build_portrait_response` reads and validates the local portrait and includes its bytes in the response to server action `0x49`. |
| `0x57` | [`CMulti`](client/087-57-multi.md) | Yes | `Darkages.exe:0x004A29D7`, `Darkages.exe:0x004A2EB0`. |
| `0x62` | [literal `baram`](client/098-62-baram.md) | Yes | `Darkages.exe:0x004AD50B`. The five submitted logical bytes are the literal ASCII string `baram`, so byte zero is action `0x62`. This special packet is named by its literal bytes rather than the `C...` convention. |

## `baram` bootstrap packet

`Darkages.exe:0x004AD4E4` formats the literal ASCII string `baram` and submits exactly five bytes at `Darkages.exe:0x004AD50B`. The same control-flow branch immediately builds and queues `CVersion` at `Darkages.exe:0x004AD6A3`.

Action `0x62` is not one of the client XOR bypass values, so `net_c_encode_packet_body` inserts the current sequence byte and increments `net_c_send_sequence` when transmitting it. The global occupies uninitialized image storage and is therefore zero-filled when the process loads. On the initial startup path, `baram` carries sequence `0x00` and advances the client counter to `0x01`; `CVersion` then bypasses the sequence machinery. The only code references to the counter are the read and increment inside `net_c_encode_packet_body`, so repeating this path does not explicitly reset the client counter. The packet may reset or synchronize server-side tracking, but that effect is not visible in this client code.

## Server-directed dynamic send

Server action `0x4B` is handled by `Darkages.exe:0x0046CA54` `net_forward_embedded_client_packet`. It reads a big-endian `uint16_t` from server-packet offset `0x01`, treats bytes beginning at offset `0x03` as a logical client packet of that length, and submits them to `net_c_queue_send`. The embedded first byte therefore selects the client action dynamically.

This path still receives the normal zero sentinel, direction-specific transformation, frame header, and transfer-state gate. It does not bypass the Socket send machinery.
