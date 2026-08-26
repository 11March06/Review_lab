# Windows Audit Policy — So sánh với Linux auditd

Windows khác Linux ở chỗ:

**Linux**
```
Rule audit
   ↓
kernel audit subsystem
   ↓
auditd
   ↓
/var/log/audit/audit.log
```

**Windows** có thể hình dung:
```
Audit Policy
     ↓
Windows Security Auditing subsystem
     ↓
Sự kiện bảo mật
     ↓
Windows Event Log
     ↓
Security.evtx
     ↓
Event Viewer / PowerShell / SIEM
```

Microsoft gọi cơ chế cấu hình này là **Windows Security Audit Policy**. `auditpol.exe` dùng để xem và thay đổi audit policy; policy có thể được cấu hình theo category/subcategory và theo Success / Failure.

## 1. Audit Policy trên Windows là gì?

Có thể hiểu đơn giản:

> Audit Policy quyết định Windows cần "quan sát" loại hoạt động nào và khi nào thì ghi log.

Ví dụ Windows ghi lại:
- User đăng nhập
- User đăng xuất
- Process được tạo
- File bị truy cập
- Account bị thay đổi
- Service được cài đặt
- Audit policy bị thay đổi

--> Phải enable audit policy tương ứng.

Ví dụ: **Audit Process Creation** được bật:
```
User chạy powershell.exe
        ↓
Windows tạo process
        ↓
Audit Policy kiểm tra:
"Process Creation có được audit không?"
        ↓
Có
        ↓
Security Event Log
        ↓
Event ID 4688
```

Microsoft xác định Audit Process Creation là policy tạo audit event khi một process được tạo/chạy.

## 2. Có 2 cách cấu hình chính

### Cách 1 — GUI

Mở: `Win + R`, gõ: `secpol.msc`

Sau đó:
```
Local Policies
    ↓
Audit Policy
```

hoặc quan trọng hơn:
```
Advanced Audit Policy Configuration
    ↓
System Audit Policies
```

Ví dụ:
```
Detailed Tracking
    └── Audit Process Creation
```

-> Có thể chọn: **Success** / **Failure**

Microsoft cũng hỗ trợ cấu hình qua Local Security Policy/Group Policy.

### Cách 2 — auditpol

Đây là thứ tương đương gần nhất với việc dùng `auditctl` trên Linux.

Kiểm tra audit policy hiện tại:
```
auditpol /get /category:*
```

Sẽ có kiểu:
```
System audit policy
Category/Subcategory                      Setting
------------------------------------------------------------
System
  Security System Extension               Success
  System Integrity                         Success
  IPsec Driver                             No Auditing

Logon/Logoff
  Logon                                    Success and Failure
  Logoff                                   Success
  Account Lockout                          Success and Failure

Detailed Tracking
  Process Creation                         Success
```

Ý nghĩa:
- **Success** → ghi khi hành động thành công.
- **Failure** → ghi khi hành động thất bại.
- **Success and Failure** → ghi cả hai.
- **No Auditing** → không audit loại hoạt động đó.

Microsoft xác định các giá trị policy theo dạng Off, Success, Failure hoặc Success+Failure.

## 3. auditpol tương đương gì với Linux?

Có thể nhớ:

| Linux | Windows |
|---|---|
| `auditctl` | `auditpol` |
| audit rule | Audit Policy |
| syscall rule | Audit subcategory |
| `-S execve` | Audit Process Creation |
| `-k exec_log` | Event ID/category + filtering |
| `ausearch` | Event Viewer / PowerShell |
| `/var/log/audit/audit.log` | Windows Security Event Log |
| `auditd` | Windows Event Log/Security Auditing infrastructure |

Nhưng **không phải mapping 1-1**. Đây là điểm quan trọng.

Windows không đơn giản là `auditctl → auditpol` mà kiến trúc và loại event khác nhau.

## 4. Quan trọng nhất: Windows không phải cứ bật Audit Policy là log vào một file text

Linux có: `/var/log/audit/audit.log`

Windows chủ yếu sử dụng Windows Event Log, trong đó Security log nằm tại:

```
C:\Windows\System32\winevt\Logs\Security.evtx
```

Có thể xem bằng **Event Viewer**. Mở: `eventvwr.msc`, sau đó:
```
Windows Logs
    ↓
Security
```

Đây là nơi sẽ thực hành quan sát.

## 5. Event ID là gì?

Đây là khái niệm cực kỳ quan trọng khi chuyển từ Linux sang Windows.

**Linux:**
```
type=EXECVE
type=SYSCALL
type=PATH
```

**Windows:**
```
Event ID 4688
Event ID 4624
Event ID 4625
...
```

Ví dụ:
- Process Creation → Event ID: 4688 → một process được tạo.
- Successful Logon → Event ID: 4624 → đăng nhập thành công.
- Failed Logon → Event ID: 4625 → đăng nhập thất bại.

