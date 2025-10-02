# SMTP (Port 25)

## Connect

### Telnet

Kết nối tới máy chủ SMTP qua Telnet trên cổng 25 để tương tác trực tiếp:

```
telnet <ip> 25
```

***

## Enumeration

### Liệt kê bản ghi MX

Sử dụng dig để tìm các máy chủ email (MX servers) của một miền:

```
dig +short mx <ip>
```

### Liệt kê người dùng

Sử dụng smtp-user-enum để kiểm tra sự tồn tại của người dùng qua phương thức VRFY:

{% code overflow="wrap" %}

```
smtp-user-enum -M VRFY -U /usr/share/wordlists/metasploit/unix_users.txt -t <ip>
```

{% endcode %}

Với nhiều máy chủ:

```
for server in $(cat smtpmachines); 
do echo "*****" $server "******"; 
smtp-user-enum -M VRFY -U userlist.txt -t $server; 
done
```

***

## Attack Vectors

### Khai thác Open Relay

Kiểm tra cấu hình Open Relay để gửi email tới bất kỳ địa chỉ nào:

```
telnet <ip> 25
MAIL FROM:<test@example.com>
RCPT TO:<test2@anotherexample.com>
DATA
Subject: Test open relay
Test message
.
QUIT
```

***

## Post-Exploitation

### Các câu SMTP thông dụng

<table><thead><tr><th width="121">Command</th><th width="377">Description</th><th width="220">Usage</th></tr></thead><tbody><tr><td>HELO</td><td>Xác định the client to the server.</td><td>HELO example.com</td></tr><tr><td>EHLO</td><td>Extended HELLO.</td><td>EHLO example.com</td></tr><tr><td>MAIL FROM:</td><td>Chỉ định địa chỉ email của người gửi</td><td>MAIL FROM: &#x3C;<a href="mailto:sender@example.com">sender@example.com</a>></td></tr><tr><td>RCPT TO:</td><td>Chỉ định địa chỉ email của người nhận</td><td>RCPT TO: &#x3C;<a href="mailto:recipient@example.com">recipient@example.com</a>></td></tr><tr><td>DATA</td><td>Cho biết sự bắt đầu của phần thân thư</td><td>DATA</td></tr><tr><td>RSET</td><td>Thiết lập lại phiên làm việc</td><td>RSET</td></tr><tr><td>NOOP</td><td>Không hoạt động; dùng để test</td><td>NOOP</td></tr><tr><td>QUIT</td><td>Kết thúc phiên làm việc</td><td>QUIT</td></tr><tr><td>VRFY</td><td>Kiểm tra xem user có tồn tại hay không</td><td>VRFY &#x3C;USER></td></tr><tr><td>EXPN</td><td>Kiểm tra user có thuộc danh sách gửi thư không</td><td>EXPN &#x3C;USER></td></tr></tbody></table>