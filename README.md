# DaoMai Multi-Server ADB

> **v6.2.5: truyền file lớn hiệu năng cao.** Thêm `push-daomai`/`pull-daomai`, tự chọn LZ4 hoặc
> raw fast path, hỗ trợ kích thước 64-bit và đã kiểm tra đồng thời trên toàn bộ device online.

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
adb devices -pd
adb devices -d -l
```

Truyền file lớn tự tối ưu:

```powershell
adb -s SERIAL push-daomai FILE_LOCAL /data/local/tmp/FILE
adb -s SERIAL pull-daomai /data/local/tmp/FILE FILE_LOCAL
adb -s SERIAL push-daomai -z lz4 FILE_LOCAL /data/local/tmp/FILE
adb -s SERIAL push-daomai -Z FILE_LOCAL /data/local/tmp/FILE
adb -s SERIAL push-daomai "THU MUC LOCAL" "/data/local/tmp/THU MUC REMOTE"
adb -s SERIAL pull-daomai "/data/local/tmp/THU MUC REMOTE" "THU MUC LOCAL"
```

- `push-daomai` đọc mẫu 4 MiB đầu file. Nếu tỷ lệ LZ4 nhỏ hơn 90%, lệnh dùng LZ4; nếu file đã
  nén hoặc khó nén, lệnh dùng raw sync tối ưu để tránh tốn CPU và một lần copy cho mỗi packet.
- `pull-daomai` chọn raw cho các định dạng đã nén phổ biến (`zip`, `apk`, video, ảnh, Git pack...)
  và LZ4 cho dữ liệu còn lại. Có thể ép thuật toán bằng `-z none|brotli|lz4|zstd` hoặc tắt nén bằng `-Z`.
- Cả hai lệnh dùng ADB Sync chuẩn, tương thích adbd stock, MAIN/NODE và routing `-s`/`-P` hiện có.
  Kích thước và tiến độ dùng 64-bit; file trên 4 GiB không bị giới hạn bởi trường DATA 64 KiB.
- Không chia nhiều shell stream song song qua MAIN/NODE: stress thực tế cho thấy relay không
  multiplex an toàn kiểu đó. Auto compression cho throughput cao hơn và giữ transport ổn định.
- Hỗ trợ cả file và thư mục đệ quy. Auto-mode của thư mục dùng LZ4. Đường dẫn có khoảng trắng phải
  đặt trong dấu nháy kép. Lỗi ở file/thư mục con được truyền ra exit code, không còn trường hợp báo
  thành công nhưng âm thầm thiếu file. Đã kiểm tra push/pull thường và DaoMai với file
  4.370.142.603 byte trong thư mục; số file, kích thước và SHA-256 đều khớp.

Ý nghĩa lệnh mới:

- `adb devices`: tự dò các ADB local liên tiếp từ port 5037 đến 5057, gộp serial không trùng và
  hiện version, tổng DaoMaiX cùng số máy `Server`/`Remote` ngay trên dòng đầu. Không cần `server.txt`.
- `adb devices -d` (hoặc `-D`): hiện tổng toàn bộ port, danh sách port, thống kê và thiết bị của
  từng Server/Remote Port. Nếu một Server Port có `server.txt`, mỗi `HOST:PORT` trong file được hiện
  thành nhóm `# Remote Port: HOST:PORT` riêng, kèm total/USB/LAN/trạng thái.
- `adb devices -pd` hoặc `adb devices -dp`: dạng gọn theo port, gồm `# TOTAL`, `# TOTAL PORT` và
  danh sách thiết bị dưới từng `# Port`.
- Thêm `-l` ở bất kỳ vị trí hợp lệ nào để hiện product, model, device và transport ID, ví dụ
  `adb devices -d -l` hoặc `adb devices -l -pd`.
- `adb -P 5038 devices`: chỉ truy vấn đúng port 5038, không tự gộp các port khác.
- `adb -s SERIAL shell` và các lệnh dùng `-s`: khi không chỉ định server, client tự tìm local port
  đang sở hữu serial rồi chuyển lệnh đến đúng daemon.
- `adb -d COMMAND`, `adb -e COMMAND` và `adb -t ID COMMAND`: khi không có `-P/-H/-L`, client tự
  tìm daemon local chứa USB, TCP/emulator hoặc transport ID tương ứng. Nếu selector xuất hiện ở
  nhiều port, ADB báo mơ hồ thay vì âm thầm gọi nhầm 5037.
- `adb shell`/`adb logcat`: nếu toàn bộ các port chỉ có đúng một thiết bị, client tự chọn thiết bị
  đó; nếu có nhiều thiết bị, ADB vẫn báo lỗi chọn thiết bị theo hành vi chuẩn.
