# LINUX 
## SET UP LINUX VÀ AUDIT
### CHECK KERNEL 
```
uname -a
```
<img width="1438" height="124" alt="image" src="https://github.com/user-attachments/assets/b339dbf0-d61d-43a7-a9fa-3ff4f1dc1dcf" />

### SETUP AUDIT

Cài đặt Audit
```
sudo apt update
sudo apt install auditd audispd-plugins -y
```

Kiểm tra trạng thái hoạt động (status)
```
sudo systemctl status auditd
```
<img width="1456" height="422" alt="image" src="https://github.com/user-attachments/assets/8b03b484-91fb-4c60-bb83-a39f3ad0cdef" />

Nếu inactive, try:
```
sudo systemctl start auditd
```

Kiểm tra log
```
sudo ls -l /var/log/audit/
```
<img width="912" height="118" alt="image" src="https://github.com/user-attachments/assets/7ade1ac6-ce54-40dc-8814-d9db101922d3" />
Thông thường sẽ có : `audit.log`

Thử một vài dòng autit log
```
sudo tail -20 /var/log/audit/audit.log
```
<img width="1962" height="332" alt="image" src="https://github.com/user-attachments/assets/ce3bbe6d-ea2f-4401-b5b3-d9922d6233ce" />

Phân tích ví dụ 2 dòng log trên
- Dòng `USER_CMD`
    ```
    type=USER_CMD msg=audit(1787727563.041:83): pid=4621 uid=1000 auid=1000 ses=3 subj=unconfined msg='cwd="/home/ubuntusiem" cmd=6C73202D6C202F7661722F6C6F672F61756469742F terminal=pts/0 res=success'
    ```
- Ý nghĩa :

| Trường                   | Giá trị            | Ý nghĩa                                                     |
| ------------------------ | ------------------ | ----------------------------------------------------------- |
| `type=USER_CMD`          | `USER_CMD`         | Ghi nhận command do user thực hiện                          |
| `pid=4621`               | `4621`             | Process ID của tiến trình thực hiện lệnh                    |
| `uid=1000`               | `1000`             | User hiện tại có UID 1000                                   |
| `auid=1000`              | `1000`             | User ban đầu đăng nhập và thực hiện hành động               |
| `ses=3`                  | `3`                | Session đăng nhập số 3                                      |
| `subj=unconfined`        | `unconfined`       | Process không bị giới hạn bởi một security context đặc biệt |
| `cwd="/home/ubuntusiem"` | `/home/ubuntusiem` | Thư mục hiện tại khi command được thực hiện                 |
| `terminal=pts/0`         | `pts/0`            | Command được thực hiện từ terminal                          |
| `res=success`            | `success`          | Command được thực hiện thành công                           |
- Giải mã trường `cmd`
  - Auditd lưu command ở dạng hexadecimal:
`6C73202D6C202F7661722F6C6F672F61756469742F`
  - Giải mã hexadecimal → ASCII:
`ls -l /var/log/audit/`
  - Tức là user đã thực hiện:
`ls -l /var/log/audit/`
--> Đây là lệnh dùng để liệt kê các file trong thư mục /var/log/audit/. 

- Dòng `CRED_REFR`:

```
type=CRED_REFR msg=audit(1787727563.045:84): pid=4621 uid=0 auid=1000 ses=3 subj=unconfined msg='op=PAM:setcred grantors=pam_permit,pam_cap acct="root" exe="/usr/bin/sudo" hostname=? addr=? terminal=/dev/pts/0 res=success'
```
    
Ý nghĩa 
    Đây là sự kiện CRED_REFR liên quan đến việc sudo thiết lập/làm mới credential.
    Các trường quan trọng:

| Trường                | Giá trị     | Ý nghĩa                                |
| --------------------- | ----------- | -------------------------------------- |
| `type=CRED_REFR`      | `CRED_REFR` | Credential được thiết lập/làm mới      |
| `pid=4621`            | `4621`      | Cùng process với dòng trước            |
| `uid=0`               | `0`         | Process đang chạy với quyền **root**   |
| `auid=1000`           | `1000`      | Người dùng ban đầu vẫn là UID `1000`   |
| `ses=3`               | `3`         | Cùng session với dòng trước            |
| `acct="root"`         | `root`      | Account mà `sudo` chuyển sang sử dụng  |
| `exe="/usr/bin/sudo"` | `sudo`      | Chương trình thực hiện việc nâng quyền |
| `terminal=/dev/pts/0` | `pts/0`     | Thực hiện từ terminal                  |
| `res=success`         | `success`   | Hoạt động thành công                   |

