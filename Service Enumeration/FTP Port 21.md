

## **Connect**

### ftp

Kết nối tới máy chủ FTP tại `<ip>` và cổng `<port>` :

```
ftp <ip> <port>
```

### lftp

Sử dụng `lftp`, phiên bản cải tiến của `ftp` :

```
lftp <ip>
```

### URL

Truy cập máy chủ FTP qua trình duyệt với URL:

```
ftp://<user>:<pass>@<ip>
```

***

## Enumeration

### Kiểm tra thư mục mặc định và phổ biến

Tìm thư mục chứa thông tin nhạy cảm bằng `gobuster` :

```
gobuster dir -u ftp://<ip> -w <wordlist>
```

***

## Attack Vectors

### Đăng nhập ẩn danh

Sử dụng tài khoản `anonymous` với bất kỳ mật khẩu nào:

```
ftp <ip>
```

### Đăng nhập với thông tin mặc định

Thử các tài khoản/mật khẩu như `admin`, `administrator`, `root`, `ftpuser`, `test` :

```
ftp <ip>
```

### Tấn công FTP Bounce

Chuyển hướng dữ liệu qua máy chủ FTP bằng lệnh `PORT`.

#### **Thực hiện FTP Bounce**

1. Kết nối tới máy chủ FTP:

```
ftp <ip>
```

2. Chuyển hướng dữ liệu:

```
quote PORT <target_ip>,<port>
```

3. Khởi tạo truyền tệp:

```
get <file>
```

#### **Bounce bằng Metasploit**

Quét cổng mở qua máy chủ FTP:

```
msf> use auxiliary/scanner/ftp/ftp_bounce
> set RHOSTS <ftp_server>
> set RPORT <port>
> run
```

***

## Post-Exploitation

### Các câu FTP thông dụng

<table data-header-hidden><thead><tr><th width="81"></th><th width="258"></th><th width="218"></th></tr></thead><tbody><tr><td><strong>Lệnh</strong></td><td><strong>Mô tả</strong></td><td><strong>Cách dùng</strong></td></tr><tr><td><code>lcd</code></td><td>Thay đổi thư mục cục bộ</td><td><code>lcd /path/to/directory</code></td></tr><tr><td><code>cd</code></td><td>Thay đổi thư mục trên máy chủ</td><td><code>cd /path/to/directory</code></td></tr><tr><td><code>ls</code></td><td>Liệt kê các tệp trên máy chủ</td><td><code>ls</code></td></tr><tr><td><code>get</code></td><td>Tải tệp từ máy chủ về</td><td><code>get filename.txt</code></td></tr><tr><td><code>mget</code></td><td>Tải nhiều tệp từ máy chủ về</td><td><code>mget *.txt</code></td></tr><tr><td><code>put</code></td><td>Tải tệp lên máy chủ</td><td><code>put filename.txt</code></td></tr><tr><td><code>mput</code></td><td>Tải nhiều tệp lên máy chủ</td><td><code>mput *.txt</code></td></tr><tr><td><code>bin</code></td><td>Đặt chế độ truyền nhị phân</td><td><code>bin</code></td></tr><tr><td><code>ascii</code></td><td>Đặt chế độ truyền ASCII</td><td><code>ascii</code></td></tr><tr><td><code>quit</code></td><td>Thoát khỏi ứng dụng FTP</td><td><code>quit</code></td></tr></tbody></table>

### Tải xuống tất cả các file

```
wget -m ftp://<user>:<pass>@<ip>
```