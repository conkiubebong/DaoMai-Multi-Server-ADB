# DaoMai Multi-Server ADB

[Tiáº¿ng Viá»‡t](#tiáº¿ng-viá»‡t) Â· [English](#english) Â· [ä¸­æ–‡](#ä¸­æ–‡)

Aggregate local and remote Android devices from multiple ADB servers into one fast, compatible
ADB endpoint for scrcpy, Android Studio, automation, file transfer, shell, and port forwarding.

## Tiáº¿ng Viá»‡t

Má»™t source ADB duy nháº¥t build ra `adb.exe` Windows vÃ  `adb` Linux. CÃ¹ng binary cÃ³ thá»ƒ cháº¡y nhÆ°
NODE bÃ¬nh thÆ°á»ng hoáº·c MAIN aggregator dá»±a trÃªn file cáº¥u hÃ¬nh náº±m cáº¡nh executable.

> Tráº¡ng thÃ¡i: listener LAN, firewall, parser/whitelist, multi-port, app quáº£n lÃ½ LAN, snapshot
> aggregation, virtual transport ID vÃ  smart-socket stream relay Ä‘Ã£ Ä‘Æ°á»£c triá»ƒn khai. Xem káº¿t quáº£
> build/live test vÃ  cÃ¡c soak test cÃ²n láº¡i trong `ADB_MULTI_SERVER_PROGRESS.md`.

## Package

Windows:

```text
adb.exe
AdbWinApi.dll
AdbWinUsbApi.dll
DaoMaiAdbManager.exe
server.txt        # chá»‰ Ä‘áº·t á»Ÿ MAIN
remote.txt        # tÃ¹y chá»n á»Ÿ NODE Ä‘á»ƒ whitelist device
```

Linux:

```text
adb
server.txt        # chá»‰ Ä‘áº·t á»Ÿ MAIN
remote.txt        # tÃ¹y chá»n á»Ÿ NODE
```

File Ä‘Æ°á»£c tÃ¬m theo thÆ° má»¥c chá»©a executable, khÃ´ng phá»¥ thuá»™c current working directory. Cáº£ LF
(Linux) vÃ  CRLF (Windows), dÃ²ng trá»‘ng, whitespace vÃ  comment báº¯t Ä‘áº§u báº±ng `#` Ä‘á»u Ä‘Æ°á»£c há»— trá»£.

## NODE / REMOTE

KhÃ´ng Ä‘áº·t `server.txt`. ADB giá»¯ USB, TCP, emulator vÃ  toÃ n bá»™ command stock. Server máº·c Ä‘á»‹nh bind
`0.0.0.0:PORT`, trong khi client trÃªn cÃ¹ng mÃ¡y váº«n káº¿t ná»‘i localhost.

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

The normal header includes `total`, direct MAIN `usb`, LAN/NODE `lan`, online `onl`, offline
`off`, recovery `rec`, and all remaining states as `other`. `adb devices -d` groups devices under
`May Hien Tai` for MAIN-local transports and under each configured NODE endpoint.

Khi server thá»±c sá»± Ä‘Æ°á»£c spawn, output cho biáº¿t mode vÃ  má»i IPv4 LAN thá»±c táº¿:

```text
* daemon not running; starting now at tcp:5037
  DaoMai ADB REMOTE/NODE
    10.10.0.2:5037
* daemon started successfully
```

Windows server thá»­ táº¡o firewall rule `DaoMai ADB LAN PORT`: inbound TCP Ä‘Ãºng port, remote
`LocalSubnet`, profile Domain/Private. Rule Ä‘Æ°á»£c kiá»ƒm tra idempotent, khÃ´ng má»Ÿ Any/Public, khÃ´ng táº¯t
firewall vÃ  khÃ´ng tá»± báº­t UAC. Náº¿u thiáº¿u Administrator permission, lá»—i Ä‘Æ°á»£c ghi vÃ o ADB log vÃ  server
váº«n cháº¡y.

### Giá»›i háº¡n device báº±ng remote.txt

KhÃ´ng cÃ³ `remote.txt` nghÄ©a lÃ  cho phÃ©p táº¥t cáº£ device nhÆ° stock. Náº¿u file tá»“n táº¡i, má»—i dÃ²ng lÃ  má»™t
serial hoáº·c USB device path Ä‘Æ°á»£c phÃ©p; file rá»—ng nghÄ©a lÃ  khÃ´ng nháº­n device nÃ o.

```text
# NhÃ³m mÃ¡y cho process nÃ y
5200785ab8011549
R58A001ABC
10.10.2.1:5555
emulator-5554
```

Whitelist Ä‘Æ°á»£c kiá»ƒm tra trÆ°á»›c khi USB transport Ä‘Æ°á»£c claim, nhá» Ä‘Ã³ cÃ¡c ADB process/port khÃ¡c nhau
khÃ´ng giá»¯ nháº§m toÃ n bá»™ USB. TCP device vÃ  emulator cÅ©ng chá»‹u cÃ¹ng whitelist. File Ä‘Æ°á»£c watch má»—i
giÃ¢y; transport bá»‹ xÃ³a khá»i whitelist sáº½ Ä‘Æ°á»£c kick trÃªn ADB fdevent thread.

## MAIN / AGGREGATOR

Äáº·t `server.txt` cáº¡nh ADB:

```text
# DaoMai ADB Remote Servers
10.10.0.2:5037
10.10.0.2:5038
10.10.0.2:5039
10.10.0.3:5037
adb-node-01:5037
```

Má»—i `HOST:PORT` lÃ  má»™t node Ä‘á»™c láº­p; endpoint trÃ¹ng hoÃ n toÃ n Ä‘Æ°á»£c loáº¡i bá». Hostname/IPv4 vÃ  port
1-65535 Ä‘Æ°á»£c validate. DÃ²ng sai Ä‘Æ°á»£c log rá»“i bá» qua, khÃ´ng lÃ m ADB crash.

Startup MAIN hiá»ƒn thá»‹ trá»±c tiáº¿p cÃ¡c endpoint:

```text
* daemon not running; starting now at tcp:5037
  DaoMai ADB SERVER
    10.10.0.2:5037
    10.10.0.2:5038
    10.10.0.3:5037
* daemon started successfully
```

Manager ná»n cache Ä‘áº§y Ä‘á»§ dÃ²ng `devices-l`: serial, tráº¡ng thÃ¡i `device`, `offline`,
`unauthorized`, `recovery`, `sideload`, `bootloader`, product/model/device vÃ  virtual transport ID.
Node cháº¿t khÃ´ng Ä‘Æ°á»£c lÃ m `adb devices` block; retry/reconnect vÃ  thay Ä‘á»•i device Ä‘Æ°á»£c Ä‘áº©y tá»›i cÃ¡c
variant `track-devices`.

Náº¿u MAIN Ä‘Ã£ cÃ³ local TCP serial nhÆ° `10.10.2.1:5555`, báº£n cÃ¹ng serial do remote bÃ¡o sáº½ bá»‹ bá» khá»i
aggregate Ä‘á»ƒ khÃ´ng relay vÃ²ng vÃ  khÃ´ng tÄƒng táº£i. Duplicate USB serial giá»¯a nhiá»u node sáº½ dÃ¹ng alias
á»•n Ä‘á»‹nh rá»“i Ä‘Æ°á»£c dá»‹ch ngÆ°á»£c vá» serial tháº­t khi route upstream.

## Nhiá»u process / nhiá»u port

Stock client socket selection Ä‘Æ°á»£c giá»¯ láº¡i vÃ  má»—i port lÃ  má»™t server process riÃªng:

```powershell
D:\ADB_Group_A\adb.exe -P 5037 devices
D:\ADB_Group_B\adb.exe -P 5038 devices
D:\ADB_Group_C\adb.exe -P 5039 devices
```

Má»—i thÆ° má»¥c cÃ³ thá»ƒ chá»©a `remote.txt`/`server.txt` riÃªng. Banner LAN vÃ  Windows firewall rule dÃ¹ng
Ä‘Ãºng port truyá»n qua `-P`, khÃ´ng hardcode 5037.

## Help vÃ  UTF-8

CÃ¡c form sau Ä‘á»u hiá»‡n banner DaoMai vÃ  hÆ°á»›ng dáº«n multi-server:

```powershell
adb help
adb -help
adb --help
adb /?
adb -P 5038 help
```

TrÃªn Windows, help Ä‘áº·t console output code page UTF-8 trÆ°á»›c khi in tiáº¿ng Viá»‡t.

## DaoMai ADB LAN Manager

Source náº±m táº¡i `tools/daomai_adb_manager`. ÄÃ¢y lÃ  app C++ Win32 native, build thÃ nh má»™t
`DaoMaiAdbManager.exe`, khÃ´ng cáº§n Python/.NET/Qt.

- Combobox IP/hostname, Scan LAN, Edit/Save vÃ  Ã´ port.
- Add IP/Remove IP cáº­p nháº­t `server.txt` cáº¡nh app/ADB.
- Scan subnet báº±ng bounded worker pool, UI khÃ´ng lÃ m network I/O.
- Chá»‰ endpoint má»Ÿ TCP port má»›i Ä‘Æ°á»£c query báº±ng `adb -H IP -P PORT devices -l`.
- Grid hiá»ƒn thá»‹ tá»•ng theo endpoint vÃ  chi tiáº¿t serial/state/model.
- Cáº¥u hÃ¬nh UI lÆ°u vÃ o `manager_config.ini` cáº¡nh EXE.

## Build

Tree build hiá»‡n dÃ¹ng AOSP 14 AP2A vÃ  Windows host-cross:

```bash
cd /media/daomai/DATA/android/AOSP_14_STOCK
source build/envsetup.sh
lunch aosp_x86_64-ap2a-eng
HOST_CROSS_OS=windows m adb adb_test -j55
```

Artifact debug `eng` cÃ³ symbol nÃªn lá»›n (khoáº£ng 62 MiB Windows). Package phÃ¡t hÃ nh pháº£i strip:

```bash
prebuilts/clang/host/linux-x86/clang-r510928/bin/llvm-strip path/to/adb.exe
prebuilts/clang/host/linux-x86/clang-r510928/bin/llvm-strip path/to/adb
```

Báº£n Ä‘Ã£ strip hiá»‡n khoáº£ng 5.6 MiB Windows vÃ  6.5 MiB Linux. KhÃ´ng strip DLL báº±ng cÃ¡ch tÃ¹y tiá»‡n náº¿u
chÆ°a kiá»ƒm tra PE exports.

## Test báº¯t buá»™c trÆ°á»›c release final

- NODE: local USB/TCP/emulator, shell, push/pull lá»›n, install, logcat, forward/reverse, scrcpy.
- LAN: listener `0.0.0.0:PORT`, direct `adb -H NODE -P PORT devices`, firewall LocalSubnet.
- MAIN: local + má»i remote trong má»™t `devices`/`devices-l`; `-s` vÃ  `-t` route Ä‘Ãºng.
- Long stream: shell/logcat/scrcpy vÃ  file lá»›n khÃ´ng cÃ³ transfer timeout.
- Node cháº¿t/reconnect, cáº¯m/rÃºt phone, hot reload config, duplicate serial vÃ  local TCP collision.
- Track devices: short, long, proto-text vÃ  proto-binary mÃ  branch há»— trá»£.

Káº¿t quáº£ quan sÃ¡t Ä‘Æ°á»£c cáº­p nháº­t trong `ADB_MULTI_SERVER_PROGRESS.md`; test pháº§n cá»©ng chÆ°a cháº¡y khÃ´ng
Ä‘Æ°á»£c ghi PASS.

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

## ä¸­æ–‡

DaoMai Multi-Server ADB å¯æŠŠæœ¬æœº Android è®¾å¤‡å’Œå¤šä¸ªå±€åŸŸç½‘ ADB èŠ‚ç‚¹ä¸Šçš„è®¾å¤‡èšåˆåˆ°ä¸€ä¸ªæ ‡å‡†
ADB æœåŠ¡ä¸­ã€‚scrcpyã€Android Studioã€è‡ªåŠ¨åŒ–å·¥å…·ã€æ–‡ä»¶ä¼ è¾“å’Œ shell å¯ä»¥ç»§ç»­ä½¿ç”¨æ™®é€šåºåˆ—å·æˆ–
è™šæ‹Ÿ transport IDï¼Œæ— éœ€ä¿®æ”¹å®¢æˆ·ç«¯åè®®ã€‚

### éƒ¨ç½²æ–¹å¼

- NODEï¼šæ”¾ç½® `adb.exe` å’Œ DLLï¼Œä¸è¦åˆ›å»º `server.txt`ã€‚
- MAINï¼šåœ¨ `adb.exe` åŒç›®å½•çš„ `server.txt` ä¸­ï¼Œæ¯è¡Œå¡«å†™ä¸€ä¸ª `HOST:PORT`ã€‚
- NODE å¯é€‰ç”¨ `remote.txt` é™åˆ¶è¯¥è¿›ç¨‹å…è®¸ç®¡ç†çš„ USBã€TCP æˆ–æ¨¡æ‹Ÿå™¨åºåˆ—å·ã€‚

```text
# MAIN çš„ server.txt
10.10.0.2:5037
10.10.0.2:5038
adb-node-01:5037
```

é¡¹ç›®æ”¯æŒè®¾å¤‡åˆ—è¡¨ç¼“å­˜ä¸Žçƒ­æ›´æ–°ã€ç¨³å®šçš„é‡å¤åºåˆ—å·åˆ«åã€è™šæ‹Ÿ transport IDã€shellã€
sync/push/pullã€é•¿æ—¶é—´äºŒè¿›åˆ¶æµã€track-devicesï¼Œä»¥åŠ scrcpy æ‰€éœ€çš„ MAIN æœ¬åœ°ç«¯å£è½¬å‘ä¸­ç»§ã€‚
è¿œç¨‹æ•°æ®æµå…·å¤‡èƒŒåŽ‹æŽ§åˆ¶ï¼Œå¹¶ä¸”æ²¡æœ‰ç©ºé—²è¶…æ—¶ã€‚

```powershell
adb kill-server
adb devices -l
adb -s SERIAL shell
scrcpy -s SERIAL
```

Windows ä¸Šçš„ MAIN/NODE ä¼šç›‘å¬é…ç½®çš„å±€åŸŸç½‘ç«¯å£ã€‚è¯·ä½¿ç”¨é¡¹ç›®æä¾›çš„ LocalSubnet é˜²ç«å¢™è§„åˆ™ï¼Œ
ä¸è¦æŠŠæœªå—ä¿æŠ¤çš„ ADB æœåŠ¡æš´éœ²åˆ°å…¬ç½‘ã€‚