Điểm đáng chú ý là:
```
uid=0
auid=1000
```
Điều này không có nghĩa user ban đầu là root.

Nó có nghĩa : 
``` User ban đầu : UID 1000 --> Sudo --> Quyền thực thi : UID 0 (root)```
--> 
`auid=1000` giúp auditd giữ lại danh tính của người dùng ban đầu, ngay cả khi process đã được nâng quyền lên `root`.

## RULE
- Test rule :
```
sudo auditctl -l
```
<img width="688" height="108" alt="image" src="https://github.com/user-attachments/assets/d85e6bb1-8e7f-4a4f-bd3c-000a8778ee66" />
--> Hiện tại chưa có rule

Nạp thử 1 rule
```
sudo auditctl -a always,exit -F arch=b64 -S execve -k exec_log
sudo auditctl -l
```
<img width="1526" height="122" alt="image" src="https://github.com/user-attachments/assets/b2eafcde-5b1f-47ca-952f-1c9bccef90c1" />

## Tạo event
```
whoami
id
```
<img width="1522" height="192" alt="image" src="https://github.com/user-attachments/assets/11290907-73af-43e3-886d-d0da1aa29335" />

```
ls /tmp
cat /etc/hostname
```
<img width="1504" height="708" alt="image" src="https://github.com/user-attachments/assets/a6cfc3d8-87cb-4ccd-aef6-94fb510a547a" />

- Cụ thể log sau những lệnh vừa rồi:

`whoami`
<img width="2034" height="482" alt="image" src="https://github.com/user-attachments/assets/9fa7214f-b762-41ef-8ba9-9ffde2955c25" />

`id`
<img width="2038" height="486" alt="image" src="https://github.com/user-attachments/assets/a4f28bc2-ad2a-415c-91b4-84b9fba2912d" />

`ls /tmp`
<img width="2032" height="496" alt="image" src="https://github.com/user-attachments/assets/be1fc384-9205-475a-9a62-e9d9d40d2aa5" />

`cat /etc/hostname`
<img width="2048" height="490" alt="image" src="https://github.com/user-attachments/assets/7cdbc4f2-55db-4bc7-9d45-11ea6ea5d863" />

## Đặt câu hỏi : Tại sao một lệnh `whoami` lại sinh ra nhiều dòng log?
- Chạy `whoami` và audit log tạo ra : `PROCTITLE`, `PATH`, `PATH`, `CWD`, `EXECVE`, `SYSCALL`
- Tất cả có cùng `msg=audit(...:149)`: Số `149` là serial/event ID
--> Nghĩa là đây không phải 6 sự kiện khác nhau mà là:
<img width="502" height="416" alt="image" src="https://github.com/user-attachments/assets/9f981c77-7fe6-499c-8f40-fe5aa0da91f9" />
--> Audit subsystem thường ghi nhiều record để mô tả đầy đủ một hành động.



# Auditd Linux — Theo dõi `execve` và `SYSCALL`

## 1. Theo dõi `execve`

### 1.1. Phân tích `EXECVE`

Ví dụ với:

```bash
whoami
```

Audit ghi:

```text
type=EXECVE msg=audit(08/26/2026 00:34:21.861:149) : argc=1 a0=whoami
```

Trong đó:

- `type=EXECVE`: đây là audit record chứa thông tin về các argument của lời gọi `execve()`.
- `argc=1`: có 1 phần tử trong argument vector (`argv`).
- `a0=whoami`: argument đầu tiên.

Có thể hình dung:

```text
argv[0] = "whoami"
```

> **Lưu ý:** `argc` là số lượng phần tử trong `argv`, không nên hiểu đơn giản là "số lượng tham số sau tên command".

---

### 1.2. Ví dụ với `ls --color=auto /tmp`

Command:

```bash
ls --color=auto /tmp
```

Audit ghi:

```text
type=EXECVE msg=audit(08/26/2026 00:34:35.513:151) : argc=3 a0=ls a1=--color=auto a2=/tmp
```

Ta có:

```text
argc=3

a0 = ls
a1 = --color=auto
a2 = /tmp
```

