# DaoMai Multi-Server ADB

[Tiếng Việt](#tiếng-việt) · [English](#english) · [中文](#中文)

Aggregate local and remote Android devices from multiple ADB servers into one fast, compatible
ADB endpoint for scrcpy, Android Studio, automation, file transfer, shell, and port forwarding.

## Tiếng Việt

Một source ADB duy nhất build ra `adb.exe` Windows và `adb` Linux. Cùng binary có thể chạy như
NODE bình thường hoặc MAIN aggregator dựa trên file cấu hình nằm cạnh executable.

> Trạng thái: listener LAN, firewall, parser/whitelist, multi-port, app quản lý LAN, snapshot
> aggregation, virtual transport ID và smart-socket stream relay đã được triển khai. Xem kết quả
> build/live test và các soak test còn lại trong `ADB_MULTI_SERVER_PROGRESS.md`.

## Package

Windows:

```text
adb.exe
AdbWinApi.dll
AdbWinUsbApi.dll
DaoMaiAdbManager.exe
server.txt        # chỉ đặt ở MAIN
remote.txt        # tùy chọn ở NODE để whitelist device
```

Linux:

```text
adb
server.txt        # chỉ đặt ở MAIN
remote.txt        # tùy chọn ở NODE
```

File được tìm theo thư mục chứa executable, không phụ thuộc current working directory. Cả LF
(Linux) và CRLF (Windows), dòng trống, whitespace và comment bắt đầu bằng `#` đều được hỗ trợ.

## NODE / REMOTE

Không đặt `server.txt`. ADB giữ USB, TCP, emulator và toàn bộ command stock. Server mặc định bind
`0.0.0.0:PORT`, trong khi client trên cùng máy vẫn kết nối localhost.

```powershell
adb kill-server
adb devices -l
netstat -ano | findstr :5037
```

Device inventory commands:

```powershell
adb devices
adb devices -d
```

`adb devices` keeps the standard ADB header. `adb devices -d` adds `total`, direct MAIN `usb`,
LAN/NODE `lan`, online `onl`, offline `off`, recovery `rec`, and all remaining states as `other`,
then groups devices under `May Hien Tai` for MAIN-local transports and each configured NODE.

Khi server thực sự được spawn, output cho biết mode và mọi IPv4 LAN thực tế:

```text
* daemon not running; starting now at tcp:5037
  DaoMai ADB REMOTE/NODE
    10.10.0.2:5037
* daemon started successfully
```

Windows server thử tạo firewall rule `DaoMai ADB LAN PORT`: inbound TCP đúng port, remote
`LocalSubnet`, profile Domain/Private. Rule được kiểm tra idempotent, không mở Any/Public, không tắt
firewall và không tự bật UAC. Nếu thiếu Administrator permission, lỗi được ghi vào ADB log và server
vẫn chạy.

### Giới hạn device bằng remote.txt

Không có `remote.txt` nghĩa là cho phép tất cả device như stock. Nếu file tồn tại, mỗi dòng là một
serial hoặc USB device path được phép; file rỗng nghĩa là không nhận device nào.

```text
# Nhóm máy cho process này
5200785ab8011549
R58A001ABC
10.10.2.1:5555
emulator-5554
```

Whitelist được kiểm tra trước khi USB transport được claim, nhờ đó các ADB process/port khác nhau
không giữ nhầm toàn bộ USB. TCP device và emulator cũng chịu cùng whitelist. File được watch mỗi
giây; transport bị xóa khỏi whitelist sẽ được kick trên ADB fdevent thread.

## MAIN / AGGREGATOR

Đặt `server.txt` cạnh ADB:

```text
# DaoMai ADB Remote Servers
10.10.0.2:5037
10.10.0.2:5038
10.10.0.2:5039
10.10.0.3:5037
adb-node-01:5037
```

Mỗi `HOST:PORT` là một node độc lập; endpoint trùng hoàn toàn được loại bỏ. Hostname/IPv4 và port
1-65535 được validate. Dòng sai được log rồi bỏ qua, không làm ADB crash.

Startup MAIN hiển thị trực tiếp các endpoint:

```text
* daemon not running; starting now at tcp:5037
  DaoMai ADB SERVER
    10.10.0.2:5037
    10.10.0.2:5038
    10.10.0.3:5037
* daemon started successfully
```

Manager nền cache đầy đủ dòng `devices-l`: serial, trạng thái `device`, `offline`,
`unauthorized`, `recovery`, `sideload`, `bootloader`, product/model/device và virtual transport ID.
Node chết không được làm `adb devices` block; retry/reconnect và thay đổi device được đẩy tới các
variant `track-devices`.

Nếu MAIN đã có local TCP serial như `10.10.2.1:5555`, bản cùng serial do remote báo sẽ bị bỏ khỏi
aggregate để không relay vòng và không tăng tải. Duplicate USB serial giữa nhiều node sẽ dùng alias
ổn định rồi được dịch ngược về serial thật khi route upstream.

## Nhiều process / nhiều port

Stock client socket selection được giữ lại và mỗi port là một server process riêng:

```powershell
D:\ADB_Group_A\adb.exe -P 5037 devices
D:\ADB_Group_B\adb.exe -P 5038 devices
D:\ADB_Group_C\adb.exe -P 5039 devices
```

Mỗi thư mục có thể chứa `remote.txt`/`server.txt` riêng. Banner LAN và Windows firewall rule dùng
đúng port truyền qua `-P`, không hardcode 5037.

## Help và UTF-8

Các form sau đều hiện banner DaoMai và hướng dẫn multi-server:

```powershell
adb help
adb -help
adb --help
adb /?
adb -P 5038 help
```

Trên Windows, help đặt console output code page UTF-8 trước khi in tiếng Việt.

## DaoMai ADB LAN Manager

Source nằm tại `tools/daomai_adb_manager`. Đây là app C++ Win32 native, build thành một
`DaoMaiAdbManager.exe`, không cần Python/.NET/Qt.

- Combobox IP/hostname, Scan LAN, Edit/Save và ô port.
- Add IP/Remove IP cập nhật `server.txt` cạnh app/ADB.
- Scan subnet bằng bounded worker pool, UI không làm network I/O.
- Chỉ endpoint mở TCP port mới được query bằng `adb -H IP -P PORT devices -l`.
- Grid hiển thị tổng theo endpoint và chi tiết serial/state/model.
- Cấu hình UI lưu vào `manager_config.ini` cạnh EXE.

## Build

Tree build hiện dùng AOSP 14 AP2A và Windows host-cross:

```bash
cd /media/daomai/DATA/android/AOSP_14_STOCK
source build/envsetup.sh
lunch aosp_x86_64-ap2a-eng
HOST_CROSS_OS=windows m adb adb_test -j55
```

Artifact debug `eng` có symbol nên lớn (khoảng 62 MiB Windows). Package phát hành phải strip:

```bash
prebuilts/clang/host/linux-x86/clang-r510928/bin/llvm-strip path/to/adb.exe
prebuilts/clang/host/linux-x86/clang-r510928/bin/llvm-strip path/to/adb
```

Bản đã strip hiện khoảng 5.6 MiB Windows và 6.5 MiB Linux. Không strip DLL bằng cách tùy tiện nếu
chưa kiểm tra PE exports.

## Test bắt buộc trước release final

- NODE: local USB/TCP/emulator, shell, push/pull lớn, install, logcat, forward/reverse, scrcpy.
- LAN: listener `0.0.0.0:PORT`, direct `adb -H NODE -P PORT devices`, firewall LocalSubnet.
- MAIN: local + mọi remote trong một `devices`/`devices-l`; `-s` và `-t` route đúng.
- Long stream: shell/logcat/scrcpy và file lớn không có transfer timeout.
- Node chết/reconnect, cắm/rút phone, hot reload config, duplicate serial và local TCP collision.
- Track devices: short, long, proto-text và proto-binary mà branch hỗ trợ.

Kết quả quan sát được cập nhật trong `ADB_MULTI_SERVER_PROGRESS.md`; test phần cứng chưa chạy không
được ghi PASS.

## English

Detailed Xiaowei auto-view protocol notes, including its empty `tcp:` dynamic-port syntax,
stateful transport selection, `XWCaptureScreen` startup, `localabstract` retry behavior, and the
other observed modes are documented in [ADB_MULTI_SERVER_PROGRESS.md](ADB_MULTI_SERVER_PROGRESS.md#xiaowei-startup-and-view-protocol).

DaoMai Multi-Server ADB combines local Android devices and devices connected to multiple LAN ADB
nodes into one standard ADB server. Existing tools continue to use normal serial numbers or virtual
transport IDs without requiring client-side protocol changes.

### Packages

- NODE: place `adb.exe` with its DLLs and do not create `server.txt`.
- MAIN: place one `HOST:PORT` endpoint per line in `server.txt` next to `adb.exe`.
- Optional `remote.txt` on a NODE limits which USB, TCP, or emulator serials that process owns.

```text
# server.txt on MAIN
10.10.0.2:5037
10.10.0.2:5038
adb-node-01:5037
```

The server supports cached device aggregation, hot reload, stable duplicate aliases, virtual
transport IDs, shell, sync/push/pull, long-running binary streams, track-devices, and MAIN-local
forward relays required by scrcpy. Remote stream sockets have backpressure and no idle timeout.

```powershell
adb kill-server
adb devices -l
adb -s SERIAL shell
scrcpy -s SERIAL
```

Windows MAIN/NODE servers listen on the configured LAN port. Restrict access with the included
LocalSubnet firewall rule and never expose an unauthenticated ADB server to the public Internet.

## 中文

DaoMai Multi-Server ADB 可把本机 Android 设备和多个局域网 ADB 节点上的设备聚合到一个标准
ADB 服务中。scrcpy、Android Studio、自动化工具、文件传输和 shell 可以继续使用普通序列号或
虚拟 transport ID，无需修改客户端协议。

### 部署方式

- NODE：放置 `adb.exe` 和 DLL，不要创建 `server.txt`。
- MAIN：在 `adb.exe` 同目录的 `server.txt` 中，每行填写一个 `HOST:PORT`。
- NODE 可选用 `remote.txt` 限制该进程允许管理的 USB、TCP 或模拟器序列号。

```text
# MAIN 的 server.txt
10.10.0.2:5037
10.10.0.2:5038
adb-node-01:5037
```

项目支持设备列表缓存与热更新、稳定的重复序列号别名、虚拟 transport ID、shell、
sync/push/pull、长时间二进制流、track-devices，以及 scrcpy 所需的 MAIN 本地端口转发中继。
远程数据流具备背压控制，并且没有空闲超时。

```powershell
adb kill-server
adb devices -l
adb -s SERIAL shell
scrcpy -s SERIAL
```

Windows 上的 MAIN/NODE 会监听配置的局域网端口。请使用项目提供的 LocalSubnet 防火墙规则，
不要把未受保护的 ADB 服务暴露到公网。
