# Page 2

Chắc chắn rồi, đây là phần **Information Gathering** được định dạng lại bằng Markdown với các khối mã cho rõ ràng.

***

#### Information Gathering

**memcached**

* **Tham khảo:** `https://tinyurl.com/yscq43ox`
* **Tấn công:** memcrashed / 11211/UDP

**Lệnh**

```bash
# Cài đặt công cụ CLI
npm install -g memcached-cli

# Kết nối (nếu có creds)
memcached-cli <USERNAME>:<PASSWORD>@<RHOST>:11211

# Lấy thông tin thống kê qua netcat (UDP)
echo -en "\x00\x00\x00\x00\x00\x01\x00\x00stats\r\n" | nc -q1 -u 127.0.0.1 11211

# Quét bằng Nmap
sudo nmap <RHOST> -p 11211 -sU -sS --script memcached-info
```

**Ví dụ Output**

```
STAT pid 21357
STAT uptime 41557034
STAT time 1519734962
```

**Các lệnh trong memcached để lấy dữ liệu**

* `stats items`
* `stats cachedump <slab_id> <limit>`
* `get <key>` (ví dụ: `get link`, `get file`, `get user`, `get passwd`)

**NetBIOS**

```bash
nbtscan <RHOST>
nmblookup -A <RHOST>
```

**Nmap**

```bash
```

**snmpwalk**

```bash
# Quét với community 'public'
snmpwalk -c public -v1 <RHOST>
snmpwalk -v2c -c public <RHOST> .1

# Quét các OID (Object Identifier) cụ thể
snmpwalk -v2c -c public <RHOST> nsExtendObjects

# OID hữu ích:
# Lấy tên máy chủ
snmpwalk -c public -v1 <RHOST> .1.3.6.1.2.1.1.5
# Liệt kê các tiến trình đang chạy
snmpwalk -c public -v1 <RHOST> 1.3.6.1.2.1.25.4.2.1.2
# Lấy thông tin tài khoản người dùng
snmpwalk -c public -v1 <RHOST> 1.3.6.1.4.1.77.1.2.25
# Lấy thông tin cổng TCP đang mở
snmpwalk -c public -v1 <RHOST> 1.3.6.1.2.1.6.13.1.3
# Liệt kê các phần mềm đã cài đặt
snmpwalk -c public -v1 <RHOST> 1.3.6.1.2.1.25.6.3.1.2
```