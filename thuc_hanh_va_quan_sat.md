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
