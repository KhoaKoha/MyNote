# IMAP (Port 143, 993)

## Connect

### Telnet

Kết nối tới máy chủ IMAP qua Telnet để tương tác trực tiếp với các lệnh:

```
telnet <ip> <port>
```

### Email Client

Cấu hình email client như Outlook, Thunderbird, hoặc Apple Mail để kết nối IMAP với địa chỉ máy chủ, cổng, tên người dùng, và mật khẩu:

***

## Enumeration

### Liệt kê Mailbox

Liệt kê các mailbox có sẵn trên máy chủ IMAP sau khi kết nối:

```
tag LIST "" "*"
```

### Liệt kê Header Email

Truy xuất header của email từ một mailbox cụ thể:

```
FETCH <message_range> BODY[HEADER.FIELDS (FROM TO SUBJECT DATE)]
```

***

## Attack Vectors

### Đăng nhập với thông tin mặc định

Thử đăng nhập với thông tin đăng nhập mặc định hoặc yếu như `admin`/`admin`:

```
telnet <ip> 143
LOGIN <username> <password>
```

### IMAP Injection

Tiêm các lệnh độc hại vào yêu cầu IMAP để khai thác lỗ hổng hoặc truy cập trái phép:

```
telnet <ip> 143
tag SEARCH <malicious_input>
```

### Man-in-the-Middle (MitM) Attacks

Chặn và sửa đổi lưu lượng IMAP để đánh cắp hoặc thay đổi nội dung email:

```
ettercap -T -q -M arp:remote /<gateway_ip>// /<ip>//
```

***

## Post-Exploitation

### Trích xuất Email

Trích xuất thông tin nhạy cảm từ mailbox sau khi truy cập:

```
tag SELECT <mailbox_name>
tag STATUS <mailbox_name> (MESSAGES)
tag FETCH <message_range> BODY[HEADER] BODY[1]
```

{% hint style="info" %}
Ví dụ: FETCH 1:5 BODY\[TEXT] (lấy nội dung email 1 đến 5)
{% endhint %}

### Thao tác Email

Xóa, chuyển tiếp, hoặc sửa đổi email để đạt được mục tiêu tấn công:

```
tag STORE <message_range> +FLAGS \Deleted
```

### Thiết lập chuyển tiếp Email

Tạo quy tắc chuyển tiếp tự động để gửi email đến địa chỉ do kẻ tấn công kiểm soát