Có thể biểu diễn dưới dạng `argv`:

```text
argv[0] = "ls"
argv[1] = "--color=auto"
argv[2] = "/tmp"
```

Trong đó:

- `a0`: phần tử đầu tiên của `argv`, thường chứa tên chương trình.
- `a1`: argument thứ nhất sau `a0`.
- `a2`: argument thứ hai sau `a0`.
- ...
- `argc`: tổng số phần tử trong `argv`.

Vì vậy:

```text
a0 → tên chương trình / argv[0]
a1 → argument thứ nhất
a2 → argument thứ hai
...
```

---

### 1.3. Ví dụ với `cat /etc/hostname`

Command:

```bash
cat /etc/hostname
```

Audit ghi:

```text
type=EXECVE msg=audit(08/26/2026 00:36:33.948:152) : argc=2 a0=cat a1=/etc/hostname
```

Tương ứng:

```text
argc=2

argv[0] = "cat"
argv[1] = "/etc/hostname"
```

Như vậy có thể khôi phục command:

```bash
cat /etc/hostname
```

---

### 1.4. `EXECVE` cho biết điều gì?

Có thể nhớ đơn giản:

```text
EXECVE
   ↓
User/process đã thực thi chương trình gì?
   ↓
Các argument được truyền vào là gì?
```

Ví dụ:

```text
EXECVE
argc=3
a0=ls
a1=--color=auto
a2=/tmp
```

cho biết process đã thực thi:

```bash
ls --color=auto /tmp
```

Đây là lý do `execve` rất quan trọng trong **Linux auditing, SIEM và forensic**: nó giúp phát hiện và truy vết việc thực thi command.

---

## 2. Theo dõi `SYSCALL`

### 2.1. `syscall` là gì?

**System call (syscall)** là cơ chế để chương trình ở **userspace** yêu cầu **kernel** thực hiện một thao tác mà chương trình không thể hoặc không nên thực hiện trực tiếp.

Có thể hình dung:

```text
Userspace
   │
   │ System Call
   ▼
Kernel
   │
   │ thực hiện thao tác
   ▼
Kết quả trả về Userspace
```

Ví dụ:

```text
Userspace
    │
    │ execve("/usr/bin/ls", ...)
    ▼
  Kernel
    │
    │ thực hiện việc nạp chương trình
    ▼
  /usr/bin/ls được thực thi
```

---

### 2.2. Một số syscall phổ biến

| Syscall | Chức năng |
|---|---|
| `execve` | Thực thi một chương trình |
| `openat` | Mở/truy cập một file |
| `read` | Đọc dữ liệu |
| `write` | Ghi dữ liệu |
| `unlink` | Xóa một file |
| `connect` | Thiết lập kết nối mạng |
| `setuid` | Thay đổi User ID |

Auditd có thể tạo rule để theo dõi từng syscall cụ thể.

Ví dụ:

```bash
sudo auditctl -a always,exit -F arch=b64 -S connect -k network_connect
```

Rule trên yêu cầu audit theo dõi syscall `connect`.

---

### 2.3. `SYSCALL` trong audit log

Trong log:

```text
type=SYSCALL
arch=x86_64
syscall=execve
success=yes
exit=0
```

Ý nghĩa:

- `type=SYSCALL`: đây là audit record mô tả một syscall.
- `arch=x86_64`: syscall được thực hiện theo kiến trúc/ABI 64-bit.
- `syscall=execve`: process đã gọi syscall `execve`.
- `success=yes`: syscall thực hiện thành công.
- `exit=0`: giá trị trả về của syscall là `0`, trong trường hợp này biểu thị thành công.

Ví dụ đầy đủ:

```text
type=SYSCALL
arch=x86_64
syscall=execve
success=yes
exit=0
pid=4892
ppid=2357
auid=ubuntusiem
uid=ubuntusiem
gid=ubuntusiem
tty=pts0
ses=3
comm=ls
exe=/usr/bin/ls
key=exec_log
```

Một số trường quan trọng:

