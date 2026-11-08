# DaoMai Multi-Server ADB v6.4.0 — cổng COM qua adb (Control Box P20/P30)

## Mới

| Dịch vụ host | Trả lời |
|---|---|
| `host:daomai-serial-list` | Danh sách cổng serial của máy chạy daemon, mỗi dòng `COM3\tUSB-SERIAL CH340`; Windows còn thêm dòng `!\tCH340 (no driver)` khi có chip CH34x (VID 1A86) cắm vào nhưng chưa có driver |
| `host:daomai-serial:COM3[:115200]` | Cầu byte hai chiều tới cổng: sau `OKAY` của host, daemon gửi trạng thái thứ hai `OKAY` (đã mở) hoặc `FAIL` + lý do; từ đó socket là luồng byte thô |
| `host:daomai-serial-driver` | Windows: cài gói driver WCH CH341SER 3.9.2024 (ký số, **nhúng trong adb.exe**) bằng `pnputil /add-driver … /install` chạy nâng quyền (UAC hiện trên máy đó). Không có CH34x nào thiếu driver → `skipped`, không bật UAC |
| `host:daomai-node:<HOST:PORT>:<dịch vụ host>` | MAIN chuyển một dịch vụ host bất kỳ tới NODE trong danh sách (vd `daomai-node:192.168.1.20:5037:daomai-serial:COM3`), trả lời của NODE chuyển nguyên về client |

## Cách giữ cổng

Cổng COM chỉ mở trong lúc có client giữ socket. Hai luồng bơm byte (socket→cổng, cổng→socket); bất kỳ
phía nào lỗi (client đóng, cổng bị rút, ghi/đọc thất bại) thì **đóng cả cổng lẫn socket ngay** — adb
không bao giờ giữ COM khi không dùng. Cổng mở sai tên / đang bị chương trình khác giữ → `FAIL` kèm thông
báo Windows.

## Khác

- `adb version` → `Version 36.0.0-eng.daomai` + `kDaoMaiRebuildVersion = "6.4.0"`.
- Linux: liệt kê `/dev/ttyUSB*`, `/dev/ttyACM*`; mở bằng termios 8N1; `serial-driver` trả `not needed`.
- Link thêm `setupapi`, `cfgmgr32`, `shell32` trên Windows.

## Đã kiểm (Linux, 2 adb + cặp PTY `socat`, không phần cứng)

Xem `DAOMAI_VIEW.md` mục 6.4.0.
