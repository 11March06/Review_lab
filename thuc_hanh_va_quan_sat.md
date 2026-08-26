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

### 1.1. Mục tiêu

Đây là bước quan trọng nhất trong phần thực hành `auditd`.

Mục tiêu là quan sát quá trình:

```text
User chạy một chương trình
        ↓
Shell tạo/thực thi process
        ↓
Process gọi syscall execve()
        ↓
Kernel Audit Subsystem phát hiện syscall
        ↓
Tạo Audit Event
        ↓
auditd nhận và ghi event
        ↓
/var/log/audit/audit.log
        ↓
ausearch dùng để truy vấn
```

Điểm quan trọng nhất của `execve` là record `EXECVE` giúp ta biết **chương trình nào được thực thi và các argument được truyền vào chương trình đó**.

---

### 1.2. Nạp rule theo dõi `execve`

Rule:

```bash
sudo auditctl -a always,exit -F arch=b64 -S execve -k exec_log
```

Ý nghĩa:

- `-a always,exit`: tạo audit event khi syscall kết thúc.
- `-F arch=b64`: áp dụng cho kiến trúc 64-bit.
- `-S execve`: theo dõi syscall `execve`.
- `-k exec_log`: gắn nhãn `exec_log` cho các event khớp rule.

Kiểm tra rule đã được nạp:

```bash
sudo auditctl -l
```

Có thể thấy:

```text
-a always,exit -F arch=b64 -S execve -F key=exec_log
```

---

### 1.3. Tạo một số event

Chạy một vài command:

```bash
whoami
id
ls --color=auto /tmp
cat /etc/hostname
```

Mỗi khi một chương trình được thực thi, syscall `execve()` có thể tạo ra một audit event tương ứng.

---

### 1.4. Xem các event mà rule `exec_log` bắt được

Xem toàn bộ record thuộc key:

```bash
sudo ausearch -k exec_log -i
```

Nếu chỉ muốn xem record `EXECVE`:

```bash
sudo ausearch -k exec_log -m EXECVE -i
```

Trong đó:

- `-k exec_log`: chỉ lấy event có key `exec_log`.
- `-m EXECVE`: chỉ lấy record có type `EXECVE`.
- `-i`: `interpret`, giúp `ausearch` diễn giải một số giá trị sang dạng dễ đọc hơn.

---

### 1.5. Phân tích `EXECVE`

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

### 1.6. Ví dụ với `ls --color=auto /tmp`

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

### 1.7. Ví dụ với `cat /etc/hostname`

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

### 1.8. `EXECVE` cho biết điều gì?

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
