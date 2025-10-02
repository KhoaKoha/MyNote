# Manual Enumeration

### 1. Kiểm tra Định danh Người dùng và Hệ thống

```powershell
whoami
```

**Mục đích:**\
Xác định người dùng hiện tại và vai trò hệ thống.

* **Tên người dùng:** Ví dụ: `dave` → Có thể là tài khoản người dùng thông thường.
* **Tên máy chủ (Hostname):** Ví dụ: `clientwk220` → Máy trạm của người dùng, không phải máy chủ.
* **Vai trò hệ thống:** Các hostname như `WEB01`, `MSSQL01`, hoặc `PRINT01` cho thấy vai trò cụ thể (máy chủ web, cơ sở dữ liệu, máy chủ in).

### 2. Kiểm tra Thành viên Nhóm

```powershell
whoami /groups
```

**Mục đích:**\
Xác định quyền hạn của người dùng dựa trên thành viên nhóm.

* **Remote Desktop Users:** Người dùng có thể truy cập hệ thống qua RDP.
* **Backup Operators:** Người dùng có quyền đọc tất cả tệp, tiềm năng cho việc leo thang đặc quyền.
* **Nhóm tùy chỉnh:** Các nhóm như `Helpdesk`, `AdminTeam`, hoặc `Power Users` đáng nghi, cần điều tra thêm.

### 3. Kiểm tra Tài khoản và Nhóm Hiện có

```powershell
Get-LocalUser
Get-LocalGroup
Get-LocalGroupMember administrators
```

**Mục đích:**\
Xác định các tài khoản có giá trị cao và nhóm tùy chỉnh.

* **Tài khoản có giá trị cao:** Tìm kiếm từ khóa trong tên tài khoản:
  * `daveadmin` → Có thể là tài khoản quản trị cho `dave`.
  * `backupadmin` → Có thể có quyền đọc tệp.
  * `Administrator (disabled)` → Tài khoản quản trị mặc định, kiểm tra xem có bị kích hoạt lại không.
* **Nhóm tùy chỉnh:** Các nhóm như `adminteam` hoặc `BackupUsers` có thể được dùng để che giấu tài khoản đặc quyền hoặc tránh bị phát hiện.

### 4. Kiểm tra Phiên bản và Kiến trúc Hệ điều hành

```powershell
systeminfo
```

**Mục đích:**\
Xác định thông tin hệ điều hành để chọn khai thác phù hợp.

* **Hệ điều hành:** Ví dụ: `Windows 11 Pro`.
* **Bản build:** Ví dụ: `22621` → Kiểm tra mức vá để tìm khai thác tương thích.
* **Kiến trúc:** Ví dụ: `x64-based PC` → Sử dụng khai thác 64-bit hoặc 32-bit tùy theo.

### 5. Kiểm tra Thông tin Mạng

```powershell
ipconfig /all
route print
netstat -ano
arp -a
```

**Mục đích:**\
Lập bản đồ cấu hình mạng và xác định các cổng/dịch vụ đang mở.

* **Địa chỉ IP:** Ví dụ: `192.168.50.220` (IP tĩnh) → Có thể là máy quan trọng.
* **DNS:** Ví dụ: `8.8.8.8` → Hệ thống không thuộc miền (domain).
* **Cổng mở:**
  * `3306` → MySQL đang chạy.
  * `80, 443` → Có thể là Apache/XAMPP.
  * `3389 (ESTABLISHED)` → Phiên RDP đang hoạt động, tiềm năng sử dụng Mimikatz sau khi leo thang lên SYSTEM.

### 6. Kiểm tra Ứng dụng Đã cài đặt

{% code overflow="wrap" %}

```powershell
Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname
```

{% endcode %}

**Mục đích:**\
Xác định các ứng dụng dễ bị tấn công hoặc cấu hình sai.

* **KeePass:** Tấn công mật khẩu chính hoặc tìm tệp cơ sở dữ liệu `.kdbx`.
* **FileZilla:** Kiểm tra `recent.xml` để tìm mật khẩu dạng plaintext.
* **XAMPP:** Cấu hình mặc định của Apache/MySQL thường dễ bị khai thác.
* **SQL Server:** Kiểm tra mật khẩu mặc định của tài khoản SA hoặc thông tin xác thực bị lộ.

### 7. Kiểm tra Tiến trình Đang chạy

```powershell
Get-Process | Select ProcessName,Path
```

**Mục đích:**\
Xác định các dịch vụ đang chạy và các điểm tấn công tiềm năng.