| Trường | Ý nghĩa |
|---|---|
| `type=SYSCALL` | Record mô tả syscall |
| `arch=x86_64` | Kiến trúc/ABI của syscall |
| `syscall=execve` | Syscall được gọi |
| `success=yes` | Syscall thành công |
| `exit=0` | Giá trị trả về của syscall |
| `pid=4892` | Process thực hiện syscall |
| `ppid=2357` | Process cha |
| `auid=ubuntusiem` | Audit User ID của session |
| `uid=ubuntusiem` | UID hiện tại của process |
| `tty=pts0` | Terminal mà process gắn vào |
| `ses=3` | Audit session ID |
| `comm=ls` | Tên process |
| `exe=/usr/bin/ls` | Executable thực tế |
| `key=exec_log` | Key của audit rule đã match |

---

### 2.4. `execve` và `SYSCALL` có phải là hai thứ khác nhau không?

Cần phân biệt:

```text
execve
    ↓
là một SYSTEM CALL

SYSCALL
    ↓
là một LOẠI AUDIT RECORD dùng để mô tả syscall
```

Ví dụ:

```text
type=SYSCALL
syscall=execve
```

có nghĩa:

> Audit record loại `SYSCALL` đang mô tả việc process gọi syscall `execve()`.

Trong cùng một audit event, có thể xuất hiện nhiều record:

```text
type=SYSCALL
type=EXECVE
type=PATH
type=CWD
type=PROCTITLE
```

Chúng bổ sung thông tin cho nhau.

---

### 2.5. `execve()` thực chất làm gì?

Khi bạn gõ:

```bash
whoami
```

Shell không tự biến thành chương trình `/usr/bin/whoami`.

Quá trình đơn giản hóa:

```text
bash
 │
 │ tạo/chuẩn bị process
 ▼
process
 │
 │ execve(...)
 ▼
/usr/bin/whoami
```

`execve()` thay thế **program image** hiện tại của process bằng program image mới.

Điều này rất quan trọng:

> `execve()` không tạo ra một process mới theo nghĩa trực tiếp. Thông thường shell có thể tạo child process trước, sau đó child gọi `execve()` để biến mình thành chương trình cần chạy.

Ví dụ:

```text
bash
 │
 │ fork()
 ▼
child process
 │
 │ execve("/usr/bin/whoami", ...)
 ▼
/usr/bin/whoami
```

---

### 2.6. Dạng khái niệm của `execve()`

Có thể hình dung syscall:

```c
execve(
    pathname,
    argv,
    envp
);
```

Trong đó:

- `pathname`: đường dẫn executable cần chạy.
- `argv`: mảng các argument truyền cho chương trình.
- `envp`: mảng các biến môi trường truyền cho chương trình.

Ví dụ command:

```bash
ls --color=auto /tmp
```

có thể tương ứng về mặt khái niệm với:

```text
pathname:
    /usr/bin/ls

argv:
    argv[0] = "ls"
    argv[1] = "--color=auto"
    argv[2] = "/tmp"
```

và:

```text
argc = 3
```

Trong audit record `EXECVE`, các phần tử `argv` này được thể hiện dưới dạng:

```text
argc=3
a0=ls
a1=--color=auto
a2=/tmp
```

---

### 2.7. Mối quan hệ giữa `SYSCALL` và `EXECVE`

Có thể nhớ bằng bảng sau:

| Record | Cho biết |
|---|---|
| `SYSCALL` | Process gọi syscall nào, thành công/thất bại, PID, UID, executable... |
| `EXECVE` | Các argument được truyền vào `execve()` |
| `PATH` | Filesystem object liên quan đến event |
| `CWD` | Current Working Directory của process |
| `PROCTITLE` | Command/process title được audit ghi nhận |

Ví dụ:

```text
type=SYSCALL
syscall=execve
pid=4892
uid=ubuntusiem
auid=ubuntusiem
exe=/usr/bin/ls
key=exec_log

type=EXECVE
argc=3
a0=ls
a1=--color=auto
a2=/tmp

type=CWD
cwd=/home/ubuntusiem

type=PATH
name=/usr/bin/ls
```

Ghép các record lại, ta có thể hiểu:

```text
User/session: ubuntusiem
        ↓
Process: PID 4892
        ↓
Thực thi executable: /usr/bin/ls
        ↓
Command:
    ls --color=auto /tmp
        ↓
Working directory:
/home/ubuntusiem
        ↓
Syscall:
    execve
        ↓
Kết quả:
    success=yes
```

> **Đây mới là cách đọc auditd đúng:** không nên nhìn riêng `EXECVE` hoặc `SYSCALL`, mà ghép các record có cùng `msg=audit(...:serial)` để tái dựng **một audit event hoàn chỉnh**.

