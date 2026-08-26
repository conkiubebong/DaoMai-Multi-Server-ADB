# DaoMai Multi-Server ADB

**Tiếng Việt** · [English](#english) · [中文](#中文)

## Tiếng Việt

DaoMai Multi-Server ADB hợp nhất thiết bị Android từ nhiều ADB NODE trong mạng LAN vào một ADB MAIN duy nhất. Bản phát hành hỗ trợ scrcpy, Xiaowei, Android Studio, automation, shell, push/pull và port forwarding.

### Tải xuống

Tải bản mới nhất tại [GitHub Releases](../../releases/latest):

- Windows: `DaoMai-Multi-Server-ADB-phase6-windows.zip`
- Linux: `DaoMai-Multi-Server-ADB-phase6-linux.zip`

### Cấu hình nhanh

1. Giải nén gói phù hợp.
2. Tạo `server.txt` bên cạnh `adb.exe` hoặc `adb`.
3. Mỗi dòng ghi một ADB NODE theo dạng `HOST:PORT`:

```text
# Một NODE trên mỗi dòng
10.10.0.2:5037
10.10.0.3:5037
```

4. Khởi động lại ADB và kiểm tra:

```bash
adb kill-server
adb start-server
adb devices -l
```

MAIN trả ngay snapshot thiết bị hiện có. Các NODE được quét song song; NODE lỗi hoặc timeout không chặn NODE khỏe, và thiết bị đến sau sẽ xuất hiện ở lần gọi `adb devices` tiếp theo.`adb devices` hiển thị tổng số cùng thống kê USB, LAN/NODE, online, offline, recovery và trạng thái khác. Dùng `adb devices -D` để chia danh sách theo Máy Hiện Tại và từng địa chỉ NODE.

Gói Windows kèm `DaoMaiAdbManager.exe` để thay thế an toàn các bản ADB đang chạy. Hãy sao lưu và kiểm tra đường dẫn trước khi xác nhận thay thế.

> Repo public này chỉ phân phối README và binary release chính thức; không chứa mã nguồn.

## English

DaoMai Multi-Server ADB aggregates Android devices from multiple LAN ADB NODEs behind one MAIN ADB endpoint. It supports scrcpy, Xiaowei, Android Studio, automation, shell, push/pull, and port forwarding.

Download the latest Windows or Linux package from [GitHub Releases](../../releases/latest). Create `server.txt` beside the ADB executable and add one `HOST:PORT` NODE per line. Endpoint discovery is concurrent and incremental: unavailable NODEs do not block healthy NODEs or cached `adb devices` results. `adb devices` reports USB, LAN/NODE, online, offline, recovery and other counts; `adb devices -D` groups entries by MAIN and NODE endpoint.

> This public repository contains only documentation and official binary releases. Source code is not included.

## 中文

DaoMai Multi-Server ADB 可将多个局域网 ADB NODE 的 Android 设备汇总到一个 MAIN ADB 端点，支持 scrcpy、Xiaowei、Android Studio、自动化、shell、文件传输和端口转发。

请从 [GitHub Releases](../../releases/latest) 下载 Windows 或 Linux 最新版本。在 ADB 可执行文件旁创建 `server.txt`，每行填写一个 `HOST:PORT`。NODE 会并行、增量扫描：离线或超时的 NODE 不会阻塞正常 NODE，也不会阻塞已有的 `adb devices` 快照。`adb devices` 会统计 USB、LAN/NODE、在线、离线、recovery 和其他状态；`adb devices -D` 按 MAIN 与各 NODE 地址分组显示。

> 此公共仓库仅包含说明文档和官方二进制发布包，不包含源代码。