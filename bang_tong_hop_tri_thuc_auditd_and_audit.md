# Bảng Tổng Hợp Tri Thức: auditd (Linux) & Audit Policy (Windows)

## Linux — Công cụ & Daemon

| Thuật ngữ | Giải thích |
|---|---|
| `auditd` | Daemon (tiến trình nền) nhận sự kiện từ kernel audit subsystem và ghi ra file log |
| `auditctl` | Lệnh nạp/xem rule audit vào kernel *tại runtime* (mất khi reboot nếu không lưu vào rules.d) |
| `augenrules` | Công cụ biên dịch các file `.rules` trong `/etc/audit/rules.d/` thành `/etc/audit/audit.rules` rồi nạp vào kernel — dùng để rule bền vững qua reboot |
| `ausearch` | Công cụ truy vấn log audit theo điều kiện (key, thời gian, loại sự kiện...) |
| `aureport` | Công cụ tổng hợp thống kê nhanh từ log audit (ví dụ báo cáo đăng nhập, thực thi...) |
| `audisp-remote` | Plugin chuyển tiếp log audit đến máy chủ SIEM từ xa |
| netlink | Cơ chế giao tiếp giữa kernel và userspace mà auditd dùng để nhận sự kiện từ kernel |
| LSM (Linux Security Module) | Framework hook bảo mật trong kernel (ví dụ SELinux, AppArmor) mà audit subsystem cũng móc vào để sinh sự kiện |

## Linux — File cấu hình & log

| Thuật ngữ | Giải thích |
|---|---|
| `/etc/audit/auditd.conf` | File cấu hình chính của daemon auditd (log_file, xoay vòng log, hành vi khi đầy đĩa...) |
| `/etc/audit/rules.d/*.rules` | Thư mục chứa các file rule nguồn, được `augenrules` biên dịch |
| `/var/log/audit/audit.log` | File log chính, dạng văn bản, nơi auditd ghi mọi sự kiện |
| `log_format` (ENRICHED / RAW) | ENRICHED: log dịch sẵn UID/GID/syscall sang tên dễ đọc; RAW: giữ nguyên số hiệu |
| `max_log_file` / `num_logs` | Dung lượng tối đa mỗi file log (MB) / số file giữ lại khi xoay vòng |
| `max_log_file_action` | Hành động khi file log đầy: `ROTATE` (xoay vòng), `KEEP_LOGS` (giữ tất cả), `SUSPEND` (ngừng ghi), `HALT` (tắt máy), `SYSLOG`/`EMAIL`/`EXEC` (cảnh báo/thực thi script) |
| `space_left_action` / `admin_space_left_action` | Hành động khi dung lượng đĩa còn lại chạm ngưỡng cảnh báo / ngưỡng nghiêm trọng |
| `disk_full_action` / `disk_error_action` | Hành động khi đĩa đầy hẳn hoặc gặp lỗi ghi đĩa |
| `-e 2` | Rule đặc biệt: khóa cấu hình audit, không cho sửa đổi khi đang chạy (phải reboot mới thay đổi được) |

## Linux — Cú pháp rule

| Thuật ngữ | Giải thích |
|---|---|
| `-a action,filter` | Thêm rule kiểu syscall; `action` = `always`/`never`, `filter` = điểm chèn hook (`exit` phổ biến nhất) |
| `-S syscall_name` | Chỉ định tên syscall cần theo dõi (ví dụ `execve`, `open`, `unlink`) |
| `-F field=value` | Điều kiện lọc bổ sung (arch, uid, auid, path, perm, exit=...) |
| `-k key_name` | Gắn nhãn cho rule để lọc log sau này bằng `ausearch -k` |
| `-w path` | Rule "watch" — theo dõi trực tiếp một đường dẫn file/thư mục |
| `-p` (r/w/x/a) | Loại quyền cần theo dõi trong rule watch: đọc / ghi / thực thi / đổi thuộc tính |
| `arch=b64` / `arch=b32` | Chỉ định kiến trúc 64-bit hay 32-bit (số hiệu syscall khác nhau giữa hai kiến trúc) |

## Linux — Loại bản ghi (record type) sinh ra

| Thuật ngữ | Giải thích |
|---|---|
| `SYSCALL` | Bản ghi cơ bản mô tả một lời gọi syscall (PID, UID, kết quả...) |
| `EXECVE` | Bản ghi kèm theo, chứa đầy đủ tham số dòng lệnh khi có `execve` |
| `USER_AUTH` | Kết quả xác thực (thành công/thất bại) của một cơ chế PAM cụ thể |
| `USER_LOGIN` | Sự kiện đăng nhập (qua login, sshd, gdm...) |
| `USER_START` / `USER_END` | Bắt đầu / kết thúc một phiên PAM |
| `USER_CMD` | Sinh ra khi dùng `sudo` để chạy lệnh |
| `CRED_ACQ` / `CRED_DISP` | Cấp phát / giải phóng credential (mật khẩu, token) trong phiên PAM |

## Linux — PAM (Pluggable Authentication Modules)