## 3. Theo dõi `PATH` và file system object

### 3.1. `PATH` là gì?

Trong auditd, `PATH` là một **loại audit record** dùng để mô tả filesystem object (file/directory) có liên quan đến một audit event.

Ví dụ:

```text
type=PATH
item=0
name=/etc/passwd
inode=123456
dev=08:05
mode=file,644
ouid=root
ogid=root
nametype=NORMAL
```

Record `PATH` giúp trả lời:

> **"File hoặc filesystem object nào có liên quan đến hành động mà auditd vừa ghi nhận?"**

Cần phân biệt:

```text
PATH record
    ↓
Thông tin về file/object trong audit event

PATH rule / watch rule
    ↓
Rule yêu cầu audit theo dõi một file/object cụ thể
```

Hai khái niệm này liên quan nhưng **không giống nhau**.

---

### 3.2. Một số trường quan trọng của `PATH`

| Trường | Ý nghĩa |
|---|---|
| `type=PATH` | Đây là audit record mô tả filesystem object |
| `item=0` | Index của object trong audit event |
| `name=/etc/passwd` | Đường dẫn của object |
| `inode=123456` | Inode của file |
| `dev=08:05` | Device chứa filesystem object |
| `mode=file,644` | Loại object và permission |
| `ouid=root` | Owner UID của object |
| `ogid=root` | Owner GID của object |
| `nametype=NORMAL` | Loại quan hệ của path với syscall |

---

### 3.3. Ví dụ từ event `execve`

Trong log:

```text
type=PATH
item=0
name=/usr/bin/whoami
inode=1443106
dev=08:05
mode=file,755
ouid=root
ogid=root
nametype=NORMAL
```

và:

```text
type=PATH
item=1
name=/lib64/ld-linux-x86-64.so.2
inode=1444000
dev=08:05
mode=file,755
ouid=root
ogid=root
nametype=NORMAL
```

Điều này cho thấy audit event của việc thực thi `whoami` có liên quan đến:

```text
/usr/bin/whoami
/lib64/ld-linux-x86-64.so.2
```

Đặc biệt:

```text
exe=/usr/bin/whoami
```

trong `SYSCALL` cho biết executable được thực thi, còn `PATH` cung cấp thông tin filesystem object liên quan đến event.

---

## 4. Theo dõi một `PATH` cụ thể bằng watch rule

Nếu mục tiêu là:

> **"Tôi muốn biết khi nào `/etc/passwd` bị thay đổi."**

Có thể sử dụng watch rule:

```bash
sudo auditctl -w /etc/passwd -p wa -k passwd_changes
```

Trong đó:

```text
-w /etc/passwd
```

→ theo dõi `/etc/passwd`.

```text
-p w
```

→ theo dõi việc ghi vào file.

```text
-p a
```

→ theo dõi thay đổi thuộc tính của file, ví dụ các thao tác như `chmod`, `chown`.

```text
-p wa
```

→ theo dõi cả:

```text
write
+
attribute change
```

```text
-k passwd_changes
```

→ gắn key `passwd_changes` để dễ tìm event sau này.

---

### 4.1. Tạo event để kiểm tra

Không nên tùy tiện sửa `/etc/passwd` trong bài lab.

Có thể tạo một file lab:

```bash
sudo touch /tmp/audit_test
sudo auditctl -w /tmp/audit_test -p wa -k test_path
```

Sau đó:

```bash
echo "audit-test" | sudo tee -a /tmp/audit_test
```

Xem event:

```bash
sudo ausearch -k test_path -i
```

Có thể thấy:

```text
type=PATH
name=/tmp/audit_test
...
```

---

## 5. Watch rule và syscall rule khác nhau như thế nào?

### Cách 1: Watch rule

```bash
sudo auditctl -w /etc/passwd -p wa -k passwd_changes
```

Cách này đơn giản:

```text
Tôi quan tâm đến file:
/etc/passwd
```

### Cách 2: Rule theo syscall/path

Ví dụ:

```bash
sudo auditctl -a always,exit \
-F arch=b64 \
-F path=/etc/passwd \
-F perm=wa \
-k passwd_changes
```

Cách này cho phép xây dựng điều kiện audit chi tiết hơn.

Có thể nhớ:

