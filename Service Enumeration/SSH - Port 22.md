
## Connect

### ssh

Kết nối tới máy chủ SSH với thông tin đăng nhập:

```
ssh <user>@<ip>
```

Đăng nhập bằng SSH private key sau khi thiết lập quyền:

```
chmod 600 id_rsa
ssh -i id_rsa <user>@<ip>
```

***

## Enumeration

### Kiểm tra tự động bằng ssh-audit

Phân tích chi tiết kết nối SSH (banner, thuật toán, khuyến nghị bảo mật):

```
ssh-audit <ip> 22
```

### Kiểm tra người dùng bằng Metasploit

Liệt kê tên người dùng hợp lệ trên máy chủ SSH bằng module Metasploit:

```
msf> use auxiliary/scanner/ssh/ssh_enumusers
```

***

## Attack Vectors

#### Thử thông tin đăng nhập phổ biến

Thử các tài khoản/mật khẩu phổ biến như `admin`, `root`, `user`:

```
ssh <user>@<ip>
```

***

## Post-Exploitation

### Thực thi lệnh từ xa

Chạy lệnh trên máy chủ từ xa:

```
ssh <user>@<ip> '<command>'
```

### Duy trì truy cập bằng khóa SSH

**Trên máy tấn công**

Tạo và lấy khóa công khai SSH:

```
ssh-keygen -t rsa -b 4096
cat ~/.ssh/id_rsa.pub
```

**Trên máy mục tiêu (không có thông tin đăng nhập)**

Thêm khóa công khai vào `authorized_keys` thủ công:

```
nano ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

**Trên máy mục tiêu (có thông tin đăng nhập)**

Tự động thêm khóa công khai:

```
ssh-copy-id <user>@<ip>
```

### Truyền tệp bằng SCP

**Tải xuống tệp**

Tải tệp từ máy chủ SSH về máy cục bộ:

```
scp <user>@<ip>:<file> <destination>
```

**Tải lên tệp**

Tải tệp từ máy cục bộ lên máy chủ SSH:

```
scp <file> <user>@<ip>:<destination>
```

### SSH Tunneling

{% content-ref url="misc/tunneling" %}
[tunneling](https://t1tans.gitbook.io/wiki/misc/tunneling)
{% endcontent-ref %}