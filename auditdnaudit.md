## Phần 1: auditd trên Linux

### 1.1 Kiến trúc tổng quan

`auditd` là daemon userspace nhận sự kiện từ **kernel audit subsystem** (Linux Auditing System, LAS) qua netlink socket. Luồng dữ liệu:

```
Kernel (syscall, LSM hooks, PAM) → audit subsystem → auditd → /var/log/audit/audit.log
```

Các thành phần chính:
- **auditd**: daemon ghi log ra đĩa (hoặc chuyển tiếp qua audisp-remote đến SIEM).
- **auditctl**: công cụ dòng lệnh để nạp/xem rule vào kernel *tại runtime*.
- **audit.rules** (thường ở `/etc/audit/rules.d/*.rules`): định nghĩa rule bền vững, được `augenrules` biên dịch và nạp lại khi khởi động hoặc khi chạy `augenrules --load`.
- **ausearch / aureport**: công cụ truy vấn và tổng hợp log.

### 1.2 Các loại rule quan trọng

#### a) Rule theo syscall (`-a` / `-A`)

Cú pháp chung:
```
-a action,filter -S syscall_name[,syscall2,...] [-F field=value] -k key_name
```

- `action`: `always` hoặc `never`
- `filter`: điểm chèn hook — `exit` (phổ biến nhất, sau khi syscall kết thúc), `entry` (đã deprecated), `task`, `user`, `exclude`
- `-S`: tên syscall (ví dụ `execve`, `open`, `unlink`, `connect`)
- `-F`: điều kiện lọc bổ sung (arch, uid, auid, path, perm...)
- `-k`: gắn nhãn (key) để dễ lọc log sau này bằng `ausearch -k key_name`

**Ví dụ rule giám sát `execve` (theo dõi thực thi chương trình):**
```bash
-a always,exit -F arch=b64 -S execve -k exec_log
-a always,exit -F arch=b32 -S execve -k exec_log
```
Giải thích: mỗi khi tiến trình gọi `execve` (tức là khi có chương trình mới được thực thi — dấu hiệu cốt lõi để phát hiện chạy binary/script lạ), kernel sinh ra bản ghi `SYSCALL` kèm `EXECVE` (chứa đầy đủ tham số dòng lệnh) gắn nhãn `exec_log`. Cần khai báo cả `arch=b64` và `arch=b32` vì số hiệu syscall khác nhau giữa kiến trúc 32-bit và 64-bit.

**Ví dụ theo dõi một `path` cụ thể (không cần biết trước inode):**
```bash
-w /etc/passwd -p wa -k passwd_changes
-w /etc/shadow -p wa -k shadow_changes
```
Đây là dạng rút gọn của rule theo dõi file (`watch`), tương đương với việc theo dõi các syscall `open/openat/creat/rename/unlink...` tác động lên đường dẫn đó. `-p` chỉ định quyền cần theo dõi: `r` (đọc), `w` (ghi), `x` (thực thi), `a` (đổi thuộc tính — chown, chmod...).

Nếu muốn theo dõi theo `path` với điều kiện phức tạp hơn (ví dụ chỉ khi truy cập bằng syscall cụ thể), dùng dạng đầy đủ:
```bash
-a always,exit -F path=/etc/passwd -F perm=wa -k passwd_direct
```

#### b) Rule liên quan xác thực người dùng (`userauth`)

auditd không có rule "userauth" riêng theo cú pháp `-a` — thay vào đó, sự kiện xác thực (đăng nhập, sudo, su, PAM) được sinh ra bởi **PAM module `pam_tty_audit` hoặc `pam_unix`/`pam_faillock`** khi các module này gọi vào audit subsystem qua `libaudit`. Các loại bản ghi tương ứng gồm:

- `USER_AUTH`: kết quả xác thực (thành công/thất bại) của một cơ chế PAM cụ thể.
- `USER_LOGIN`: sự kiện đăng nhập (qua `login`, `sshd`, `gdm`...).
- `USER_START` / `USER_END`: bắt đầu/kết thúc phiên PAM.
- `USER_CMD`: khi dùng `sudo` để chạy lệnh.
- `CRED_ACQ` / `CRED_DISP`: quá trình cấp/giải phóng credential (mật khẩu, token).

Để đảm bảo các log này được ghi, cần trong `/etc/pam.d/`:
```
session required pam_loginuid.so
```
Đây là điều kiện tiên quyết để mọi bản ghi audit có gắn đúng `auid` (audit user ID) — tức "ai thực sự đăng nhập ban đầu", ngay cả khi sau đó dùng `su`/`sudo` chuyển sang user khác. Không có `pam_loginuid`, mọi hành vi sau khi chuyển user sẽ mất dấu vết người dùng gốc.

