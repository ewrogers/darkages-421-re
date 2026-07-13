# `CBulletin` (`0x3B`)

[Previous: CMessage](058-3a-message.md) | [Client action index](../client-actions.md) | [Next: CPutToContainer](060-3c-put-to-container.md)

`CBulletin` is client-to-server action `0x3B` in the 4.21 protocol.

**Direction:** client to server

**Encrypted:** Yes. See [Sequence and XOR Transformation](../xor-transformation.md).

Payload offsets begin with the first byte after the action. The frame marker, frame length, action, sequence, and trailing zero are excluded.

## Payload format

Every `CBulletin` payload begins with a one-byte command subtype.

| Subtype | Purpose | Confirmed remaining payload |
|---:|---|---|
| `1` | Start bulletin session | None. The logical packet is `[0x3B, 0x01]`. |
| `2` | Request article or mail list | Two big-endian `uint16_t` values and one `uint8_t`. The concrete fields vary between board, article-list, and mail-list callers. |
| `3` | Request one article or mail | Two big-endian `uint16_t` identifiers and one `uint8_t`. |
| `4` | Post an article | Big-endian board identifier, one-byte title length and title, big-endian content length and content. |
| `5` | Delete an article or mail | Two big-endian `uint16_t` identifiers, with an additional zero byte in two list-originated variants. |
| `6` | Post mail | Big-endian identifier, one-byte receiver length and receiver, one-byte title length and title, big-endian content length and content. |
| `7` | Change article highlight state | Two big-endian `uint16_t` identifiers. |

## Queue call sites

| Queue call | Containing IDA function | Function address |
|---:|---|---:|
| `Darkages.exe:0x0040ECE1` | `net_c_send_bulletin_start` | `Darkages.exe:0x0040ECA4` |
| `Darkages.exe:0x004109EC` | `net_c_send_bulletin_article_list_request` | `Darkages.exe:0x00410964` |
| `Darkages.exe:0x0041294C` | `net_c_send_bulletin_mail_list_request` | `Darkages.exe:0x004128C4` |
| `Darkages.exe:0x004145B4` | `net_c_send_bulletin_board_article_list_request` | `Darkages.exe:0x00414534` |
| `Darkages.exe:0x00415738` | `net_c_send_bulletin_article_request` | `Darkages.exe:0x004156B4` |
| `Darkages.exe:0x00415934` | `net_c_send_bulletin_article_highlight` | `Darkages.exe:0x004158C4` |
| `Darkages.exe:0x0041715C` | `net_c_send_bulletin_article_request_from_dialog` | `Darkages.exe:0x004170D4` |
| `Darkages.exe:0x0041817E` | `net_c_send_bulletin_post_article` | `Darkages.exe:0x00417F94` |
| `Darkages.exe:0x004187E8` | `net_c_send_bulletin_mail_request` | `Darkages.exe:0x00418764` |
| `Darkages.exe:0x0041A73C` | `net_c_send_bulletin_mail_request_from_dialog` | `Darkages.exe:0x0041A6B4` |
| `Darkages.exe:0x0041AE45` | `net_c_send_bulletin_post_mail` | `Darkages.exe:0x0041AB84` |
| `Darkages.exe:0x0041B4E4` | `net_c_send_bulletin_delete_article_from_list` | `Darkages.exe:0x0041B474` |
| `Darkages.exe:0x0041B60B` | `net_c_send_bulletin_delete_article` | `Darkages.exe:0x0041B5A4` |
| `Darkages.exe:0x0041B8A8` | `net_c_send_bulletin_delete_mail_from_list` | `Darkages.exe:0x0041B824` |
| `Darkages.exe:0x0041BA1B` | `net_c_send_bulletin_delete_mail` | `Darkages.exe:0x0041B9B4` |

## UI flow

Window button W creates `BulletinSession` and sends subtype `1`. `SBulletin` subtype `1` creates BoardListDialog. Navigation, view, new, reply, delete, and highlight controls emit the later commands while the session remains registered. Server subtypes `1` through `5` replace the session's active child with the matching board, article, or mail dialog.

The command field widths above are established at all 15 send sites. Some identifier meanings differ between board and mail contexts and remain unnamed until their server-side correspondence is confirmed.

See [UI, Input, and Packet Flows](../../architecture/ui-network-flows.md#bulletin-and-mail-child-replacement).
