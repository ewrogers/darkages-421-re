# Client Actions

This index records every fixed action byte placed in a logical client packet before `net_queue_send`. The send-site addresses are the calls that submit the completed byte array. A neutral name means the action is established but its payload meaning has not yet reached the documentation threshold.

All entries use the client-to-server direction. The send queue appends a zero sentinel. Except for actions `0x00`, `0x48`, and `0x10`, `net_encode_packet_body` also inserts a sequence byte and applies the XOR transformation before framing.

## Fixed action index

| Action | Working name | Send sites and established notes |
|---:|---|---|
| `0x00` | `client_version` | `Darkages.exe:0x004AD6A3`. Logical fields are action, a big-endian data-version `uint16_t`, and ASCII `LK`; the builder submits 5 bytes before the send queue adds the sentinel. XOR bypass. |
| `0x02` | `client_action_02` | `Darkages.exe:0x00462730`. Semantic purpose not yet established. |
| `0x03` | `client_action_03` | `Darkages.exe:0x00462D2F`. Semantic purpose not yet established. |
| `0x04` | `client_action_04` | `Darkages.exe:0x00461725`. Semantic purpose not yet established. |
| `0x05` | `client_action_05` | `Darkages.exe:0x00465D7A`. Semantic purpose not yet established. |
| `0x06` | `client_action_06` | `Darkages.exe:0x0048853A`. Semantic purpose not yet established. |
| `0x07` | `client_action_07` | `Darkages.exe:0x00454213`. Semantic purpose not yet established. |
| `0x08` | `client_action_08` | `Darkages.exe:0x004530AE`. Semantic purpose not yet established. |
| `0x0B` | `client_action_0b` | `Darkages.exe:0x0046471C`, `Darkages.exe:0x004889DC`, `Darkages.exe:0x00488E2E`, `Darkages.exe:0x004922B1`, `Darkages.exe:0x0049236B`, `Darkages.exe:0x0049544D`. |
| `0x0C` | `client_action_0c` | `Darkages.exe:0x0046B1C2`. Semantic purpose not yet established. |
| `0x0D` | `client_action_0d` | `Darkages.exe:0x004C1691`, `Darkages.exe:0x004C1956`, `Darkages.exe:0x004C1A66`. |
| `0x0E` | `client_action_0e` | `Darkages.exe:0x004811F1`, `Darkages.exe:0x00482DA9`, `Darkages.exe:0x0048865F`, `Darkages.exe:0x004C0987`. |
| `0x10` | `client_transfer_server` | `Darkages.exe:0x00460617`, `Darkages.exe:0x00463119`, `Darkages.exe:0x0046C901`. A `MapPane::ProcessTransferServer` diagnostic names `kClientTransferServer`; this is the only action allowed while the Socket transfer gate is active. XOR bypass. |
| `0x11` | `client_action_11` | `Darkages.exe:0x0046822F`, `Darkages.exe:0x004884D4`. |
| `0x13` | `client_action_13` | `Darkages.exe:0x0048895C`. Semantic purpose not yet established. |
| `0x18` | `client_action_18` | `Darkages.exe:0x0043E39C`. Semantic purpose not yet established. |
| `0x19` | `client_action_19` | `Darkages.exe:0x004C1182`. Semantic purpose not yet established. |
| `0x1B` | `client_action_1b` | `Darkages.exe:0x00491AC4`. Semantic purpose not yet established. |
| `0x1C` | `client_action_1c` | `Darkages.exe:0x004533C6`. Semantic purpose not yet established. |
| `0x1D` | `client_action_1d` | `Darkages.exe:0x00488694`. Semantic purpose not yet established. |
| `0x23` | `client_action_23` | `Darkages.exe:0x00493C99`, `Darkages.exe:0x0049421C`. |
| `0x24` | `client_action_24` | `Darkages.exe:0x004534E1`. Semantic purpose not yet established. |
| `0x26` | `client_action_26` | `Darkages.exe:0x00463A45`, `Darkages.exe:0x004642F0`. |
| `0x29` | `client_action_29` | `Darkages.exe:0x0045336A`. Semantic purpose not yet established. |
| `0x2A` | `client_action_2a` | `Darkages.exe:0x0045360B`. Semantic purpose not yet established. |
| `0x2D` | `client_action_2d` | `Darkages.exe:0x0043D4DC`. Semantic purpose not yet established. |
| `0x2E` | `client_action_2e` | `Darkages.exe:0x004306AE`. Semantic purpose not yet established. |
| `0x2F` | `client_action_2f` | `Darkages.exe:0x0042F1D0`. Semantic purpose not yet established. |
| `0x30` | `client_action_30` | `Darkages.exe:0x0042499F`, `Darkages.exe:0x00441216`, `Darkages.exe:0x004427DF`, `Darkages.exe:0x0044395F`. |
| `0x38` | `client_action_38` | `Darkages.exe:0x00488110`. Semantic purpose not yet established. |
| `0x39` | `client_action_39` | `Darkages.exe:0x0047B8CD`, `Darkages.exe:0x0047BBB7`, `Darkages.exe:0x0047C175`, `Darkages.exe:0x0047C2CF`, `Darkages.exe:0x0047C60F`, `Darkages.exe:0x0047C8C8`, `Darkages.exe:0x0047D3BF`, `Darkages.exe:0x0047D46D`, `Darkages.exe:0x0047D7DC`. |
| `0x3A` | `client_action_3a` | `Darkages.exe:0x0047E85C`, `Darkages.exe:0x0047F9AC`, `Darkages.exe:0x0047FA89`, `Darkages.exe:0x004805CC`, `Darkages.exe:0x004806A9`, `Darkages.exe:0x0048113C`, `Darkages.exe:0x004812AE`, `Darkages.exe:0x00481EEC`, `Darkages.exe:0x0048205B`, `Darkages.exe:0x00482B5C`, `Darkages.exe:0x00482F1C`. |
| `0x3B` | `client_action_3b` | `Darkages.exe:0x0040ECE1`, `Darkages.exe:0x004109EC`, `Darkages.exe:0x0041294C`, `Darkages.exe:0x004145B4`, `Darkages.exe:0x00415738`, `Darkages.exe:0x00415934`, `Darkages.exe:0x0041715C`, `Darkages.exe:0x0041817E`, `Darkages.exe:0x004187E8`, `Darkages.exe:0x0041A73C`, `Darkages.exe:0x0041AE45`, `Darkages.exe:0x0041B4E4`, `Darkages.exe:0x0041B60B`, `Darkages.exe:0x0041B8A8`, `Darkages.exe:0x0041BA1B`. |
| `0x3C` | `client_action_3c` | `Darkages.exe:0x00424B24`. Semantic purpose not yet established. |
| `0x3D` | `client_action_3d` | `Darkages.exe:0x004412D8`, `Darkages.exe:0x00453A15`. |
| `0x3E` | `client_action_3e` | `Darkages.exe:0x00454C96`. Semantic purpose not yet established. |
| `0x3F` | `client_action_3f` | `Darkages.exe:0x0043953A`. Semantic purpose not yet established. |
| `0x41` | `client_action_41` | `Darkages.exe:0x00494E2C`. Semantic purpose not yet established. |
| `0x42` | `client_action_42` | `Darkages.exe:0x00434B43`. Semantic purpose not yet established. |
| `0x43` | `client_action_43` | `Darkages.exe:0x0047B755`, `Darkages.exe:0x00488723`, `Darkages.exe:0x0048C820`, `Darkages.exe:0x0048F17F`. |
| `0x44` | `client_action_44` | `Darkages.exe:0x0042F8E2`. Semantic purpose not yet established. |
| `0x45` | `client_action_45` | `Darkages.exe:0x00465522`. Semantic purpose not yet established. |
| `0x46` | `client_action_46` | `Darkages.exe:0x0042F919`. Semantic purpose not yet established. |
| `0x47` | `client_action_47` | `Darkages.exe:0x0044028E`. Semantic purpose not yet established. |
| `0x48` | `client_action_48` | `Darkages.exe:0x0048763C`, `Darkages.exe:0x004953EC`, `Darkages.exe:0x00495F55`, `Darkages.exe:0x0049609D`. XOR bypass. |
| `0x4A` | `client_action_4a` | `Darkages.exe:0x0043071A`, `Darkages.exe:0x0043596E`, `Darkages.exe:0x00435A5A`, `Darkages.exe:0x00435ACA`, `Darkages.exe:0x00435E90`, `Darkages.exe:0x00437310`, `Darkages.exe:0x00437AE8`. |
| `0x4D` | `client_action_4d` | `Darkages.exe:0x00455CDF`. Semantic purpose not yet established. |
| `0x4E` | `client_action_4e` | Direct sites `Darkages.exe:0x004548F1` and `Darkages.exe:0x00455921`; indirect submissions at `Darkages.exe:0x00455815` and `Darkages.exe:0x00456997` use the action `0x4E` builder at `Darkages.exe:0x00455834`. |
| `0x4F` | `client_portrait_response` | `Darkages.exe:0x0040A7E3`. `net_build_portrait_response` reads and validates the local portrait and includes its bytes in the response to server action `0x49`. |
| `0x57` | `client_action_57` | `Darkages.exe:0x004A29D7`, `Darkages.exe:0x004A2EB0`. |
| `0x62` | `client_baram_handshake` | `Darkages.exe:0x004AD50B`. The five submitted logical bytes are the literal ASCII string `baram`, so byte zero is action `0x62`. |

## Server-directed dynamic send

Server action `0x4B` is handled by `Darkages.exe:0x0046CA54` `net_forward_embedded_client_packet`. It reads a big-endian `uint16_t` from server-packet offset `0x01`, treats bytes beginning at offset `0x03` as a logical client packet of that length, and submits them to `net_queue_send`. The embedded first byte therefore selects the client action dynamically.

This path still receives the normal zero sentinel, direction-specific transformation, frame header, and transfer-state gate. It does not bypass the Socket send machinery.