Ngoài ra có thể thêm rule audit theo dõi trực tiếp file cấu hình xác thực:
```bash
-w /etc/pam.d/ -p wa -k pam_config_changes
-w /var/log/faillog -p wa -k login_failures
-w /var/run/faillock -p wa -k login_failures
```

#### c) Rule liên quan dừng/khởi động dịch vụ (`service stop`)

Không có rule riêng tên "service stop" — thay vào đó cần theo dõi ở hai lớp:

**Lớp 1 — theo dõi lệnh systemctl (qua execve, đã có ở trên) kết hợp filter theo đường dẫn binary:**
```bash
-a always,exit -F arch=b64 -F path=/usr/bin/systemctl -S execve -k service_control
```

**Lớp 2 — theo dõi chính socket của systemd (D-Bus) là khó vì audit subsystem hoạt động ở tầng syscall, không hiểu ngữ nghĩa D-Bus.** Cách thực dụng nhất: kết hợp
- Giám sát `execve` với tham số chứa `stop`/`disable`/`mask` (lọc ở tầng phân tích log, không phải ở auditctl, vì auditd không lọc theo nội dung argv).
- Bật log của chính `systemd` qua `journalctl -u <service>` hoặc rsyslog, vì systemd tự sinh sự kiện `Stopped <service>` trong journal — đây mới là nguồn đáng tin cậy nhất để biết dịch vụ nào bị dừng, do lệnh có thể gọi trực tiếp qua D-Bus API mà không qua binary `systemctl`.

Ví dụ rule bổ sung theo dõi thay đổi unit file (cách phổ biến để vô hiệu hóa dịch vụ bền vững):
```bash
-w /etc/systemd/system/ -p wa -k systemd_unit_changes
-w /lib/systemd/system/ -p wa -k systemd_unit_changes
```

### 1.3 Cấu hình `auditd.conf`

File chính: `/etc/audit/auditd.conf`. Các tham số quan trọng:

```ini
log_file = /var/log/audit/audit.log
log_format = ENRICHED        # ENRICHED: dịch sẵn uid/gid/syscall sang tên; RAW: giữ số
flush = INCREMENTAL_ASYNC
freq = 50                     # số bản ghi giữa mỗi lần flush ra đĩa
max_log_file = 100            # dung lượng tối đa (MB) mỗi file log
max_log_file_action = ROTATE  # ROTATE | KEEP_LOGS | IGNORE | SUSPEND | SYSLOG | EMAIL | EXEC | HALT
num_logs = 10                 # số file rotate giữ lại
space_left = 200               # (MB) ngưỡng cảnh báo dung lượng đĩa còn lại
space_left_action = SYSLOG
admin_space_left = 100
admin_space_left_action = SUSPEND   # hành vi khi gần hết dung lượng nghiêm trọng
disk_full_action = SUSPEND
disk_error_action = SUSPEND
```

**Giải thích các lựa chọn xử lý khi đầy log** (quan trọng cho hệ thống tuân thủ, ví dụ PCI-DSS/CIS benchmark):
- `SUSPEND`: dừng ghi log audit nhưng hệ thống vẫn chạy — dùng khi ưu tiên uptime.
- `HALT`: tắt hẳn hệ thống — dùng khi chính sách yêu cầu "no audit, no action" (thường thấy trong môi trường compliance nghiêm ngặt).

### 1.4 Quản lý rule bền vững qua `augenrules`

Thay vì chỉnh trực tiếp bằng `auditctl` (mất khi reboot), nên đặt rule vào `/etc/audit/rules.d/*.rules` rồi:
```bash
augenrules --load          # biên dịch tất cả file .rules thành /etc/audit/audit.rules và nạp vào kernel
systemctl restart auditd   # hoặc chỉ cần load lại rule không cần restart toàn bộ daemon
auditctl -l                # xem rule đang active trong kernel
```

Khóa rule (chống sửa đổi khi đang chạy, cần reboot mới đổi được) — thường đặt ở cuối file rule, tuân theo khuyến nghị CIS:
```bash
-e 2
```

### 1.5 Bộ rule tham chiếu thường dùng (rút gọn theo tinh thần CIS Benchmark / STIG)

