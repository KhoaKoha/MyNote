# SNMP (Port 161, 162)

## Connect

Việc tương tác với các thiết bị hỗ trợ SNMP thường được thực hiện thông qua các công cụ dòng lệnh có khả năng gửi yêu cầu SNMP đến một agent.

### snmpwalk

`snmpwalk` là một ứng dụng dòng lệnh sử dụng các yêu cầu SNMP GETNEXT để truy vấn một thực thể mạng nhằm thu thập một cây thông tin.

Đối với SNMPv1/v2c (phổ biến trong pentesting do bảo mật yếu hơn):

```
snmpwalk -c <community_string> -v1 <ip>
snmpwalk -c <community_string> -v2c <ip>
```

### snmp-check

`snmp-check` là một công cụ hữu ích để liệt kê thông tin SNMP dưới định dạng dễ đọc.

{% code overflow="wrap" %}

```
snmp-check -t <ip> -c <community_string>
```

{% endcode %}

### snmpget

`snmpget` được sử dụng để lấy giá trị của một đối tượng MIB (Management Information Base) cụ thể từ một SNMP agent.

```
snmpget -v2c -c <community_string> <ip> <OID>
```

## Enumeration

Việc liệt kê bao gồm việc khám phá các community string có thể đọc (và có thể ghi) và trích xuất thông tin giá trị qua SNMP.

### Khám phá Community Strings

Community string giống như mật khẩu cho SNMPv1 và SNMPv2c. Các string mặc định phổ biến là "public" (chỉ đọc) và "private" (đọc-ghi).

Các công cụ như `onesixtyone` hoặc các module Metasploit có thể được sử dụng để brute-force community strings.

{% code overflow="wrap" %}

```
onesixtyone -c /path/to/community_string_list.txt <ip>
```

{% endcode %}

### Liệt kê Thông tin Hệ thống

Khi tìm được community string hợp lệ, các công cụ như `snmpwalk` hoặc `snmp-check` có thể được sử dụng để liệt kê nhiều thông tin.

Sử dụng snmp-check để liệt kê toàn diện

{% code overflow="wrap" %}

```
snmp-check -t <ip> -c <community_string>
```

{% endcode %}

Duyệt các MIB phổ biến

{% code overflow="wrap" %}

```
snmpwalk -c <community_string> -v2c <ip> system # Thông tin hệ thống
snmpwalk -c <community_string> -v2c <ip> interfaces # Giao diện mạng
snmpwalk -c <community_string> -v2c <ip> ipAddrTable # Địa chỉ IP
snmpwalk -c <community_string> -v2c <ip> hrSystemUptime # Thời gian hoạt động của máy
snmpwalk -c <community_string> -v2c <ip> hrStorageTable # Thông tin lưu trữ
snmpwalk -c <community_string> -v2c <ip> hrSWRunTable # Phần mềm đang chạy
snmpwalk -c <community_string> -v1  <ip> NET-SNMP-EXTEND-MIB::nsExtendObjects
```

{% endcode %}

### Metasploit

Metasploit cung cấp các module để quét và liệt kê SNMP.

{% code overflow="wrap" %}

```
msfconsole
msf > use auxiliary/scanner/snmp/snmp_enum
msf auxiliary(scanner/snmp/snmp_enum) > set RHOSTS <target_ip>
msf auxiliary(scanner/snmp/snmp_enum) > set COMMUNITY <community_string>
msf auxiliary(scanner/snmp/snmp_enum) > run

msf > use auxiliary/scanner/snmp/snmp_enumusers
msf auxiliary(scanner/snmp/snmp_enumusers) > set RHOSTS <target_ip>
msf auxiliary(scanner/snmp/snmp_enumusers) > run

msf > use auxiliary/scanner/snmp/snmp_enumshares
msf auxiliary(scanner/snmp/snmp_enumshares) > set RHOSTS <target_ip>
msf auxiliary(scanner/snmp/snmp_enumshares) > run
```

{% endcode %}

## Attack Vectors

### Community Strings Mặc định và Phổ biến

Cấu hình sai phổ biến nhất là sử dụng các community string mặc định như "public" (chỉ đọc) và "private" (đọc-ghi). Tấn công viên thường thử các giá trị này trước.

```
snmpwalk -c public -v1 <ip>
snmpwalk -c private -v1 <ip>
```

Nếu tìm thấy "private" hoặc một community string có thể ghi khác, có thể dẫn đến việc sửa đổi thiết bị.

### Brute-forcing Community Strings

Nếu các string mặc định không hoạt động, brute-forcing là bước tiếp theo.

#### Brute-forcing với Hydra

Hydra không hỗ trợ brute-forcing SNMP trực tiếp do SNMP dựa trên UDP và không có kết nối. Các công cụ như `onesixtyone`, `snmpbrute` (từ SECFORCE), hoặc các script Nmap/Metasploit được ưu tiên hơn.

#### Brute-forcing với Nmap

{% code overflow="wrap" %}

```
nmap -sU -p 161 --script snmp-brute --script-args snmp-brute.communitiesdb=<wordlist> <ip>
```

{% endcode %}

#### Brute-forcing với Metasploit

```
msfconsole
msf > use auxiliary/scanner/snmp/snmp_login
msf auxiliary(scanner/snmp/snmp_login) > set RHOSTS <target_ip>
msf auxiliary(scanner/snmp/snmp_login) > set PASS_FILE /path/to/community_wordlist.txt
msf auxiliary(scanner/snmp/snmp_login) > run
```

### Bẻ khóa Thông tin SNMPv3

