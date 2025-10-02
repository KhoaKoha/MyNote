# Manual Enumeration

### 1. Kiểm tra Định danh Người dùng

**Lệnh**:

```
id
```

**Mục đích**:

* Kiểm tra xem tài khoản hiện tại có đặc quyền thấp (UID >= 1000) hay không.
* Nếu UID < 1000 hoặc thuộc các GID đặc biệt, nghi ngờ là tài khoản dịch vụ có đặc quyền cao.
* Nếu nhóm bao gồm `sudo`, `adm`, `lxd`, `docker`, hoặc `lpadmin`, ngay lập tức kiểm tra quyền sudo

  hoặc tìm cơ hội leo thang dựa trên container (LPE).

***

### 2. Kiểm tra Người dùng và Nhóm Hiện có

**Lệnh**:

```
cat /etc/passwd
```

**Mục đích**:

* Xác định các tài khoản có shell để tìm dữ liệu hữu ích (lịch sử lệnh, khóa SSH, thông tin xác thực).
* Tìm kiếm người dùng bất ngờ (ví dụ: `admin`, `developer`, `test`, `backup`).
* Nếu có người dùng khác, tìm kiếm:

  ```bash
  ls -lah ~/.bash_history ~/.ssh ~/.config ~/.gnupg ~/.aws ~/.git-credentials
  ```

***

### 3. Kiểm tra Tên Máy chủ

**Lệnh**:

```
hostname
```

**Mục đích**:

* Tên máy chủ có thể gợi ý vai trò hệ thống (ví dụ: `web01`, `db01`, `staging`, `prod`, `backup`).
* Máy có tên như `backup` hoặc `staging` thường có các tập lệnh sao lưu theo lịch (cron), có thể khai thác để LPE.

***

### 4. Kiểm tra Hệ điều hành và Phiên bản

**Lệnh**:

```
cat /etc/issue
cat /etc/os-release
uname -a
```

**Mục đích**:

* Xác định phiên bản kernel để tìm lỗ hổng khai thác.
* Nếu kernel < 4.13 (Ubuntu) hoặc 3.x (CentOS), thử xem các CVE.

***

### 5. Liệt kê Tiến trình Hệ thống

**Lệnh**:

```
ps aux
./pspy
```

**Mục đích**:

* Tìm các dịch vụ như Apache, MySQL, cron, backup, Python, Node.js, hoặc Java.
* Kiểm tra các tập lệnh `.sh` hoặc `.py` đang chạy.
* Nếu tập lệnh như `python /opt/script.py` chạy với quyền root, kiểm tra quyền ghi trên tập lệnh hoặc thư viện nhập.

***

### 6. Kiểm tra Địa chỉ IP

**Lệnh**:

```
ip a
```

**Mục đích**:

* Tìm mạng Docker (giao diện `veth`, `br-xxxx`) hoặc nhiều giao diện (VDI, VPN, nội bộ).
* Nếu trong container, kiểm tra:

  ```
  cat /proc/1/cgroup
  ```

  để tìm cơ hội thoát container.
* Nhiều giao diện gợi ý khả năng di chuyển ngang (lateral movement).

***

### 7. Kiểm tra Định tuyến

**Lệnh**:

```
routel
```

**Mục đích**:

* Nhiều mạng cho thấy tiềm năng di chuyển ngang.
* Kiểm tra dịch vụ gắn với giao diện nội bộ (ví dụ: `127.0.0.1`).

***

### 8. Kiểm tra Kết nối Mạng

**Lệnh**:

```
ss -anp
```

**Mục đích**:

* Ghi nhận các dịch vụ gắn với `127.0.0.1:<port>` hoặc cổng bất thường (8888, 3306, 5432, 5000, 8080, 6379).
* Dịch vụ gắn cục bộ là mục tiêu LPE nếu có thể truy cập.

***

### 9. Kiểm tra Tường lửa

**Lệnh**:

```
cat /etc/iptables/rules.v4
```

**Mục đích**:

* Tìm quy tắc cho phép cổng bất thường (ví dụ: 1999, 9001).
* Dịch vụ mở cục bộ nhưng không mở bên ngoài có thể khai thác qua lỗ hổng cục bộ.

***

### 10. Kiểm tra Công việc theo Lịch

**Lệnh**:

```
ls -lah /etc/cron*
crontab -l
```

**Mục đích**:

* Tìm tập lệnh chạy với quyền root trong thư mục người dùng có thể ghi (ví dụ: `/home/user/.scripts/backup.sh`).
* Kiểm tra quyền ghi trên tập lệnh cron hoặc thông tin xác thực trong tập lệnh.
* Nếu tập lệnh chạy mỗi phút và có thể chỉnh sửa, chèn shell ngược.

**Đường dẫn phổ biến**:

* `/etc/cron.d/`
* `/var/spool/cron/crontabs/`

***

### 11. Liệt kê Ứng dụng Đã cài đặt

**Lệnh**:

```
dpkg -l
```

**Mục đích**:

* Xác định phiên bản cũ của MySQL, Apache2, PHP, Python, Perl, Samba, v.v.
* Tìm ứng dụng chưa vá (legacy) bằng `searchsploit` hoặc cơ sở dữ liệu CVE.

***

### 12. Tìm Thư mục Có thể Ghi

**Lệnh**:

```
find / -writable -type d 2>/dev/null
```

**Mục đích**:

* Tìm thư mục có thể ghi trong `/etc`, `/opt`, `/usr/local/bin`, hoặc `/var`.
* Nếu tập lệnh có thể ghi (ví dụ: `/opt/myscript.sh`) được chạy bởi root, khai thác nó.

***

### 13. Kiểm tra Hệ thống Tệp Gắn kết

**Lệnh**:

```
cat /etc/fstab
mount
```

**Mục đích**:

* Tìm ổ đĩa chưa gắn kết hoặc cấu hình sai.
* Kiểm tra thông tin xác thực trong hệ thống tệp gắn kết.
* Nếu gắn kết đọc-ghi chứa tệp chạy bởi root, khai thác nó.

***

### 14. Kiểm tra Ổ đĩa Có sẵn

**Lệnh**:

```
lsblk
```

**Mục đích**:

* Xác định ổ đĩa chưa gắn kết hoặc tên ổ đĩa bất thường.
* Ổ đĩa sao lưu/phục hồi thường bị cấu hình sai và dễ khai thác.

***

### 15. Kiểm tra Trình điều khiển và Mô-đun Kernel

**Lệnh**:

```
lsmod
/sbin/modinfo libata
```

**Mục đích**:

* Tìm mô-đun lỗi thời hoặc không phổ biến.
* Mô-đun `modprobe` cấu hình sai có thể bị khai thác để tạo tệp giả mạo cho LPE.

***

### 16. Tìm Tệp nhị phân SUID

**Lệnh**:

```
find / -perm -u=s -type f 2>/dev/null
```

**Mục đích**:

* Tìm tệp nhị phân có cờ SUID như `find`, `nano`, `vim`, `nmap`, `perl`, `python`, hoặc `bash`.
* Kiểm tra tệp nhị phân tùy chỉnh (ví dụ: `/opt/runme`).
* Sử dụng [GTFOBins](https://gtfobins.github.io/) để tìm cách khai thác.