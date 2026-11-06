# Steam Unblock (zapret auto-installer)

🇻🇳 [Tiếng Việt](#tiếng-việt) · 🇬🇧 [English](#english)

---

## Tiếng Việt

Bộ script Windows giúp vào Steam **không cần VPN** khi nhà mạng chặn bằng DPI/SNI. Script tự tải [zapret (bản Flowseal)](https://github.com/Flowseal/zapret-discord-youtube), **tự dò cấu hình hợp với mạng của bạn**, cài thành service chạy cùng Windows, và tự kiểm tra lại sau mỗi lần khởi động máy.

> ⚠️ Mới được test trên mạng **FPT Telecom** (Việt Nam). Chưa đảm bảo chạy được trên mọi nhà mạng.

### Công cụ này dùng cho trường hợp nào?

- Steam client báo lỗi `-101`, Store/Community không tải được, tải game không chạy.
- Bật VPN thì vào được, tắt VPN thì không.
- Đổi DNS (1.1.1.1, 8.8.8.8) **không có tác dụng**.

Kiểm tra nhanh trong CMD (tắt VPN):

```bat
nslookup store.steampowered.com 1.1.1.1
curl -I https://store.steampowered.com --connect-timeout 10
```

Nếu `nslookup` trả về IP bình thường nhưng `curl` báo `Recv failure: Connection was reset` thì đúng là kiểu chặn theo SNI mà công cụ này xử lý. Nếu `curl` bị timeout hoàn toàn thì có thể IP bị chặn, công cụ này sẽ không giúp được.

### Cách dùng

1. Tải `SteamUnblock.zip` ở mục Releases và **giải nén** (đừng chạy trực tiếp trong file zip).
2. Bấm đúp `CAI-DAT.bat`, chọn **Yes** ở bảng UAC, nhấn `Y` để tiếp tục.
3. Đợi vài phút. Script sẽ tự thử từng cấu hình `general*.bat`, cái nào qua được Steam TLS 1.3 thì giữ lại.
4. Thoát hẳn Steam (chuột phải biểu tượng khay hệ thống → Exit) rồi mở lại.

### Script làm gì

| Bước | Mô tả |
|------|-------|
| 1 | Thêm `C:\zapret` vào loại trừ của Windows Defender (driver WinDivert hay bị báo nhầm) |
| 2 | Tải bản zapret-discord-youtube mới nhất từ GitHub về `C:\zapret` |
| 3 | Thêm các tên miền Steam vào `lists\list-general-user.txt` |
| 4 | Thử lần lượt từng `general*.bat`, test `curl --tlsv1.3` tới Steam 5 lần, cần cả 5 lần trả về HTTP 200 |
| 5 | Lấy dòng lệnh `winws` của cấu hình thắng, tạo service `zapret` tự khởi động cùng Windows |
| 6 | Tạo tác vụ `ZapretWatchdog` (quyền SYSTEM, chạy sau khi khởi động 1 phút) |

**Watchdog** chạy mỗi lần bật máy: đợi mạng, test lại Steam TLS 1.3. Nếu hỏng, nó thử các cấu hình còn lại và tạo lại service. Nếu không còn cấu hình nào dùng được, nó dừng và vô hiệu hóa service, rồi tạo `zapret_khong_hoat_dong.txt` trên Desktop.

### Các file được tạo trong `C:\zapret`

| File | Nội dung |
|------|----------|
| `zapret_hoat_dong.txt` | Cấu hình đang dùng, dòng lệnh `winws` |
| `zapret_test_log.txt` | Log lúc cài đặt |
| `zapret_watchdog_log.txt` | Log của lần watchdog gần nhất |
| `zapret_khong_hoat_dong.txt` | Báo thất bại (cũng được copy ra Desktop) |
| `zapret_service_loi.txt` | Báo lỗi tạo service (cũng được copy ra Desktop) |

Kiểm tra watchdog không cần khởi động lại máy:

```bat
schtasks /run /tn ZapretWatchdog
```

Đợi vài phút rồi mở `zapret_watchdog_log.txt`.

### Gỡ cài đặt

Bấm đúp `GO-CAI-DAT.bat`. Nó dừng và xóa service, tác vụ watchdog, driver WinDivert, và (nếu bạn chọn) xóa `C:\zapret` cùng mục loại trừ Windows Defender.

### Cấu hình

Các biến ở đầu `auto_zapret_full.bat`:

| Biến | Mặc định | Ý nghĩa |
|------|----------|---------|
| `TESTS` / `NEED_OK` | 5 / 5 | Số lần test và số lần phải ra HTTP 200 |
| `WAIT_START` | 6 | Số giây chờ zapret khởi động (tăng lên 8–10 nếu máy chậm) |
| `PATTERN` | `general*.bat` | Mẫu tên file cấu hình cần thử |
| `REMOVE_TASK_ON_FAIL` | 1 | Xóa watchdog khi hết cấu hình để thử |
| `URL` | `https://store.steampowered.com` | Địa chỉ dùng để test |

### Câu hỏi thường gặp

**Antivirus báo virus?** `WinDivert64.sys` là driver bắt gói tin mạng, thường bị báo nhầm. Script tự thêm loại trừ cho Windows Defender. Nếu dùng antivirus khác, hãy tự thêm `C:\zapret` vào loại trừ rồi chạy lại.

**Không tải được từ GitHub?** Tải thủ công bản mới nhất ở [Releases của Flowseal](https://github.com/Flowseal/zapret-discord-youtube/releases), giải nén sao cho `general.bat` nằm trực tiếp trong `C:\zapret`, rồi chạy lại `CAI-DAT.bat`.

**Store vào được nhưng một phần vẫn lỗi?** Thêm tên miền còn thiếu vào `C:\zapret\lists\list-general-user.txt` (mỗi dòng một tên miền) rồi chạy `sc stop zapret & sc start zapret` trong CMD Admin.

**Nhà mạng đổi cách chặn?** Chạy lại `CAI-DAT.bat`.

### An toàn và minh bạch

- Toàn bộ là file `.bat` dạng text, bạn có thể đọc trước khi chạy.
- Script **không chứa mã của zapret**, chỉ tải bản chính thức từ GitHub của Flowseal.
- Script cài: một service chạy quyền SYSTEM, driver WinDivert, một tác vụ chạy lúc khởi động, và một mục loại trừ Windows Defender cho `C:\zapret`. Hãy chắc chắn bạn đồng ý trước khi chạy.
- Dùng với rủi ro của bạn. Việc vượt chặn có thể vi phạm quy định của nhà mạng hoặc pháp luật tại nơi bạn sống.

---

## English

A set of Windows scripts that let you reach Steam **without a VPN** when your ISP blocks it with DPI/SNI filtering. It downloads [zapret (Flowseal build)](https://github.com/Flowseal/zapret-discord-youtube), **automatically finds a working strategy for your network**, installs it as a Windows service, and re-checks it on every boot.

> ⚠️ Only tested on **FPT Telecom** (Vietnam). No guarantee it works on every ISP.

### When does this apply?

- Steam client shows error `-101`, the Store/Community won't load, downloads don't work.
- Everything works with a VPN and fails without it.
- Changing DNS (1.1.1.1, 8.8.8.8) **doesn't help**.

Quick check in CMD (VPN off):

```bat
nslookup store.steampowered.com 1.1.1.1
curl -I https://store.steampowered.com --connect-timeout 10
```

If `nslookup` returns a normal IP but `curl` fails with `Recv failure: Connection was reset`, you are seeing the SNI-based blocking this tool handles. A full `curl` timeout suggests an IP-level block, which this tool can't fix.

### Usage

1. Download `SteamUnblock.zip` from Releases and **extract it** (don't run it from inside the zip).
2. Double-click `CAI-DAT.bat`, accept the UAC prompt, press `Y` to continue.
3. Wait a few minutes. It tries every `general*.bat` strategy and keeps the first one that passes Steam over TLS 1.3.
4. Fully exit Steam (tray icon → Exit) and reopen it.

### What it does

| Step | Description |
|------|-------------|
| 1 | Adds `C:\zapret` to the Windows Defender exclusion list (the WinDivert driver is often a false positive) |
| 2 | Downloads the latest zapret-discord-youtube release into `C:\zapret` |
| 3 | Adds Steam domains to `lists\list-general-user.txt` |
| 4 | Tries each `general*.bat`, runs `curl --tlsv1.3` against Steam 5 times; all 5 must return HTTP 200 |
| 5 | Reads the winning `winws` command line and creates a `zapret` service that starts with Windows |
| 6 | Creates a `ZapretWatchdog` scheduled task (SYSTEM, runs 1 minute after boot) |

**The watchdog** runs on every boot: it waits for the network and re-tests Steam over TLS 1.3. If that fails, it tries the remaining strategies and recreates the service. If none work, it stops and disables the service and writes `zapret_khong_hoat_dong.txt` ("zapret not working") to your Desktop.

### Files created in `C:\zapret`

| File | Contents |
|------|----------|
| `zapret_hoat_dong.txt` | Working strategy and `winws` command line |
| `zapret_test_log.txt` | Install-time log |
| `zapret_watchdog_log.txt` | Log of the most recent watchdog run |
| `zapret_khong_hoat_dong.txt` | Failure report (also copied to Desktop) |
| `zapret_service_loi.txt` | Service creation error report (also copied to Desktop) |

Test the watchdog without rebooting:

```bat
schtasks /run /tn ZapretWatchdog
```

Wait a few minutes, then open `zapret_watchdog_log.txt`.

### Uninstall

Double-click `GO-CAI-DAT.bat`. It stops and removes the service, the watchdog task and the WinDivert driver, and (if you choose) deletes `C:\zapret` and the Defender exclusion.

### Configuration

Variables at the top of `auto_zapret_full.bat`:

| Variable | Default | Meaning |
|----------|---------|---------|
| `TESTS` / `NEED_OK` | 5 / 5 | Number of probes and how many must return HTTP 200 |
| `WAIT_START` | 6 | Seconds to wait for zapret to start (raise to 8–10 on slow machines) |
| `PATTERN` | `general*.bat` | File pattern of strategies to try |
| `REMOVE_TASK_ON_FAIL` | 1 | Remove the watchdog when no strategy works |
| `URL` | `https://store.steampowered.com` | Endpoint used for probing |

### FAQ

**Antivirus flags it?** `WinDivert64.sys` is a packet-capture driver and is commonly a false positive. The installer adds a Windows Defender exclusion. With another antivirus, add `C:\zapret` to its exclusions yourself and rerun.

**Can't download from GitHub?** Download the latest build from [Flowseal's Releases](https://github.com/Flowseal/zapret-discord-youtube/releases), extract it so `general.bat` sits directly in `C:\zapret`, then rerun `CAI-DAT.bat`.

**Store loads but parts still fail?** Add the missing domains to `C:\zapret\lists\list-general-user.txt` (one per line), then run `sc stop zapret & sc start zapret` in an elevated CMD.

**ISP changed its blocking?** Run `CAI-DAT.bat` again.

### Security and transparency

- Everything is plain-text `.bat` files you can read before running.
- The scripts **contain no zapret code**; they only download the official release from Flowseal's GitHub.
- They install a SYSTEM service, the WinDivert driver, a boot-time scheduled task and a Windows Defender exclusion for `C:\zapret`. Make sure you are comfortable with that before running.
- Use at your own risk. Circumventing network filtering may violate your ISP's terms or local law.

### Credits

- [bol-van/zapret](https://github.com/bol-van/zapret), the DPI bypass engine.
- [Flowseal/zapret-discord-youtube](https://github.com/Flowseal/zapret-discord-youtube), the Windows bundle this project downloads.

### License

MIT, see `LICENSE`.