| Thuật ngữ | Giải thích |
|---|---|
| PAM | Hệ thống module xác thực có thể cắm/thay thế được của Linux, đứng sau các cơ chế đăng nhập |
| `pam_loginuid` | Module PAM bắt buộc để gán đúng `auid` (audit user ID = người dùng đăng nhập gốc) cho mọi hành động sau này, kể cả sau khi `su`/`sudo` |
| `pam_tty_audit` | Module PAM ghi lại nội dung nhập từ bàn phím trong phiên TTY vào audit log |
| `pam_unix` | Module PAM xác thực chuẩn dựa trên `/etc/passwd`+`/etc/shadow` |
| `pam_faillock` | Module PAM đếm và khóa tài khoản sau nhiều lần đăng nhập sai |
| `auid` | Audit User ID — ID người dùng đăng nhập gốc, giữ nguyên xuyên suốt phiên dù đổi user |

## Linux — Dịch vụ hệ thống

| Thuật ngữ | Giải thích |
|---|---|
| `systemctl` | Lệnh điều khiển dịch vụ systemd (start/stop/enable/disable) |
| `journalctl` | Lệnh xem log của systemd journal — nguồn đáng tin cậy để biết dịch vụ bị dừng, kể cả khi gọi qua D-Bus không qua binary `systemctl` |
| D-Bus | Cơ chế giao tiếp liên tiến trình mà systemd dùng nội bộ; audit subsystem không hiểu ngữ nghĩa D-Bus |
| Unit file (`/etc/systemd/system/`) | File cấu hình định nghĩa một dịch vụ systemd |

## Linux — Chuẩn tham chiếu

| Thuật ngữ | Giải thích |
|---|---|
| CIS Benchmark | Bộ khuyến nghị cấu hình bảo mật chuẩn (Center for Internet Security), thường dùng làm cơ sở viết bộ rule auditd |
| STIG | Security Technical Implementation Guide — chuẩn cấu hình bảo mật của Bộ Quốc phòng Mỹ, cũng thường dùng tham chiếu |

---

## Windows — Kiến trúc & công cụ

| Thuật ngữ | Giải thích |
|---|---|
| LSASS (Local Security Authority Subsystem Service) | Tiến trình hệ thống xử lý xác thực và sinh audit log ở tầng bảo mật |
| Security Reference Monitor | Thành phần kernel-mode kiểm tra quyền truy cập và sinh sự kiện audit tương ứng |
| `secpol.msc` | Công cụ Local Security Policy — nơi cấu hình Basic Audit Policy |
| `gpedit.msc` | Local Group Policy Editor — cấu hình chính sách cục bộ (máy đơn lẻ) |
| GPMC (Group Policy Management Console) | Công cụ quản lý Group Policy cho toàn domain |
| `auditpol` | Lệnh dòng lệnh xem/cấu hình Advanced Audit Policy — tương đương `auditctl` bên Linux |
| Basic Audit Policy | Chính sách audit cũ, 9 nhóm lớn, ít chi tiết |
| Advanced Audit Policy Configuration | Chính sách audit chi tiết (~60 subcategory), khuyến nghị dùng thay Basic |
| `gpupdate /force` | Lệnh áp dụng ngay chính sách Group Policy vừa chỉnh, không cần chờ chu kỳ refresh |

## Windows — Quyền truy cập đối tượng

| Thuật ngữ | Giải thích |
|---|---|
| SACL (System Access Control List) | Danh sách quy định *hành động nào cần được audit* trên một file/thư mục — bắt buộc phải gắn riêng, khác với auditd chỉ cần 1 rule |
| DACL (Discretionary Access Control List) | Danh sách quy định *ai được phép truy cập* (khác với SACL là để audit, DACL là để phân quyền) |
| `icacls` | Lệnh chỉnh ACL/SACL của file, dùng `/setaudit` để gắn rule audit vào file cụ thể |

## Windows — Event ID quan trọng

| Event ID | Ý nghĩa |
|---|---|
| 4688 | Process Creation — tương đương `execve` bên Linux |
| 4624 | Đăng nhập thành công |
| 4625 | Đăng nhập thất bại |
| 4776 | Credential Validation (xác thực domain) |
| 4672 | Special Logon — đăng nhập có quyền đặc biệt/quản trị |
| 4663 | Truy cập đối tượng đã được audit (theo SACL) |
| 4656 | Yêu cầu handle đến một đối tượng được audit |
| 7036 | Dịch vụ chuyển trạng thái (running/stopped) — nằm trong System log, không phải Security log |
| 7040 | Thay đổi start type của dịch vụ (ví dụ từ Automatic sang Disabled) |

## Windows — Công cụ giám sát/truy vấn bổ sung

| Thuật ngữ | Giải thích |
|---|---|
| ETW (Event Tracing for Windows) | Cơ chế trace sự kiện ở tầng thấp của Windows, gần nhất với khái niệm "theo dõi syscall" bên Linux |
| Sysmon | Công cụ miễn phí của Sysinternals (Microsoft), log chi tiết hơn nhiều so với Windows Event Log mặc định (process, network, file, registry...) |
| Sysinternals | Bộ công cụ hệ thống của Microsoft (chứa Sysmon, Process Explorer, Autoruns...) |
| `wevtutil` | Lệnh dòng lệnh quản lý Event Log (đặt kích thước, export log ra file .evtx...) |
| `Get-WinEvent` | Lệnh PowerShell truy vấn Event Log — tương đương `ausearch` bên Linux |
| WEF / WEC (Windows Event Forwarding / Collector) | Cơ chế chuyển tiếp log tập trung từ nhiều máy về một máy chủ thu thập, tương đương `audisp-remote` bên Linux |
