# Port Scanning

### Automatic

```
autorecon <IP>
```

### Port Scanning

Speed scan

```
rustscan -a <IP>
```

Scan all port

```
nmap -Pn -T4 -p- -sS <IP>
nmap -Pn -T4 -p <port> -sSVC <IP>
```

### Find script nmap

```
locate .nse | grep <service>
ls /usr/share/nmap/scripts | grep <service>
```

{% code overflow="wrap" %}

```
# Quét UDP với phát hiện phiên bản
sudo nmap -sV -sU <RHOST>

# Quét với script 'vuln' để tìm lỗ hổng đã biết
sudo nmap -A -T4 -sC -sV --script vuln <RHOST>

# Quét với script 'discovery' và lưu output
sudo nmap -A -T4 -p- -sS -sV -oN initial --script discovery <RHOST>

# Quét với độ trễ để tránh bị phát hiện
sudo nmap -sC -sV -p- --scan-delay 5s <RHOST>

# Liệt kê người dùng Kerberos
sudo nmap $TARGET -p 88 --script krb5-enum-users --script-args krb5-enum-users.realm='<REALM>'

# Tìm các script nmap trên hệ thống
ls -lh /usr/share/nmap/scripts/*ssh*
locate -r '\.nse$' | xargs grep "categories" | grep "default\|version"
```

{% endcode %}

### **Port Scanning (Bash One-liners)**

{% code overflow="wrap" %}

```bash
# Quét nhanh tất cả các cổng bằng nc và ghi lại các cổng đóng
for p in {1..65535}; do nc -vn <IP> $p -w 1 -z & done 2> closed_ports.txt

# Quét nhanh bằng bash /dev/tcp để tìm các cổng mở
export ip=<IP>; for port in $(seq 1 65535); do (timeout 0.01 bash -c "</dev/tcp/$ip/$port") && echo "Port $port is open"; done
```

{% endcode %}
[[DNS (Port 53)]]