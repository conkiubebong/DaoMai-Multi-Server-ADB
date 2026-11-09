# DaoMai Multi-Server ADB v6.5.1 — share.txt + peer đối xứng (không đệ quy)

## Mới

`remote.txt` vẫn là danh sách máy NODE *nhận*. `share.txt` (cùng thư mục `server.txt`/`remote.txt`,
`DAOMAI_ADB_TXT_PATH`) là danh sách máy NODE *cho MAIN ở PC khác thấy*; app của chính NODE (nối từ
127.0.0.1) luôn thấy đủ. Không có file = chia sẻ tất cả (như các bản trước); file rỗng = không chia sẻ gì.

```text
# share.txt trên NODE
014AY1RN1M                        # mọi MAIN
10.10.250.152:5555@192.168.1.20   # chỉ MAIN nối từ 192.168.1.20
emulator-5554@192.168.1.20
emulator-5554@192.168.1.21        # nhiều dòng cho một serial được
```

| Dịch vụ host (trên NODE) | Trả lời |
|---|---|
| `host:daomai-share` | `mode=all\|list`, `source=runtime\|file\|none`, `path=<share.txt>`, rồi mỗi dòng một mục (`SERIAL` / `SERIAL@IP`); file lỗi có thêm `warning=…` |
| `host:daomai-set-share:<e1,e2,...>` | Đặt danh sách runtime (không cần file, không khởi động lại), trả `list`. `host:daomai-set-share:*` → chia sẻ tất cả (`all`); `host:daomai-set-share:-` → không chia sẻ gì (`list`). Mục sai → `FAIL` kèm lý do |
| `host:daomai-clear-share` | Bỏ danh sách runtime, quay về file: `file` (có share.txt) hoặc `none` (không có → chia sẻ tất cả) |

## Cách lọc

- Client là **REMOTE** khi địa chỉ TCP đối diện không phải loopback (127.0.0.0/8, `::1`, kể cả
  `::ffff:127.x`). Socket unix / không rõ địa chỉ → cục bộ.
- Với client REMOTE: `host:devices`, `host:devices-l`, `host:track-devices` (+ `-l`, `-proto-*`),
  `devices-daomai-summary/grouped/grouped-l` bỏ máy chưa chia sẻ; `host:transport:<serial>`,
  `host:transport-id:<id>`, `host:tport:*`, `host-serial:<serial>:*`, `host-transport-id:<id>:*`,
  `host:daomai-viewing:<serial>:on|off` với máy chưa chia sẻ → `FAIL device not shared`.
- Mục `SERIAL@IP` so với IPv4 của chính kết nối MAIN đó. Danh sách đánh giá tại thời điểm yêu cầu nên máy
  cắm vào sau cũng được lọc đúng. File đọc lại khi metadata ổn định ≥ 1 s; file sai cú pháp giữ danh sách
  tốt cuối; `track-devices` của MAIN nhận danh sách mới trong ~1 s sau khi đổi file / gọi `set-share`.

## Khác

- `adb version` → `kDaoMaiRebuildVersion = "6.5.1"`.
- Mã mới `remote_share_config.{h,cpp}` + gtest `remote_share_config_test.cpp`; `adb_getpeername` trong
  sysdeps (Windows: qua lớp FH như `adb_getsockname`). Không thêm thư viện Windows.
- `remote.txt`, `daomai-viewing`, relay MAIN ⇄ NODE giữ nguyên hành vi khi không có `share.txt`.

## Đã kiểm

Xem `DAOMAI_VIEW.md` mục 6.5.0.

## 6.5.1

- Peer đối xứng: PC khác hỏi qua LAN chỉ thấy máy CẮM Ở CHÍNH PC được hỏi, không bao giờ nhận lại máy gộp từ
  node thứ ba → A↔B trỏ lẫn nhau không đệ quy, không trùng máy. Client cục bộ vẫn thấy đủ.
