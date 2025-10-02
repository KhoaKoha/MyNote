# Telnet (Port 23)

## Connect

### Telnet

Kết nối tới máy chủ Telnet trên địa chỉ và cổng cụ thể (mặc định là 23):

```
telnet <ip> <port>
```

***

## Enumeration

### Thu thập banner

Sử dụng Netcat để lấy thông tin dịch vụ và phiên bản qua banner:

```
nc -nv <ip> 23
```

***

## Attack Vectors

### Đăng nhập ẩn danh

Kết nối tới máy chủ Telnet mà không cần mật khẩu bằng tài khoản công khai:

```
telnet <ip>
```

### Đăng nhập với thông tin mặc định

Thử các tài khoản/mật khẩu phổ biến như `admin`, `administrator`, `root`, `user`, `test`:

```
telnet <ip>
```

### Man-in-the-Middle (MitM) Spoofing

Sử dụng Metasploit để chặn thông tin đăng nhập Telnet qua tấn công MitM:

```
msf> use auxiliary/server/capture/telnet
   > set srvhost <ip>
   > set banner Hackviser Telnet Server
   > exploit
```

***

## Post-Exploitation

### Các câu Telnet thông dụng

<table><thead><tr><th width="120">Command</th><th width="331">Description</th><th width="221">Usage</th></tr></thead><tbody><tr><td><code>open</code></td><td>Kết nối đến máy chủ</td><td><code>telnet open &#x3C;IP> 23</code></td></tr><tr><td><code>close</code></td><td>Đóng kết nối hiện tại</td><td><code>telnet> close</code></td></tr><tr><td><code>quit</code></td><td>Thoát</td><td><code>telnet> quit</code></td></tr><tr><td><code>status</code></td><td>Trạng thái hiện tại của user</td><td><code>telnet> status</code></td></tr><tr><td><code>z</code></td><td>Tạm dùng telnet</td><td><code>telnet> z</code></td></tr><tr><td><code>set</code></td><td>Tùy chọn</td><td><code>telnet> set term vt100</code></td></tr><tr><td><code>unset</code></td><td>Hủy thiết lập tùy chọn</td><td><code>telnet> unset term</code></td></tr><tr><td><code>display</code></td><td>Hiển thị các cài đặt hiện tại</td><td><code>telnet> display</code></td></tr><tr><td><code>send</code></td><td>Gửi các ký tự hoặc chuối đặc biệt</td><td><code>telnet> send break</code></td></tr><tr><td><code>mode</code></td><td>Chế độ hoạt động (từng dòng, từng ký tự)</td><td><code>telnet> mode character</code></td></tr><tr><td><code>logout</code></td><td>Đăng xuất</td><td><code>telnet> logout</code></td></tr></tbody></table>