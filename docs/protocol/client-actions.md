# Client Actions

This index records every fixed action byte placed in a logical client packet before `net_c_queue_send`. The send-site addresses are the calls that submit the completed byte array.

The `C...` names are direction-prefixed reference names supplied from later-client dumps. They are not RTTI or symbols recovered from the Stone executable, and a matching action value does not by itself establish that the later payload schema is identical. Each row separately records what is established in the 4.21 code. The supplied later-client list matches all 51 fixed action values found in Stone, with no additional fixed client action found. The spelling `CNewUserApperance` is preserved from the supplied name.

All entries use the client-to-server direction. The send queue appends a zero sentinel. Except for actions `0x00`, `0x48`, and `0x10`, `net_c_encode_packet_body` also inserts a sequence byte, applies the XOR transformation, and increments the send sequence before framing. The three cleartext bypass actions neither carry nor consume a sequence value.

## Fixed action index

| Action | Later-client reference name | Stone send sites and established notes |
|---:|---|---|
| `0x00` | `CVersion` | `Darkages.exe:0x004AD6A3`. Logical fields are action, a big-endian data-version `uint16_t`, and ASCII `LK`; the builder submits 5 bytes before the send queue adds the sentinel. XOR bypass. The later-client scope is lobby or login. |
| `0x02` | `CNewUser` | `Darkages.exe:0x00462730`. The later-client scope is lobby or login; the Stone payload semantics are not yet independently established. |
| `0x03` | `CLogin` | `Darkages.exe:0x00462D2F`. The later-client scope is lobby or login; the Stone payload semantics are not yet independently established. |
| `0x04` | `CNewUserApperance` | `Darkages.exe:0x00461725`. The later-client scope is lobby or login; the Stone payload semantics are not yet independently established. |
| `0x05` | `CMapRequest` | `Darkages.exe:0x00465D7A`. Stone payload semantics not yet independently established. |
| `0x06` | `CMove` | `Darkages.exe:0x0048853A`. Stone payload semantics not yet independently established. |
| `0x07` | `CGet` | `Darkages.exe:0x00454213`. Stone payload semantics not yet independently established. |
| `0x08` | `CDrop` | `Darkages.exe:0x004530AE`. Stone payload semantics not yet independently established. |
| `0x0B` | `CQuit` | `Darkages.exe:0x0046471C`, `Darkages.exe:0x004889DC`, `Darkages.exe:0x00488E2E`, `Darkages.exe:0x004922B1`, `Darkages.exe:0x0049236B`, `Darkages.exe:0x0049544D`. |
| `0x0C` | `CPutGround` | `Darkages.exe:0x0046B1C2`. Stone payload semantics not yet independently established. |
| `0x0D` | `CBlockListen` | `Darkages.exe:0x004C1691`, `Darkages.exe:0x004C1956`, `Darkages.exe:0x004C1A66`. |
| `0x0E` | `CSay` | `Darkages.exe:0x004811F1`, `Darkages.exe:0x00482DA9`, `Darkages.exe:0x0048865F`, `Darkages.exe:0x004C0987`. |
| `0x10` | `CTransferServer` | `Darkages.exe:0x00460617`, `Darkages.exe:0x00463119`, `Darkages.exe:0x0046C901`. A `MapPane::ProcessTransferServer` diagnostic names `kClientTransferServer`; this is the only action allowed while the Socket transfer gate is active. XOR bypass. |
| `0x11` | `CChangeDirection` | `Darkages.exe:0x0046822F`, `Darkages.exe:0x004884D4`. |
| `0x13` | `CAttack` | `Darkages.exe:0x0048895C`. Stone payload semantics not yet independently established. |
| `0x18` | `CWho` | `Darkages.exe:0x0043E39C`. Stone payload semantics not yet independently established. |
| `0x19` | `CWhisper` | `Darkages.exe:0x004C1182`. Stone payload semantics not yet independently established. |
| `0x1B` | `CUserSetting` | `Darkages.exe:0x00491AC4`. Stone payload semantics not yet independently established. |
| `0x1C` | `CUse` | `Darkages.exe:0x004533C6`. Stone payload semantics not yet independently established. |
| `0x1D` | `CEmotion` | `Darkages.exe:0x00488694`. Stone payload semantics not yet independently established. |
| `0x23` | `CExitEditingMode` | `Darkages.exe:0x00493C99`, `Darkages.exe:0x0049421C`. The later-client context is paper editing. |
| `0x24` | `CDropGold` | `Darkages.exe:0x004534E1`. Stone payload semantics not yet independently established. |
| `0x26` | `CChangePassword` | `Darkages.exe:0x00463A45`, `Darkages.exe:0x004642F0`. The later-client scope is lobby or login. |
| `0x29` | `CGive` | `Darkages.exe:0x0045336A`. Stone payload semantics not yet independently established. |
| `0x2A` | `CGiveGold` | `Darkages.exe:0x0045360B`. Stone payload semantics not yet independently established. |
| `0x2D` | `CSelfLook` | `Darkages.exe:0x0043D4DC`. Stone payload semantics not yet independently established. |
| `0x2E` | `CGroup` | `Darkages.exe:0x004306AE`. Stone payload semantics not yet independently established. |
| `0x2F` | `CGroupToggle` | `Darkages.exe:0x0042F1D0`. Stone payload semantics not yet independently established. |
| `0x30` | `CChangeSlot` | `Darkages.exe:0x0042499F`, `Darkages.exe:0x00441216`, `Darkages.exe:0x004427DF`, `Darkages.exe:0x0044395F`. |
| `0x38` | `CRefreshUser` | `Darkages.exe:0x00488110`. Stone payload semantics not yet independently established. |
| `0x39` | `CMenuCode` | `Darkages.exe:0x0047B8CD`, `Darkages.exe:0x0047BBB7`, `Darkages.exe:0x0047C175`, `Darkages.exe:0x0047C2CF`, `Darkages.exe:0x0047C60F`, `Darkages.exe:0x0047C8C8`, `Darkages.exe:0x0047D3BF`, `Darkages.exe:0x0047D46D`, `Darkages.exe:0x0047D7DC`. |
| `0x3A` | `CMessage` | `Darkages.exe:0x0047E85C`, `Darkages.exe:0x0047F9AC`, `Darkages.exe:0x0047FA89`, `Darkages.exe:0x004805CC`, `Darkages.exe:0x004806A9`, `Darkages.exe:0x0048113C`, `Darkages.exe:0x004812AE`, `Darkages.exe:0x00481EEC`, `Darkages.exe:0x0048205B`, `Darkages.exe:0x00482B5C`, `Darkages.exe:0x00482F1C`. |
| `0x3B` | `CBulletin` | `Darkages.exe:0x0040ECE1`, `Darkages.exe:0x004109EC`, `Darkages.exe:0x0041294C`, `Darkages.exe:0x004145B4`, `Darkages.exe:0x00415738`, `Darkages.exe:0x00415934`, `Darkages.exe:0x0041715C`, `Darkages.exe:0x0041817E`, `Darkages.exe:0x004187E8`, `Darkages.exe:0x0041A73C`, `Darkages.exe:0x0041AE45`, `Darkages.exe:0x0041B4E4`, `Darkages.exe:0x0041B60B`, `Darkages.exe:0x0041B8A8`, `Darkages.exe:0x0041BA1B`. |
| `0x3C` | `CPutToContainer` | `Darkages.exe:0x00424B24`. Stone payload semantics not yet independently established. |
| `0x3D` | `CGetFromContainer` | `Darkages.exe:0x004412D8`, `Darkages.exe:0x00453A15`. |
| `0x3E` | `CUseSkill` | `Darkages.exe:0x00454C96`. Stone payload semantics not yet independently established. |
| `0x3F` | `CFieldMap` | `Darkages.exe:0x0043953A`. Stone payload semantics not yet independently established. |
| `0x41` | `CGetParcel` | `Darkages.exe:0x00494E2C`. Stone payload semantics not yet independently established. |
| `0x42` | `CException` | `Darkages.exe:0x00434B43`. Stone payload semantics not yet independently established. |
| `0x43` | `CInteract` | `Darkages.exe:0x0047B755`, `Darkages.exe:0x00488723`, `Darkages.exe:0x0048C820`, `Darkages.exe:0x0048F17F`. |
| `0x44` | `CRemoveEquip` | `Darkages.exe:0x0042F8E2`. Stone payload semantics not yet independently established. |
| `0x45` | `CReplyCRC` | `Darkages.exe:0x00465522`. Stone payload semantics not yet independently established. |
| `0x46` | `CGroupView` | `Darkages.exe:0x0042F919`. Stone payload semantics not yet independently established. |
| `0x47` | `CAddStat` | `Darkages.exe:0x0044028E`. Stone payload semantics not yet independently established. |
| `0x48` | `CRequestPatch` | `Darkages.exe:0x0048763C`, `Darkages.exe:0x004953EC`, `Darkages.exe:0x00495F55`, `Darkages.exe:0x0049609D`. XOR bypass. The later-client scope may be lobby or login. |
| `0x4A` | `CExchange` | `Darkages.exe:0x0043071A`, `Darkages.exe:0x0043596E`, `Darkages.exe:0x00435A5A`, `Darkages.exe:0x00435ACA`, `Darkages.exe:0x00435E90`, `Darkages.exe:0x00437310`, `Darkages.exe:0x00437AE8`. |
| `0x4D` | `CSpellDelayRequest` | `Darkages.exe:0x00455CDF`. Stone payload semantics not yet independently established. |
| `0x4E` | `CSpellDelaySay` | Direct sites `Darkages.exe:0x004548F1` and `Darkages.exe:0x00455921`; indirect submissions at `Darkages.exe:0x00455815` and `Darkages.exe:0x00456997` use the action `0x4E` builder at `Darkages.exe:0x00455834`. |
| `0x4F` | `CSendPortrait` | `Darkages.exe:0x0040A7E3`. `net_c_build_portrait_response` reads and validates the local portrait and includes its bytes in the response to server action `0x49`. |
| `0x57` | `CMulti` | `Darkages.exe:0x004A29D7`, `Darkages.exe:0x004A2EB0`. |
| `0x62` | literal `baram` | `Darkages.exe:0x004AD50B`. The five submitted logical bytes are the literal ASCII string `baram`, so byte zero is action `0x62`. No later-client `C...` class name is established for this special packet. |