Do đó có thể nhớ: Linux dùng **record type**, Windows dùng **Event ID**.

## 6. Tương ứng với execve của Linux

Trong Linux có:
```
-a always,exit -F arch=b64 -S execve -k exec_log
```

Sau đó:
```
sudo ausearch -k exec_log -m EXECVE -i
```

và thấy:
```
type=EXECVE
argc=3
a0=ls
a1=--color=auto
a2=/tmp
```

Trên Windows, ý tưởng tương ứng là **Audit Process Creation**. Sau đó Windows sinh **Event ID 4688**.

Event này có thể cung cấp thông tin như:
- New Process Name
- New Process ID
- Creator Process ID
- Subject User
- Command Line

**Command Line** đặc biệt quan trọng.

Ví dụ bạn chạy: `whoami`

Windows có thể ghi:
```
New Process Name:
C:\Windows\System32\whoami.exe

Command Line:
whoami
```

Nếu chạy: `powershell.exe -Command "Get-Process"` thì khi cấu hình phù hợp, event có thể chứa command line tương ứng.

## 7. Nhưng có một điểm rất quan trọng

Để Event 4688 chứa command line đầy đủ, chỉ bật **Audit Process Creation** chưa phải toàn bộ câu chuyện.

Windows có policy: **Include command line in process creation events**

Policy này quyết định command line có được đưa vào event process creation hay không.

Do đó:
```
Audit Process Creation
        +
Include command line in process creation events
        ↓
4688 có thông tin command line hữu ích hơn
```

Đây khá giống vấn đề vừa gặp khi học EXECVE: phải phân biệt "event được tạo" với "event chứa những field nào".

## 8. Authentication trên Windows

**Linux:**
```
USER_AUTH
USER_LOGIN
USER_START
USER_END
```

Windows thường quan sát thông qua các Security Event ID. Ví dụ:
- 4624 → Successful Logon
- 4625 → Failed Logon
- 4634 → Logoff
- 4647 → User initiated logoff

Ví dụ luồng:
```
User nhập password
       ↓
Windows authentication
       ↓
Audit Logon policy
       ↓
Security Event Log
       ↓
4624 hoặc 4625
```

Audit Logon thuộc nhóm Logon/Logoff trong Advanced Audit Policy.

## 9. 4624 quan trọng thế nào?

Ví dụ:
```
Event ID: 4624
Logon Type: 2
Account Name: tung
Workstation Name: ...
Source Network Address: ...
```

Có thể điều tra:
- Ai? → Account Name
- Đăng nhập thành công? → 4624
- Kiểu đăng nhập? → Logon Type
- Từ đâu? → Source Network Address

Đây là cách Windows forensic authentication.

## 10. Failed login

Bật **Audit Logon** với **Success and Failure**. Sau đó nhập sai password.

Windows có thể sinh **Event ID 4625**.

Từ đó có thể quan sát:
- Account Name
- Failure Reason
- Logon Type
- Source Network Address

Tương tự Linux: `ausearch -m USER_LOGIN -sv no`, nhưng cách Windows thể hiện thông tin là qua Event ID và các field trong event.

## 11. Account Management

Nếu muốn quan sát: User được tạo, User bị xóa, Password thay đổi, Group thay đổi thì dùng nhóm **Account Management**.

Ví dụ các event thường gặp:
- 4720 → User account created
- 4726 → User account deleted
- 4724 → Attempt to reset account password
- 4728 → Member added to security-enabled global group
- 4732 → Member added to security-enabled local group

Ở đây cũng cần hiểu:
```
Audit Policy
      ↓
Account Management
      ↓
event cụ thể
```

## 12. Service trên Windows

Đây cũng là phần rất giống bài service stop Linux nhưng Windows có cách riêng.

**Linux:**
```
systemctl stop ssh
        ↓
systemd
        ↓
journal
```

**Windows:**
```
Service Control Manager (SCM)
        ↓
Windows Service
```

Ví dụ: `net stop spooler` hoặc `sc stop spooler`

Có thể quan sát Windows Event Log của **Service Control Manager**.

Ví dụ **Event ID 7036** thường biểu diễn service đã chuyển trạng thái, ví dụ:
```
The Print Spooler service entered the stopped state.
```

Điểm này cần phân biệt với Security Audit Policy: không phải mọi service state event đều là Security audit event. Có thể phải quan sát thêm log System của Windows.

## 13. Vì vậy Windows cũng nên quan sát nhiều nguồn

Giống Linux: `audit.log` + `journalctl`

Windows có: **Security** + **System** + **Application**

| Mục tiêu | Windows log |
|---|---|
| Login | Security |
| Failed login | Security |
| Process creation | Security |
| Account change | Security |
| Audit policy change | Security |
| Service state | System |
| Application error | Application |

