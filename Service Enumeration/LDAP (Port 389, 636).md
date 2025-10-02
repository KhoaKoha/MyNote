# LDAP (Port 389, 636)

## Connect

### ldapsearch

Kết nối tới máy chủ LDAP và thực hiện tìm kiếm với thông tin xác thực:

{% code overflow="wrap" %}

```
ldapsearch -x -h <ip> -b <base_dn> -D <bind_dn> -w <password> -s <search_scope> <filter>
```

{% endcode %}

### Xác thực LDAP

Xác thực với máy chủ LDAP bằng thông tin đăng nhập:

```
ldapwhoami -x -h <ip> -D <bind_dn> -w <password>
```

### windapsearch

Kết nối tới máy chủ LDAP (Active Directory) với thông tin đăng nhập để lấy thông tin chi tiết:

```
windapsearch --dc-ip <ip> -u <username> -p <password> --full
```

***

## Enumeration

### Thu thập thông tin máy chủ LDAP

Liệt kê tất cả các đối tượng từ máy chủ LDAP để lấy thông tin cơ bản:

```
ldapsearch -x -h <ip> -b "" -s base "(objectclass=*)"
```

### Liệt kê người dùng

Truy vấn LDAP để liệt kê tất cả người dùng trong một đơn vị tổ chức:

```
ldapsearch -x -h <ip> -b "ou=users,dc=example,dc=com" "(objectclass=inetOrgPerson)"
```

***

## Attack Vectors

### Đăng nhập với thông tin mặc định

Thử đăng nhập với thông tin đăng nhập mặc định hoặc yếu như `admin`/`admin`:

```
ldapwhoami -x -h <ip> -D "cn=admin,dc=example,dc=com" -w admin
```

***

## Post-Exploitation

### Kết nối bằng Apache Directory Studio

Kết nối tới máy chủ LDAP bằng giao diện đồ họa với thông tin đăng nhập:

```
./ApacheDirectoryStudio
```

{% hint style="info" %}
Bind DN or user: \<user>@\<domain>
{% endhint %}

### Trích xuất thông tin thư mục

Trích xuất thông tin nhạy cảm như thông tin đăng nhập, thành viên nhóm, và đơn vị tổ chức:

```
ldapsearch -h <ip> -p <port> -x -b "<base_dn>" "(objectclass=*)"
```

### Leo thang đặc quyền

Sử dụng cấu hình sai hoặc lỗ hổng để nâng cao quyền truy cập bằng cách sửa đổi dữ liệu LDAP:

```
ldapmodify -h <ip> -p <port> -x -D "<admin_dn>" -w "<admin_password>" -f <ldif_file>
```

### Sửa đổi dữ liệu

Thêm, xóa tài khoản người dùng, nhóm, hoặc thuộc tính trong thư mục LDAP:

```
ldapmodify -h <ip> -p <port> -x -D "<admin_dn>" -w "<admin_password>" -f <ldif_file>
```