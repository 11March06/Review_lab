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



## 1. Theo dõi `execve`
- Đây là bước quan trọng nhất :
- Mục tiêu : ```User chạy một chương trình --> Kernel tạo audit event --> `auditd` ghi lại --> `EXECVE` cho ta thấy command arguments```

- Tìm các event mà rule `exec_log` bắt được:
```
sudo ausearch -k exec_log -i
```
Để chỉ xem phần command arguments: 
```
sudo ausearch -k exec_log -m EXECVE -i
```




- Xét `EXECVE` 
```
type=EXECVE msg=audit(08/26/2026 00:34:21.861:149) : argc=1 a0=whoami
```
`argc` = số lượng argument `argc=1` : nghĩa là command có 1 argument: `a0=whoami`.

Với : `ls --color=auto /tmp` --> audit ghi: 
```
type=EXECVE msg=audit(08/26/2026 00:34:35.513:151) : argc=3 a0=ls a1=--color=auto a2=/tmp
```
Có `argc=3`, `a0=1s`, `a1=--color=auto`, `a2=/tmp`. Tức là:
    - a0 --> Chương trình
    - a1 --> argument thứ nhất
    - a2 --> argument thứ hai
    ...

Tương tự với `cat /etc/hostname`:
--> `argc=2`, `a0=cat`, `a1=/etc/hostname` 

- Mục đích của `EXECVE` : biết được `User đã thực thi gì?`

## 2. Theo dõi SYSCALL

- Định nghĩa : System call (syscall) là cơ chế để chương trình ở userspace yêu cầu kernel thực hiện một thao tác.
- Ví dụ : Userspace : `execve("/usr/bin/ls")` --> Kernel và khi đó Kernel sẽ thực thi process và trả về cho Userspace
- Các syscall phổ biến:
    - execve      → thực thi chương trình
    - openat      → mở file
    - read        → đọc dữ liệu
    - write       → ghi dữ liệu
    - unlink      → xóa file
    - connect     → kết nối network
    - setuid      → thay đổi UID

- Theo trong log: 
```
type=SYSCALL
arch=x86_64
syscall=execve
success=yes
exit=0
```
- Ý nghĩa :
    - `type=SYSCALL` : là record chính mô tả syscall
    - `arch=x86_64` : syscall được thực hiện theo ABI/kiến trúc 64-bit
    - `syscall=execve` : process gọi syscall `execve()`
    - `sucess=yes` và `exit=0` : Kernel xử lý thành công
 

- Thực chất `execve()` làm gì? Khi gõ `whoami`. Shell không trực tiếp biến thành `whoami`. Quá trình cơ bản là :
```
bash --> Tạo child process --> Child process --> execve(...) --> /usr/bin/whoami
```
--> execve() là syscall dùng để nạp một chương trình mới vào process hiện tại. Cụ thể hơn, nó có dạng khái niệm: 
    ```
    execve(
        pathname,
        argv,
        envp
    )
    ```
    Ví dụ : 
    ```
    pathname:
    /usr/bin/ls
    argv:
    argv[0] = "ls"
    argv[1] = "--color=auto"
    argv[2] = "/tmp"
    ```