- `adb root`/`adb unroot`: `-s SERIAL` luôn route tới đúng port; khi không có selector và toàn hệ
  thống chỉ có một thiết bị, lệnh cũng tự route như ADB gốc.
- `adb connect HOST:PORT`: kết nối qua server đang chọn; dùng `adb -P PORT connect HOST:PORT` để
  ép một daemon cụ thể.
- `adb devices help`, `adb devices -help`, `adb devices --help`, `adb devices /?`: hiện hướng dẫn
  đầy đủ thay vì báo usage.

### Thư mục server.txt và remote.txt dùng chung

ADB đọc cả hai file từ thư mục trong biến `DAOMAI_ADB_TXT_PATH`. Nếu biến không tồn tại hoặc rỗng,
ADB fallback về thư mục chứa chính `adb.exe`/`adb`, nên cấu hình cũ vẫn hoạt động bình thường.

Trên Windows, chép `save_path_txt.bat` và `remove_patch_txt.bat` vào package. Mở thư mục muốn chứa
`server.txt`/`remote.txt`, chạy `save_path_txt.bat` để lưu đường dẫn hiện tại. Chạy
`remove_patch_txt.bat` để xóa biến. Sau mỗi thay đổi đường dẫn, chạy `adb kill-server` rồi gọi lại
`adb devices`; thay đổi nội dung file vẫn được hot-reload như trước.

Without `-P`, `adb devices` automatically discovers consecutive local ADB servers starting at port
5037, merges their unique serials, and shows the DaoMaiX total summary on the first line. `adb devices -d` adds total,
physical USB `usb`, TCP/IP `lan`, online `onl`, offline `off`, recovery `rec`, and `other` counts for
every detected port. `-pd`/`-dp` prints the compact per-port device grouping. Add `-l` to either
mode for product/model/device/transport details. An explicit `-P PORT`, `-H HOST`, or `-L SOCKET`
disables discovery and queries only that server. Commands using `-s SERIAL` automatically route to
the local ADB port that owns the serial when no server endpoint was explicitly selected.

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

## Test bắt buộc trước release final

- NODE: local USB/TCP/emulator, shell, push/pull lớn, install, logcat, forward/reverse, scrcpy.
- LAN: listener `0.0.0.0:PORT`, direct `adb -H NODE -P PORT devices`, firewall LocalSubnet.
- MAIN: local + mọi remote trong một `devices`/`devices-l`; `-s` và `-t` route đúng.
- Long stream: shell/logcat/scrcpy và file lớn không có transfer timeout.
- Node chết/reconnect, cắm/rút phone, hot reload config, duplicate serial và local TCP collision.
- Track devices: short, long, proto-text và proto-binary mà branch hỗ trợ.

Kết quả quan sát được cập nhật trong `ADB_MULTI_SERVER_PROGRESS.md`; test phần cứng chưa chạy không
được ghi PASS.

### Cách dùng chi tiết server.txt và remote.txt

`server.txt` chỉ dùng cho MAIN. Mỗi dòng hợp lệ là một NODE `HOST:PORT`; hostname hoặc IPv4 đều
được hỗ trợ. Dòng trống và dòng bắt đầu bằng `#` bị bỏ qua; endpoint trùng được loại bỏ. Không có
`server.txt` nghĩa là NODE bình thường. File tồn tại nhưng rỗng vẫn bật MAIN nhưng không lấy NODE.

`remote.txt` là whitelist thiết bị của process/NODE hiện tại. Mỗi dòng là serial USB, serial TCP
`IP:PORT`, emulator hoặc USB device path. Không có file nghĩa là cho phép tất cả; file rỗng nghĩa là
chặn tất cả. Thêm/xóa dòng sẽ được kiểm tra mỗi giây và transport bị loại sẽ được ngắt an toàn.

```text
# server.txt trên MAIN
10.10.1.7:5037
10.10.1.7:5038
adb-node-kho-02:5037

# remote.txt trên NODE
014AY1RN1M
10.10.250.152:5555
emulator-5554
3-1.1.1
```

Watcher cache `path + trạng thái tồn tại + size + mtime`, chỉ đọc nội dung sau khi metadata ổn định
ít nhất một giây. Bản đang ghi dở hoặc sai cú pháp không thay thế cấu hình hợp lệ cuối. Tạo, sửa hoặc
xóa `server.txt`/`remote.txt` đều hot-reload; chỉ khi đổi `DAOMAI_ADB_TXT_PATH` mới cần restart daemon.

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

### New multi-port commands

- `adb devices` discovers consecutive local ADB servers on ports 5037-5057, merges duplicate
  serials, and prints the DaoMaiX totals plus `Server`/`Remote` device counts on the first line. It
  does not require `server.txt`.
