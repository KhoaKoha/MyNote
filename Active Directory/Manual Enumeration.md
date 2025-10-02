# Manual Enumeration

## Cheatsheet Liệt kê Active Directory (AD)

### 1. Kiểm tra Thông tin Domain

**Lệnh**:

```
net view /domain
Get-NetDomain (PowerView)
```

**Mục đích**:

* Xác định tên domain hiện tại và các domain liên quan trong forest.
* Nếu phát hiện nhiều domain hoặc forest, kiểm tra mối quan hệ tin cậy (trust) để di chuyển ngang hoặc leo thang quyền qua domain trust.

### 2. Liệt kê Người dùng Domain

**Lệnh**:

```
net user /domain
Get-NetUser | select samAccountName,pwdlastset,lastlogon (PowerView)
```

**Mục đích**:

* Lấy danh sách người dùng domain và thông tin chi tiết (tên tài khoản, thời gian đổi mật khẩu, đăng nhập cuối).
* Tìm tài khoản không hoạt động (`active = No`), mật khẩu không đổi lâu (`pwdlastset` cũ), hoặc `password expires = Never`. Tài khoản thuộc Domain Admins với `lastlogon` gần đây là mục tiêu giá trị cao cho Kerberoasting hoặc pass-the-hash.

### 3. Liệt kê Nhóm Domain

**Lệnh**:

```
net group /domain
net group "<Group Name>" /domain
Get-NetGroup | select name,member (PowerView)
```

**Mục đích**:

* Xác định các nhóm domain và thành viên của chúng.
* Tìm nhóm tùy chỉnh (tạo muộn `whenCreated`), nhóm nhỏ chứa người dùng quyền cao (Domain Admins, Enterprise Admins), hoặc nhóm lồng nhau (nested groups) để leo thang quyền. Nhóm có ít thành viên nhưng quyền cao = cửa hậu tiềm năng.

### 4. Kiểm tra Quyền Admin Cục bộ

**Lệnh**:

```
Find-LocalAdminAccess (PowerView)
```

**Mục đích**:

* Kiểm tra xem người dùng hiện tại có quyền admin cục bộ trên máy nào trong domain.
* Quyền admin cục bộ trên bất kỳ máy nào là điểm xoay để dump hash/token/memory, hỗ trợ di chuyển ngang hoặc leo thang quyền.

### 5. Kiểm tra Phiên Đăng nhập và Session

**Lệnh**:

<pre><code>Get-NetSession -ComputerName &#x3C;Hostname> (PowerView)
<strong>Get-NetLoggedon -ComputerName &#x3C;Hostname> (PowerView)
</strong>PsLoggedon (Sysinternals)
</code></pre>

**Mục đích**:

* Xác định người dùng nào đang đăng nhập hoặc có phiên mở (file share) trên máy cụ thể.
* Nếu người dùng quyền cao (Domain Admins) có phiên trên máy, tấn công máy đó để dump thông tin đăng nhập hoặc token. Yêu cầu quyền admin từ xa cho `Get-NetLoggedon`.

### 6. Liệt kê Service Principal Names (SPNs)

**Lệnh**:

```
setspn -L <AccountName>
Get-NetUser -SPN | select samAccountName,serviceprincipalname (PowerView)
```

**Mục đích**:

* Liệt kê SPN liên kết với tài khoản dịch vụ (IIS, SQL, Exchange) sử dụng Kerberos.
* SPN trên tài khoản người dùng (không phải MSA) là mục tiêu Kerberoasting. SPN gắn với máy Tier 0 (DC) trên tài khoản Tier 1 là dấu hiệu sai cấu hình, dễ dẫn đến leo thang quyền.

### 7. Kiểm tra Quyền ACL của Đối tượng

**Lệnh**:

```
Get-ObjectAcl -Identity <ObjectName> (PowerView)
Convert-SidToName <SID> (PowerView)
```

**Mục đích**:

* Xem quyền ACL (Access Control List) của đối tượng (người dùng, nhóm, GPO, OU).
* Tìm quyền nguy hiểm:
  * `GenericAll`: Toàn quyền (thêm vào nhóm, đổi mật khẩu).
  * `WriteDACL`: Sửa ACL.
  * `WriteOwner`: Đổi chủ sở hữu.
  * `AddMember`: Thêm thành viên vào nhóm quyền cao.
  * Quyền trên GPO → Thêm script khởi động độc hại → RCE trên máy áp dụng GPO.
* **Mẹo OSCP**: Ưu tiên kiểm tra OU ServiceAccounts, IT, Domain Admins, GPO, Computers.

### 8. Liệt kê Domain Shares

**Lệnh**:

```
Find-DomainShare (PowerView)
dir \\<DC>\sysvol\<Domain>\Policies\
```

**Mục đích**:

* Tìm các share trong domain, đặc biệt là SYSVOL, chứa script, backup, hoặc Groups.xml.
* SYSVOL mặc định cho phép mọi người dùng domain đọc. Tìm file Groups.xml trong SYSVOL để lấy mật khẩu mã hóa (dùng `gpp-decrypt` để giải mã). Kiểm tra script hoặc file nhạy cảm bị bỏ quên.

### 9. Liệt kê Hệ điều hành Máy

**Lệnh**:

```
Get-NetComputer | select operatingsystem,dnshostname (PowerView)
```

**Mục đích**:

* Xác định phiên bản OS và tên máy trong domain.
* Tìm Windows 10 cũ hoặc tên máy gợi ý chức năng (web, files, db, backup) làm mục tiêu ưu tiên. Hệ điều hành lỗi thời dễ bị khai thác.

### 10. Liệt kê Tự động với SharpHound

**Lệnh**:

Lấy thông tin từ xa:

{% code overflow="wrap" %}

```
bloodhound-python -u <user> -p <pass> -ns <ip> -d <domain> -c All
```

{% endcode %}

Lấy thông tin nội bộ:

{% code overflow="wrap" %}

```
.\SharpHound.exe --CollectionMethod All --LdapUsername <user> --LdapPassword <pass> --domain <Domain> --domaincontroller <ip> --OutputDirectory <file>
```

{% endcode %}

{% code overflow="wrap" %}

```
. .\SharpHound.ps1
Invoke-BloodHound -CollectionMethod All --LdapUsername <user> --LdapPassword <pass> --OutputDirectory <file>
```

{% endcode %}

**Mục đích**:

* Thu thập dữ liệu AD (phiên, nhóm, ACL, quyền admin cục bộ, trust) từ máy đã xâm nhập.
* Chạy từ máy domain-joined để tăng tính ẩn. Tránh `-c All` trong môi trường thật (gây nhiều lưu lượng, dễ bị phát hiện). Xuất file `.json`/`.zip` để phân tích với BloodHound.

### 11. Phân tích Đường Tấn công với BloodHound

**Lệnh**:

* Truy vấn sẵn: `Find all Domain Admins`, `Shortest Paths to Domain Admins`.
* Đánh dấu “Owned” cho tài sản đã kiểm soát.\
  **Mục đích**:
* Trực quan hóa đường tấn công (leo thang quyền, di chuyển ngang) qua đồ thị.
* &#x20;Tìm quyền GenericAll (Đỏ), WriteDACL (Cam), WriteOwner (Vàng), AddMember (Xanh lá). Ví dụ: Người dùng có `AdminTo` trên máy có phiên của Domain Admin → Dump thông tin để chiếm domain.