```bash
## Theo dõi thay đổi thời gian hệ thống
-a always,exit -F arch=b64 -S adjtimex,settimeofday,clock_settime -k time-change
-w /etc/localtime -p wa -k time-change

## Theo dõi thay đổi danh tính người dùng/nhóm
-w /etc/group -p wa -k identity
-w /etc/passwd -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/sudoers -p wa -k identity

## Theo dõi thay đổi cấu hình mạng
-a always,exit -F arch=b64 -S sethostname,setdomainname -k network-changes
-w /etc/hosts -p wa -k network-changes

## Theo dõi truy cập trái phép (permission denied)
-a always,exit -F arch=b64 -S open,openat,creat -F exit=-EACCES -k access-denied
-a always,exit -F arch=b64 -S open,openat,creat -F exit=-EPERM -k access-denied

## Theo dõi module kernel (nạp/gỡ)
-w /sbin/insmod -p x -k modules
-w /sbin/rmmod -p x -k modules
-w /sbin/modprobe -p x -k modules
-a always,exit -F arch=b64 -S init_module,delete_module -k modules

## Theo dõi execve (đã nêu ở trên)
-a always,exit -F arch=b64 -S execve -k exec_log
-a always,exit -F arch=b32 -S execve -k exec_log

## Khóa cấu hình
-e 2
```

### 1.6 Truy vấn log

```bash
ausearch -k exec_log --start today            # xem log theo key, từ hôm nay
ausearch -k passwd_changes -i                  # -i: interpret (dịch UID/GID sang tên)
aureport --summary                              # tổng hợp thống kê nhanh
aureport -au -i                                 # báo cáo về sự kiện xác thực (authentication)
ausearch -m USER_LOGIN -sv no                   # tìm các lần đăng nhập thất bại (sv = success/no)
```

---

## Phần 2: Audit Policy trên Windows

### 2.1 Kiến trúc tổng quan

Windows dùng **Security Auditing Subsystem** (LSASS + Security Reference Monitor) sinh sự kiện, ghi vào **Security Event Log** (xem qua Event Viewer, kênh `Security`). Có hai lớp chính sách:

- **Basic Audit Policy** (cũ, 9 nhóm — qua `secpol.msc` → Local Policies → Audit Policy): thô, ít chi tiết.
- **Advanced Audit Policy Configuration** (khuyến nghị dùng, ~60 subcategory chi tiết): qua GPO hoặc `auditpol.exe`.

> Lưu ý quan trọng: nếu cả Basic và Advanced Policy cùng được cấu hình xung đột, Windows có thể bỏ qua Advanced trừ khi bật:
> `Group Policy → Security Options → "Audit: Force audit policy subcategory settings to override audit policy category settings"` = Enabled.

### 2.2 Cấu hình bằng `auditpol` (dòng lệnh, tương đương `auditctl` bên Linux)

```powershell
# Xem toàn bộ trạng thái audit hiện tại
auditpol /get /category:*

# Xem theo nhóm cụ thể (tương đương nhóm "Logon/Logoff")
auditpol /get /category:"Logon/Logoff"

# Bật audit thành công + thất bại cho một subcategory
auditpol /set /subcategory:"Logon" /success:enable /failure:enable

# Bật audit tạo/xóa tiến trình (tương đương execve bên Linux)
auditpol /set /subcategory:"Process Creation" /success:enable /failure:enable

# Bật audit dừng/khởi động dịch vụ (tương đương "service stop" bên Linux)
auditpol /set /subcategory:"Security System Extension" /success:enable /failure:enable
auditpol /set /subcategory:"System Integrity" /success:enable /failure:enable

# Bật audit truy cập file/thư mục cụ thể (tương đương watch -p rwxa)
auditpol /set /subcategory:"File System" /success:enable /failure:enable
```

Bảng ánh xạ khái niệm sang phía Windows:

| Khái niệm Linux (auditd) | Tương đương Windows |
|---|---|
| `execve` (thực thi chương trình) | Subcategory **"Process Creation"** (Event ID **4688**) — cần bật thêm `Include command line in process creation events` (GPO: Administrative Templates → System → Audit Process Creation) để log cả tham số dòng lệnh, tương tự trường `EXECVE` của auditd |
| `syscall` theo dõi tổng quát | Không có khái niệm tương đương trực tiếp (Windows không expose audit ở tầng syscall cho user); gần nhất là **ETW (Event Tracing for Windows)** hoặc **Sysmon** (công cụ của Sysinternals, log chi tiết hơn Windows Event Log mặc định) |
| `-w /path -p wa` (theo dõi file) | Bật **Object Access → File System**, sau đó phải gắn **SACL** (System Access Control List) lên từng file/thư mục cụ thể qua `icacls` hoặc Explorer → Properties → Security → Advanced → Auditing — sinh Event ID **4663/4656** |
| `userauth` (PAM) | Subcategory **"Logon"** (Event ID **4624** thành công, **4625** thất bại), **"Credential Validation"** (4776), **"Special Logon"** (4672 — đăng nhập có quyền đặc biệt) |
| dừng/khởi động dịch vụ | Subcategory **"Security System Extension"**, **"System Integrity"**; hoặc theo dõi trực tiếp qua **System Event Log**, Event ID **7036** (service entered running/stopped state), **7040** (đổi start type) |