- `adb devices -d` (uppercase `-D` is also accepted) prints overall totals, detected ports, and a
  detailed Server/Remote Port section for each daemon. When a Server Port has `server.txt`, every
  configured `HOST:PORT` is shown as its own `# Remote Port: HOST:PORT` group with counts.
- `adb devices -pd` or `adb devices -dp` prints the compact layout with `# TOTAL`,
  `# TOTAL PORT`, and the devices grouped below each `# Port`.
- Add `-l` in either order for product, model, device, and transport details, for example
  `adb devices -d -l` or `adb devices -l -pd`.
- `adb -P 5038 devices` remains strict and queries only port 5038.
- Without an explicit server, `adb -s SERIAL ...` finds the local port that owns the serial and
  routes the command to that daemon. `shell` and `logcat` also auto-route when exactly one unique
  device exists across all discovered ports; normal ambiguity errors remain when several exist.
- `adb root`/`adb unroot` honor `-s SERIAL` routing and auto-route when exactly one unique device
  exists across all local ports.
- Original selectors `adb -d`, `adb -e`, and `adb -t ID` also auto-route across local ports when
  no endpoint is explicit. Cross-port ambiguity is reported instead of silently querying 5037.
- `adb connect HOST:PORT` uses the selected server. Use `adb -P PORT connect HOST:PORT` to select
  a specific daemon explicitly.
- `adb devices help`, `-help`, `--help`, and `/?` display the full inventory help.

`DAOMAI_ADB_TXT_PATH` selects a shared directory for both `server.txt` and `remote.txt`. If it is
unset or empty, ADB falls back to the directory containing its executable, preserving existing
deployments. On Windows, run `save_path_txt.bat` from the desired config directory to save it, or
`remove_patch_txt.bat` to clear it. Restart the daemon after changing the path; file contents retain
their existing hot-reload behavior.

### Detailed server.txt and remote.txt usage

`server.txt` enables MAIN mode. Each non-comment line is one `HOST:PORT` NODE; IPv4 and hostnames
are accepted, blank/comment lines are ignored, and duplicates are removed. A missing file means
normal NODE mode. An existing empty file means MAIN mode with no remote endpoints.

`remote.txt` is the current process/NODE device allowlist. Put one USB serial, TCP `IP:PORT`,
emulator serial, or USB device path on each line. A missing file allows every device; an empty file
allows none. Cached `path + existence + size + mtime` metadata is checked every second; contents are
read only after it remains stable for one second. A malformed or partially-written update cannot
replace the last valid cache. Creating, editing, or removing either TXT file is hot-reloaded;
changing `DAOMAI_ADB_TXT_PATH` still requires a daemon restart.

### High-performance large-file transfer

```powershell
adb -s SERIAL push-daomai LOCAL REMOTE
adb -s SERIAL pull-daomai REMOTE LOCAL
adb -s SERIAL push-daomai -z lz4 LOCAL REMOTE
adb -s SERIAL push-daomai -Z LOCAL REMOTE
adb -s SERIAL push-daomai "LOCAL DIRECTORY" "/data/local/tmp/REMOTE DIRECTORY"
adb -s SERIAL pull-daomai "/data/local/tmp/REMOTE DIRECTORY" "LOCAL DIRECTORY"
```

`push-daomai` compresses a 4 MiB sample with LZ4 and uses LZ4 when the sample is below 90% of its
original size; already-compressed data uses the optimized raw sendrecv_v2 path. `pull-daomai`
selects raw mode for common archives, packages, media, images, and Git pack files, and LZ4 for other
paths. Override the decision with `-z none|brotli|lz4|zstd` or `-Z`. Both commands retain the stock
ADB Sync protocol, 64-bit sizes, `-s` routing, explicit `-P/-H/-L`, and stock Android adbd
compatibility. Parallel shell chunks are intentionally not used because MAIN/NODE relays cannot
safely multiplex that transfer pattern.

Files and recursive directories are supported; directory auto mode uses LZ4. Quote paths containing
spaces. Recursive failures propagate to the outer exit code, so a failed child can no longer be
silently omitted. Ordinary and DaoMai push/pull were verified with a 4,370,142,603-byte file inside
a directory by checking file count, 64-bit size, and SHA-256 after both directions.

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

### 新增多端口命令

- `adb devices` 自动扫描从 5037 到 5057 的连续本机 ADB 端口，合并重复序列号，并在首行显示
  DaoMaiX 总计以及 `Server`/`Remote` 设备数；无需 `server.txt`。
- `adb devices -d`（也兼容大写 `-D`）显示所有端口的总计、端口列表，以及每个
  Server/Remote Port 的统计与设备。如果 Server Port 配置了 `server.txt`，文件中的每个
  `HOST:PORT` 都会作为独立的 `# Remote Port: HOST:PORT` 分组显示并附带统计。