```text
watch rule
    ↓
Tập trung vào FILE

syscall rule
    ↓
Tập trung vào SYSTEM CALL

PATH record
    ↓
Mô tả FILE/OBJECT liên quan đến EVENT
```

---

## 6. Mục đích của `PATH`

Nếu `EXECVE` trả lời:

```text
"Chương trình nào được chạy?"
```

thì `PATH` giúp trả lời:

```text
"File/filesystem object nào liên quan đến hành động?"
```

Ví dụ:

```text
User
 ↓
thực hiện một hành động
 ↓
syscall
 ↓
Audit event
 ├── SYSCALL → syscall nào?
 ├── EXECVE  → command arguments?
 └── PATH    → file/object nào liên quan?
```

---

# 7. Theo dõi `USERAUTH`

## 7.1. `USERAUTH` có phải là một syscall không?

**Không.**

Đây là điểm rất quan trọng.

Không có syscall:

```text
userauth()
```

và cũng không có rule đơn giản:

```bash
auditctl -S userauth
```

Thay vào đó, `USER_AUTH`, `USER_LOGIN`, `USER_START`, `USER_END`, `CRED_ACQ`, `CRED_DISP`... là **các loại audit record liên quan đến authentication/session**, thường được tạo từ userspace/PAM thông qua audit framework.

---

## 8. PAM là gì?

**PAM (Pluggable Authentication Modules)** là framework xác thực của Linux.

Nó cung cấp các module để xử lý những hoạt động như:

```text
login
SSH authentication
sudo
su
password authentication
account locking
session management
```

Có thể hình dung:

```text
User
 ↓
SSH / login / sudo / su
 ↓
PAM
 ↓
PAM modules
 ↓
Authentication / Session
 ↓
Audit event
 ↓
auditd
 ↓
/var/log/audit/audit.log
```

Một số PAM module thường gặp:

| Module | Vai trò |
|---|---|
| `pam_unix` | Xác thực Unix truyền thống |
| `pam_faillock` | Theo dõi/xử lý login thất bại và khóa tài khoản theo policy |
| `pam_loginuid` | Thiết lập Audit Login UID (`auid`) |
| `pam_tty_audit` | Audit hoạt động bàn phím trong TTY khi được cấu hình |

---

# 9. Các authentication record quan trọng

## 9.1. `USER_AUTH`

`USER_AUTH` biểu diễn một sự kiện xác thực do userspace/PAM tạo ra.

Truy vấn:

```bash
sudo ausearch -m USER_AUTH -i
```

Mục tiêu:

```text
USER_AUTH
    ↓
Có hoạt động authentication
    ↓
Thành công hay thất bại?
```

---

## 9.2. `USER_LOGIN`

Liên quan đến sự kiện login.

Xem:

```bash
sudo ausearch -m USER_LOGIN -i
```

Tìm login thất bại:

```bash
sudo ausearch -m USER_LOGIN -sv no -i
```

Trong đó:

```text
-sv no
```

có nghĩa là tìm những event có:

```text
success = no
```

---

## 9.3. `USER_START` và `USER_END`

Hai record này liên quan đến lifecycle của session:

```text
USER_START
    ↓
Session bắt đầu

USER_END
    ↓
Session kết thúc
```

Có thể tìm:

```bash
sudo ausearch -m USER_START -i
sudo ausearch -m USER_END -i
```

---

# 10. `USER_CMD`

`USER_CMD` là audit record liên quan đến command được thực hiện thông qua cơ chế user command auditing, thường gặp trong ngữ cảnh `sudo`.

Ví dụ:

```bash
sudo id
```

Có thể xuất hiện:

```text
type=USER_CMD
...
```

Record này giúp điều tra:

```text
User nào?
    ↓
đã thực hiện command nào?
```

Tuy nhiên, không nên đồng nhất:

```text
USER_CMD
```

với:

```text
EXECVE
```

`EXECVE` là record chứa argument của syscall `execve`, còn `USER_CMD` là một loại audit record khác được sinh trong ngữ cảnh user-command auditing.

---

# 11. `auid` — User ID quan trọng khi forensic

Một trường cực kỳ quan trọng trong auditd là:

```text
auid
```

`auid` = **Audit User ID**.

Nó dùng để xác định user ban đầu của audit session.

Ví dụ:

```text
uid=root
auid=ubuntusiem
```

Có thể hiểu:

```text
Process hiện tại:
    root

User bắt đầu audit session:
    ubuntusiem
```

Điều này rất hữu ích khi user chuyển quyền:

```text
ubuntusiem
      ↓
sudo
      ↓
root
```

Sau đó:

```text
uid=root
auid=ubuntusiem
```

Nhờ `auid`, analyst vẫn có thể truy ngược:

> "Process hiện tại chạy bằng root, nhưng session ban đầu được tạo bởi user `ubuntusiem`."

---

# 12. `pam_loginuid`

Để audit session có thể gắn đúng Audit Login UID, hệ thống thường sử dụng:

```text
pam_loginuid.so
```

Kiểm tra:

```bash
grep -R "pam_loginuid" /etc/pam.d/
```

Có thể thấy:

```text
session required pam_loginuid.so
```

Luồng:

```text
User login
    ↓
PAM
    ↓
pam_loginuid
    ↓
Audit Login UID được thiết lập
    ↓
auid
    ↓
Các hành động tiếp theo
```

Đây là lý do `auid` có thể giúp truy vết user gốc ngay cả khi user sau đó dùng:

```text
su
sudo
```

để chuyển sang tài khoản khác.

---

# 13. Thực hành USERAUTH

### 13.1. Kiểm tra `pam_loginuid`

```bash
grep -R "pam_loginuid" /etc/pam.d/
```

### 13.2. Kiểm tra login

```bash
sudo ausearch -m USER_LOGIN -i
```

Authentication:

```bash
sudo ausearch -m USER_AUTH -i
```

### 13.3. Thử `sudo`

Chạy:

```bash
sudo id
```

Sau đó:

```bash
sudo ausearch -m USER_CMD -i
```

và:

```bash
sudo ausearch -m USER_AUTH -i
```

Có thể kết hợp:

```bash
sudo ausearch -k exec_log -i
```

để so sánh các loại record.

---

# 14. Theo dõi `SERVICE STOP`

## 14.1. Service stop là gì?

Ví dụ:

```bash
sudo systemctl stop ssh
```

Ý nghĩa:

```text
User yêu cầu systemd
        ↓
dừng service ssh
        ↓
systemd xử lý request
        ↓
service chuyển sang trạng thái inactive/stopped
```

Điểm quan trọng:

> **`service stop` không phải là một syscall riêng.**

Không có:

```text
-S service_stop
```

Thay vào đó phải quan sát ở nhiều lớp.

---

# 15. Lớp 1 — theo dõi `systemctl` bằng `execve`

Tạo rule:

```bash
sudo auditctl -a always,exit \
-F arch=b64 \
-F path=/usr/bin/systemctl \
-S execve \
-k service_control
```

Kiểm tra:

```bash
sudo auditctl -l
```

---

## 15.1. Tạo service-stop event

Ví dụ với service phù hợp trong lab:

```bash
sudo systemctl stop <service>
```

Nếu máy có SSH service:

```bash
sudo systemctl stop ssh
```

Sau đó:

```bash
sudo ausearch -k service_control -i
```

Có thể thấy:

```text
type=EXECVE
argc=3
a0=systemctl
a1=stop
a2=ssh
```

Có thể hiểu:

```text
a0 = systemctl
a1 = stop
a2 = ssh
```

→ User đã chạy:

```bash
systemctl stop ssh
```

---

# 16. Nhưng chỉ audit `systemctl` là chưa đủ

Đây là điểm rất quan trọng.

`systemctl` chỉ là một client dùng để gửi request tới systemd.

Một chương trình khác có thể giao tiếp với systemd thông qua IPC/API như **D-Bus** mà không cần chạy binary:

```text
/usr/bin/systemctl
```

Do đó rule:

```text
-F path=/usr/bin/systemctl
```

chỉ giúp phát hiện việc thực thi binary `systemctl`.

Nó không đảm bảo phát hiện **mọi request điều khiển systemd**.

---

# 17. Lớp 2 — `journalctl`

Systemd ghi trạng thái service vào **systemd journal**.

Xem log của một service:

```bash
sudo journalctl -u <service>
```

Ví dụ:

```bash
sudo journalctl -u ssh
```

Xem các event gần đây:

```bash
sudo journalctl --since "10 minutes ago"
```

Có thể thấy các thông tin như:

```text
Started ...
Stopped ...
Failed ...
```

Do đó:

```text
auditd
    ↓
Ai chạy command gì?
```

còn:

```text
systemd journal
    ↓
Service thực tế đã chuyển trạng thái như thế nào?
```

Hai nguồn log bổ sung cho nhau.

---

# 18. Lớp 3 — theo dõi thay đổi unit file

Một cách khác để phát hiện việc vô hiệu hóa service là theo dõi unit file.

Ví dụ:

```bash
sudo auditctl -w /etc/systemd/system/ \
-p wa \
-k systemd_unit_changes
```

Có thể theo dõi thêm:

```bash
sudo auditctl -w /lib/systemd/system/ \
-p wa \
-k systemd_unit_changes
```

Mục tiêu:

```text
Ai đó sửa unit file
        ↓
Audit phát hiện thay đổi filesystem
        ↓
PATH record
        ↓
key=systemd_unit_changes
```

Điều này quan trọng vì attacker không nhất thiết phải thực hiện:

```bash
systemctl stop ssh
```

Họ có thể thay đổi cấu hình service để service không khởi động hoặc hoạt động theo cách mong muốn.

---

# 19. Tổng hợp `SERVICE STOP`

Có thể chia thành 3 câu hỏi:

### Câu hỏi 1: Ai chạy lệnh điều khiển service?

Dùng:

```text
auditd + execve
```

Ví dụ:

```text
systemctl stop ssh
```

Audit có thể ghi:

```text
type=EXECVE
a0=systemctl
a1=stop
a2=ssh
```

### Câu hỏi 2: Service thực sự có dừng không?

Dùng:

```bash
journalctl -u ssh
```

Systemd journal cho biết trạng thái thực tế của service.

### Câu hỏi 3: Có ai sửa cấu hình service không?

Dùng audit watch:

```bash
sudo auditctl -w /etc/systemd/system/ -p wa -k systemd_unit_changes
```

và:

```bash
sudo auditctl -w /lib/systemd/system/ -p wa -k systemd_unit_changes
```

---

# 20. So sánh 5 nhóm cần nhớ

| Nhóm | Câu hỏi cần trả lời | Công cụ/record |
|---|---|---|
| `EXECVE` | User/process thực thi chương trình gì? | `EXECVE` |
| `SYSCALL` | Process gọi syscall nào, kết quả ra sao? | `SYSCALL` |
| `PATH` | File/object nào liên quan đến event? | `PATH` / watch rule |
| `USERAUTH` | User nào login/xác thực/khởi tạo session? | `USER_AUTH`, `USER_LOGIN`, `USER_START`, `USER_END`, `USER_CMD` |
| `SERVICE STOP` | Ai yêu cầu dừng service và service có thực sự dừng không? | `EXECVE` + systemd journal + unit-file audit |

---

# 21. Sơ đồ tổng thể

```text
                         LINUX AUDIT
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
     EXECVE                SYSCALL                PATH
        │                     │                     │
 "Chạy gì?"             "Syscall gì?"        "File nào?"
        │                     │                     │
        ▼                     ▼                     ▼
     argv/argc            result/PID/etc.       filesystem
        │                     │                     │
        └─────────────────────┼─────────────────────┘
                              │
                              ▼
                         AUDIT EVENT
                              │
                 ┌────────────┴────────────┐
                 │                         │
                 ▼                         ▼
             USERAUTH                SERVICE STOP
                 │                         │
           "Ai login?"              "Ai stop?"
           "Ai sudo?"               "Có thực sự stop?"
                 │                         │
                 ▼                         ▼
          PAM / auid              auditd + journal
```

---

# 22. Cách nhớ ngắn gọn

```text
EXECVE
→ Chạy chương trình gì?

SYSCALL
→ Kernel syscall nào được gọi?

PATH
→ File nào liên quan/bị tác động?

USERAUTH
→ User nào login/xác thực/session?

SERVICE STOP
→ Ai yêu cầu dừng service + service có thực sự dừng?
```

Một audit event có thể gồm nhiều record:

```text
Một audit event
    ↓
nhiều record
    ├── SYSCALL
    ├── EXECVE
    ├── PATH
    ├── CWD
    ├── PROCTITLE
    └── ...
```

Các record có cùng:

```text
msg=audit(...:serial)
```

thuộc cùng một audit event và nên được **ghép lại khi phân tích**, thay vì đọc từng record một cách độc lập.