* **Tiến trình:** `mysqld`, `httpd`, `xampp-control` → Kiểm tra tệp cấu hình, bản sao lưu, hoặc thông tin xác thực cứng.
* **Cổng bất thường:** Đối chiếu PID với `netstat -ano` để xác định tiến trình lắng nghe trên các cổng đáng nghi.

### 9. Tháo tác với các chương trình đang chạy

#### List Services Using Command Line (CMD)

List all services:

```
C:\> sc queryex type=service state=all
```

List service names only:

```
C:\> sc queryex type=service state=all | find /i "SERVICE_NAME:"
```

Search for specific service:

```
C:\> sc queryex type=service state=all | find /i "SERVICE_NAME: myService"
```

Get the status of a specific service:

```
C:\> sc query myService
```

Get a list of the running services:

```
C:\> sc queryex type=service
- or -
C:\> sc queryex type=service state=active
-or -
C:\> net start
```

Get a list of the stopped services:

```
C:\> sc queryex type=service state=inactive
```

#### List Services Using PowerShell

List all services:

```
PS C:\> Get-Service
```

Search for specific service:

```
PS C:\> Get-Service | Where-Object {$_.Name -like "*myService*"}
```

Get the status of a specific service:

```
PS C:\> Get-Service myService
```

Get a list of the running services:

```
PS C:\> Get-Service | Where-Object {$_.Status -eq "Running"}
```

Get a list of the stopped services:

```
PS C:\> Get-Service | Where-Object {$_.Status -eq "Stopped"}
```

### 8. Tìm kiếm Tệp Nhạy cảm

```powershell
# Tìm tệp cơ sở dữ liệu KeePass
Get-ChildItem -Path C:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue
# Tìm tệp cấu hình/văn bản
Get-ChildItem -Path C:\ -Include *.ini,*.txt -File -Recurse -ErrorAction SilentlyContinue
# Tìm tài liệu trong thư mục người dùng
Get-ChildItem -Path C:\Users\<target_user> -Include *.txt,*.pdf,*.xls,*.xlsx,*.doc,*.docx -File -Recurse -ErrorAction SilentlyContinue
```

**Mục đích:**\
Phát hiện thông tin xác thực hoặc dữ liệu nhạy cảm do thói quen kém của người dùng.

* **Vấn đề phổ biến:**
  * Mật khẩu lưu dưới dạng plaintext trong tệp `.txt`, `.docx`, `.pdf`.
  * Tệp cấu hình ứng dụng chứa thông tin xác thực (`.ini`, `.xml`).
  * Tái sử dụng mật khẩu giữa các người dùng hoặc dịch vụ.
* **Phần mở rộng tệp mục tiêu:**
  * **Cấu hình/Kịch bản:** `.txt`, `.ini`, `.xml`, `.bat`, `.ps1`, `.config`
  * **Tài liệu:** `.docx`, `.xlsx`, `.pdf`, `.txt` (thường dùng để lưu mật khẩu)
  * **Bảo mật/Thông tin xác thực:** `.kdbx` (KeePass), `.pfx`, `.pem`, `.crt`, `.key`

### 9. Kiểm tra Lịch sử Lệnh PowerShell (PSReadLine)

```powershell
(Get-PSReadlineOption).HistorySavePath
Get-History
```

**Mục đích:**\
Trích xuất các lệnh nhạy cảm hoặc thông tin xác thực từ lịch sử lệnh PowerShell.

* **Vị trí nhật ký:** PSReadLine (PowerShell v5, v5.1, v7) ghi lại lịch sử lệnh.
* **Chuỗi cần tìm:**
  * Liên quan đến thông tin xác thực: `Set-Secret`, `Register-SecretVault`, `ConvertTo-SecureString`, `$password = …`, `Enter-PSSession -Credential $cred`
  * Gây nhầm lẫn: `Clear-History` (người dùng nghĩ nhật ký bị xóa, nhưng PSReadLine vẫn lưu).
* **Sử dụng thông tin xác thực:** Truy cập hệ thống bằng công cụ như `evil-winrm`:

  ```bash
  evil-winrm -i <target_ip> -u <user> -p <pass>
  ```
* **Lưu ý:**
  * Kiểm tra bản ghi PowerShell (Start-Transcript) trong `$env:USERPROFILE` hoặc `C:\Users\Public\Transcripts`.
  * Ngay cả sau `Clear-History`, nhật ký PSReadLine thường vẫn còn nguyên.