SNMPv3 an toàn hơn nhưng có thể dễ bị tấn công nếu sử dụng thông tin xác thực yếu hoặc nếu biết được engine ID. Các công cụ như `snmp-brute.py` (thuộc bộ công cụ snmpwn) hoặc các script tùy chỉnh có thể được sử dụng nếu liệt kê người dùng SNMPv3 khả thi. Việc bắt lưu lượng SNMPv3 cũng có thể cho phép thử bẻ khóa mật khẩu offline đối với thông tin xác thực được mã hóa.

### Khai thác Quyền Ghi (Set Requests)

Nếu phát hiện community string đọc-ghi (ví dụ: "private"), kẻ tấn công có thể sửa đổi cấu hình thiết bị. Điều này rất nguy hiểm. Lệnh `snmpset` được sử dụng cho việc này.

```
snmpset -v[1|2c] -c <rw_community_string> <ip> <OID> <type> <value>
```

Các hành động nguy hiểm hơn:

* Sửa đổi bảng định tuyến
* Tắt giao diện
* Tải lên/tải xuống cấu hình thiết bị (ví dụ: trên thiết bị Cisco qua các OID liên quan đến TFTP)
* Xóa log&#x20;

Cần cẩn thận vì các lệnh `snmpset` không chính xác có thể làm thiết bị không hoạt động.

### Rò rỉ Thông tin

Ngay cả với quyền truy cập chỉ đọc, SNMP có thể tiết lộ lượng lớn thông tin nhạy cảm:

* Cấu trúc mạng (bảng định tuyến, bộ đệm ARP)
* Cấu hình thiết bị (mặc dù cấu hình đầy đủ thường được chuyển qua TFTP được kích hoạt bởi SNMP)
* Tên người dùng (đặc biệt trên hệ thống Windows)
* Dịch vụ và tiến trình đang chạy
* Phiên bản phần mềm (dẫn đến việc xác định lỗ hổng)
* Chính sách bảo mật

## Post-Exploitation

### Các OID SNMP Phổ biến để Thu thập Thông tin

Một lượng lớn thông tin có thể được thu thập bằng cách truy vấn các OID cụ thể. Một số OID quan trọng bao gồm:

* **1.3.6.1.2.1.1.1.0 (sysDescr)**: Mô tả hệ thống.
* **1.3.6.1.2.1.1.5.0 (sysName)**: Tên hệ thống.
* **1.3.6.1.2.1.2.2.1.2 (ifDescr)**: Mô tả giao diện.
* **1.3.6.1.2.1.4.20.1.1 (ipAdEntAddr)**: Địa chỉ IP trên các giao diện.
* **1.3.6.1.2.1.4.22.1.3 (ipNetToMediaPhysAddress)**: Bảng ARP (IP sang MAC).
* **1.3.6.1.2.1.25.4.2.1.2 (hrSWRunName)**: Chương trình đang chạy.
* **1.3.6.1.2.1.25.6.3.1.2 (hrSWInstalledName)**: Phần mềm đã cài đặt.
* **1.3.6.1.4.1.77.1.2.25 (host.hrStorage.hrStorageTable.hrStorageEntry.hrStorageDescr)**: Mô tả lưu trữ (Windows).
* **1.3.6.1.4.1.2021.11 (UCD-SNMP-MIB::ssCpuSystem)**: Tải CPU hệ thống (Net-SNMP).

### Sửa đổi Cấu hình

Nếu community string đọc-ghi bị xâm phạm, kẻ tấn công có thể thay đổi cấu hình thiết bị, bao gồm:

* Thay đổi tên máy chủ hoặc thông tin liên lạc.
* Thêm hoặc xóa các tuyến tĩnh.
* Tắt hoặc kích hoạt giao diện.
* Sửa đổi ACLs (Danh sách Kiểm soát Truy cập).
* Kích hoạt tải lên/tải xuống cấu hình đến/từ một máy chủ TFTP do kẻ tấn công kiểm soát.

### Trích xuất Dữ liệu

Trên một số thiết bị (như Cisco IOS cũ), SNMP có thể được sử dụng để kích hoạt việc sao chép cấu hình đang chạy hoặc cấu hình khởi động đến một máy chủ TFTP. Nếu kẻ tấn công kiểm soát máy chủ TFTP, họ có thể trích xuất toàn bộ cấu hình thiết bị. Điều này thường liên quan đến việc đặt các OID cụ thể liên quan đến địa chỉ IP máy chủ TFTP, tên tệp, và khởi tạo quá trình sao chép.

Các OID ví dụ cho Cisco:

* **1.3.6.1.4.1.9.9.96.1.1.1.1.2 (ccCopyProtocol)**: Đặt thành TFTP (1).
* **1.3.6.1.4.1.9.9.96.1.1.1.1.3 (ccCopySourceFileType)**: Cấu hình đang chạy (4) hoặc cấu hình khởi động (3).
* **1.3.6.1.4.1.9.9.96.1.1.1.1.4 (ccCopyDestFileType)**: Tệp mạng (1).
* **1.3.6.1.4.1.9.9.96.1.1.1.1.5 (ccCopyServerAddress)**: Địa chỉ IP máy chủ TFTP.
* **1.3.6.1.4.1.9.9.96.1.1.1.1.6 (ccCopyFileName)**: Tên tệp trên máy chủ TFTP.
* **1.3.6.1.4.1.9.9.96.1.1.1.1.14 (ccCopyEntryRowStatus)**: Đặt thành 'active' (4) để bắt đầu.