- `adb devices -pd` 或 `adb devices -dp` 使用紧凑格式，依次显示 `# TOTAL`、
  `# TOTAL PORT`，并在每个 `# Port` 下列出设备。
- 可以组合 `-l` 查看 product、model、device 和 transport ID，例如 `adb devices -d -l`
  或 `adb devices -l -pd`。
- `adb -P 5038 devices` 只查询 5038，不会合并其他端口。
- 未明确指定服务端时，`adb -s SERIAL ...` 会自动查找拥有该序列号的本机端口并路由命令。
  如果所有端口合计只有一台设备，`adb shell` 和 `adb logcat` 也会自动选择；多台设备时仍保留
  标准 ADB 的选择冲突提示。
- `adb root`/`adb unroot` 支持 `-s SERIAL` 自动路由；未指定 selector 且所有端口合计只有一台
  设备时，也会按原生 ADB 行为自动选择该设备。
- 原生 selector `adb -d`、`adb -e` 和 `adb -t ID` 在没有明确 `-P/-H/-L` 时也会自动跨本机
  端口路由；如果多个端口同时匹配，则明确报告冲突，不会静默误查 5037。
- `adb connect HOST:PORT` 使用当前选择的服务端；可通过
  `adb -P PORT connect HOST:PORT` 明确指定某个 daemon。
- `adb devices help`、`-help`、`--help` 和 `/?` 会显示完整的设备清单帮助。

### 大文件高速传输

```powershell
adb -s SERIAL push-daomai 本地文件 /data/local/tmp/文件
adb -s SERIAL pull-daomai /data/local/tmp/文件 本地文件
adb -s SERIAL push-daomai -z lz4 本地文件 /data/local/tmp/文件
adb -s SERIAL push-daomai -Z 本地文件 /data/local/tmp/文件
adb -s SERIAL push-daomai "本地目录" "/data/local/tmp/远程目录"
adb -s SERIAL pull-daomai "/data/local/tmp/远程目录" "本地目录"
```

`push-daomai` 会对文件开头 4 MiB 进行 LZ4 采样：压缩后低于原大小 90% 时使用 LZ4，已经压缩或
难以压缩的文件使用优化后的 raw sendrecv_v2 路径。`pull-daomai` 会根据远端文件类型选择 LZ4 或
raw；也可以用 `-z none|brotli|lz4|zstd` 或 `-Z` 强制指定。两个命令继续使用标准 ADB Sync、
64 位文件大小、现有 `-s`/`-P` 路由，并兼容原生 Android adbd。MAIN/NODE 无法安全复用并行 shell
分块流，因此正式版本不会启用这种不稳定方式。

同时支持单文件和递归目录；目录自动模式使用 LZ4。包含空格的路径必须加双引号。子目录或子文件
失败会正确传递到外层退出码，不再出现命令成功但缺少大文件的情况。普通与 DaoMai push/pull 均已
使用目录内 4,370,142,603 字节文件验证，双向传输后的文件数、64 位大小和 SHA-256 完全一致。

环境变量 `DAOMAI_ADB_TXT_PATH` 可为 `server.txt` 和 `remote.txt` 指定共享目录。变量未设置
或为空时，ADB 会自动回退到可执行文件所在目录，因此旧配置保持兼容。Windows 用户可在目标
配置目录运行 `save_path_txt.bat` 保存路径，运行 `remove_patch_txt.bat` 清除路径。修改路径后需
重启 daemon；文件内容本身仍会按原方式热更新。

### server.txt 与 remote.txt 详细用法

`server.txt` 用于启用 MAIN 模式。每个非注释行填写一个 `HOST:PORT` NODE，支持 IPv4 与主机名；
空行、以 `#` 开头的注释和重复端点会被忽略。文件不存在表示普通 NODE；文件存在但为空表示
MAIN 模式已启用，但当前没有远程端点。

`remote.txt` 是当前进程/NODE 的设备白名单，每行可填写 USB 序列号、TCP `IP:PORT`、模拟器
序列号或 USB device path。文件不存在表示允许全部设备；空文件表示不允许任何设备。Watcher 每秒
检查缓存的 `path + 是否存在 + size + mtime`，metadata 稳定一秒后才读取内容；写入未完成或语法
错误的版本不会替换最后有效缓存。创建、编辑或删除两个 TXT 文件均支持热更新；仅修改
`DAOMAI_ADB_TXT_PATH` 时需要重启 daemon。

Windows 上的 MAIN/NODE 会监听配置的局域网端口。请使用项目提供的 LocalSubnet 防火墙规则，
不要把未受保护的 ADB 服务暴露到公网。