## 14. Cấu hình thu log trên Windows — nhìn tổng thể

Có thể nhớ sơ đồ này:

```
                    WINDOWS
                       │
                       ▼
              AUDIT POLICY
                       │
          ┌────────────┼────────────┐
          │            │            │
          ▼            ▼            ▼
       Logon        Process       Account
       /Logoff      Creation      Management
          │            │            │
          └────────────┼────────────┘
                       ▼
               Security Auditing
                       │
                       ▼
               Windows Event Log
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
        Security.evtx       System.evtx
             │                   │
             ▼                   ▼
       Event Viewer         Event Viewer
             │
             ▼
            SIEM
```

## 15. Cách xem cấu hình hiện tại

Trên Windows mở CMD với quyền Administrator:
```
auditpol /get /category:*
```

Đây là lệnh đầu tiên nên chạy.

Sau đó nếu muốn xem riêng process:
```
auditpol /get /subcategory:"Process Creation"
```

Authentication:
```
auditpol /get /subcategory:"Logon"
```

Sẽ biết ngay:
```
Process Creation      Success
Logon                  Success and Failure
...
```

## 16. Cấu hình Process Creation

Trên CMD Administrator:
```
auditpol /set /subcategory:"Process Creation" /success:enable
```

Kiểm tra:
```
auditpol /get /subcategory:"Process Creation"
```

Sau đó chạy: `whoami` hoặc `cmd /c whoami`

Mở `eventvwr.msc` →
```
Windows Logs
→ Security
→ tìm: 4688
```

## 17. Bật command line

Mở `secpol.msc`, vào:
```
Advanced Audit Policy Configuration
→ System Audit Policies
→ Detailed Tracking
→ Audit Process Creation
```

Bật **Success**.

Sau đó tìm policy:
```
Administrative Templates
→ System
→ Audit Process Creation
→ Include command line in process creation events
```

Bật policy này.

Mục tiêu cuối:
```
cmd.exe
   ↓
Process Creation
   ↓
4688
   ↓
Command Line
```

## 18. Cấu hình Authentication

Chạy:
```
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
```

Kiểm tra:
```
auditpol /get /subcategory:"Logon"
```

Sau đó: `Win + L` đăng nhập lại.

Vào Event Viewer → Windows Logs → Security → Tìm: **4624**

## 19. Tư duy giống bài Linux

Linux theo:
```
Rule
 ↓
Hành động
 ↓
Audit event
 ↓
Record
 ↓
Quan sát log
```

Windows cũng làm y hệt về mặt tư duy:
```
Audit Policy
 ↓
Hành động
 ↓
Windows Security Auditing
 ↓
Event ID
 ↓
Event Viewer / PowerShell
```

Ví dụ:

**Linux**
```
auditctl rule
 ↓
whoami
 ↓
execve syscall
 ↓
EXECVE + SYSCALL
 ↓
ausearch
```

**Windows**
```
Audit Process Creation
 ↓
whoami.exe
 ↓
Process Creation audit
 ↓
Event ID 4688
 ↓
Event Viewer
```

## 20. Bảng đối chiếu 

| Linux auditd | Windows Audit |
|---|---|
| `auditctl` | `auditpol` |
| `auditd` | Windows Security Auditing/Event Log infrastructure |
| `audit.log` | `Security.evtx` |
| `ausearch` | Event Viewer / PowerShell |
| `execve` | Process Creation |
| `EXECVE` | Event 4688 |
| `USER_LOGIN` | Event 4624 |
| Login thất bại | Event 4625 |
| `USER_CMD` | Không có mapping 1-1 |
| `PATH` | File/object auditing qua Object Access + SACL |
| `systemctl` | Service Control Manager / service tools |
| `journalctl` | Event Viewer, đặc biệt System log |
| `auid` | Subject/Account fields trong Security Event |

> Lưu ý: bảng này là bảng tương đồng về mục đích điều tra, không phải nói rằng Windows và Linux có cùng cơ chế nội bộ.

# 5 Trọng Tâm Khi Thực Hành Quan Sát Windows Audit Log

Tập trung vào 5 thứ sau:

## 1. Process Creation

```
Process Creation
   ↓
Event 4688
   ↓
Command line / process / user
```

## 2. Authentication

```
Authentication
   ↓
4624 / 4625
   ↓
Login thành công / thất bại
```

## 3. Object Access

```
Object Access
   ↓
Audit file access
   ↓
SACL + Audit Object Access
   ↓
Security events
```

## 4. Service

```
Service
   ↓
Service Control Manager
   ↓
System Event Log
```

## 5. Audit configuration

```
Audit configuration
   ↓
auditpol
   ↓
Audit Policy
   ↓
Event Viewer
```