## `baram` bootstrap packet

`Darkages.exe:0x004AD4E4` formats the literal ASCII string `baram` and submits exactly five bytes at `Darkages.exe:0x004AD50B`. The same control-flow branch immediately builds and queues `CVersion` at `Darkages.exe:0x004AD6A3`.

Action `0x62` is not one of the client XOR bypass values, so `net_c_encode_packet_body` inserts the current sequence byte and increments `net_c_send_sequence` when transmitting it. The global occupies uninitialized image storage and is therefore zero-filled when the process loads. On the initial startup path, `baram` carries sequence `0x00` and advances the client counter to `0x01`; `CVersion` then bypasses the sequence machinery. The only code references to the counter are the read and increment inside `net_c_encode_packet_body`, so repeating this path does not explicitly reset the client counter. The packet may reset or synchronize server-side tracking, but that effect is not visible in this client code.

## Server-directed dynamic send

Server action `0x4B` is handled by `Darkages.exe:0x0046CA54` `net_forward_embedded_client_packet`. It reads a big-endian `uint16_t` from server-packet offset `0x01`, treats bytes beginning at offset `0x03` as a logical client packet of that length, and submits them to `net_c_queue_send`. The embedded first byte therefore selects the client action dynamically.

This path still receives the normal zero sentinel, direction-specific transformation, frame header, and transfer-state gate. It does not bypass the Socket send machinery.