### 2.3 Gắn SACL cho file/thư mục cụ thể (bắt buộc để có log truy cập file — không tự động chỉ vì bật category)

```powershell
# Gắn audit rule cho một file, ghi log khi có ghi/xóa bởi Everyone
icacls "C:\Windows\System32\config\SAM" /setaudit Everyone:(WD,DC) /T
```
Giải thích: `WD` = write data, `DC` = delete child — đây là các quyền cụ thể cần audit, tương tự `-p wa` bên auditd. Nếu chỉ bật category "File System" mà không gắn SACL vào file, sẽ **không có log nào được sinh ra** — đây là khác biệt lớn nhất so với auditd (nơi rule `-w` tự đủ để theo dõi, không cần bước 2 riêng).

### 2.4 Cấu hình qua Group Policy (khuyến nghị cho môi trường domain)

```
gpedit.msc (hoặc GPMC cho domain)
→ Computer Configuration
→ Windows Settings
→ Security Settings
→ Advanced Audit Policy Configuration
→ Audit Policies
   → Logon/Logoff → Audit Logon (Success, Failure)
   → Detailed Tracking → Audit Process Creation (Success)
   → Object Access → Audit File System (Success, Failure)
   → System → Audit Security State Change / Audit System Integrity
```
Sau khi chỉnh, chạy `gpupdate /force` để áp dụng ngay, hoặc đợi chu kỳ refresh GPO mặc định (90 phút ± 30 phút jitter).

### 2.5 Cấu hình lưu trữ/xoay vòng log (tương đương `max_log_file`, `num_logs` bên auditd)

```powershell
# Đặt kích thước tối đa Security log (đơn vị KB)
wevtutil sl Security /ms:1073741824   # 1 GB

# Đặt hành vi khi đầy: overwrite cũ nhất, hoặc archive rồi overwrite, hoặc không overwrite (cần dọn thủ công)
wevtutil sl Security /rt:false        # false = không tự động overwrite, cần archive thủ công
```
Hoặc qua GPO: `Computer Configuration → Windows Settings → Security Settings → Event Log`.

### 2.6 Truy vấn log (tương đương `ausearch`/`aureport`)

```powershell
# Xem 20 sự kiện đăng nhập thất bại gần nhất
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4625} -MaxEvents 20

# Xem sự kiện tạo tiến trình kèm dòng lệnh
Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4688} -MaxEvents 20 |
  Select-Object -ExpandProperty Message

# Dùng wevtutil để export toàn bộ log Security ra file (phục vụ chuyển tiếp SIEM)
wevtutil epl Security C:\export\security.evtx
```

---

## Phần 3: Tổng hợp — cách hai hệ thống "thu log" khác nhau về bản chất

| Tiêu chí | Linux auditd | Windows Audit Policy |
|---|---|---|
| Điểm chèn hook | Kernel syscall table + LSM (netlink) | LSASS / Security Reference Monitor (kernel-mode, nhưng qua API riêng của Windows, không phải syscall table trực tiếp) |
| Đơn vị cấu hình | Rule (`-a`, `-w`) nạp bằng `auditctl`/`augenrules` | Subcategory bật/tắt bằng `auditpol`, cộng SACL cho từng đối tượng |
| Theo dõi file cụ thể | Chỉ cần 1 rule `-w path -p perm` | Cần bật category **và** gắn SACL riêng cho từng file/thư mục |
| Nơi lưu log | File phẳng `/var/log/audit/audit.log` (text, có thể ENRICHED) | Binary `.evtx`, quản lý qua Event Log service |
| Công cụ truy vấn | `ausearch`, `aureport` | `Get-WinEvent`, Event Viewer, `wevtutil` |
| Gắn nhãn để lọc sau này | `-k key_name` | Không có "key" tương đương; lọc theo Event ID + subcategory |
| Chuyển tiếp log tập trung | `audisp-remote` / rsyslog / journald forward | Windows Event Forwarding (WEF/WEC), hoặc agent SIEM (Splunk UF, Wazuh, Sysmon + WEF) |

---

*Tài liệu này mang tính tham khảo kỹ thuật, tổng hợp theo tài liệu chính thức của Red Hat/CentOS (man auditd.conf, man auditctl) và Microsoft Learn (Advanced security audit policy settings). Với môi trường production, nên đối chiếu thêm với CIS Benchmark tương ứng phiên bản hệ điều hành đang dùng trước khi áp dụng.*
