---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/ad-attack/domain-enumeration/ad-module
---

# AD-Module

## Import-Module

```powershell
Import-Module C:\AD\Tools\ADModule-master\Microsoft.ActiveDirectory.Management.dll
Import-Module C:\AD\Tools\ADModule-master\ActiveDirectory\ActiveDirectory.psd1 

PS C:\> Import-Module .\Microsoft.ActiveDirectory.Management.dll
PS C:\> Import-Module .\ActiveDirectory.psd1
```

## Get-ADDomain

```powershell
Get-ADDomain
Get-ADDomain -Identity vulnableone.local
```

## DomainSID

```powershell
(Get-ADDomain).DomainSID
(Get-ADDomain -Identity vulnableone.local).DomainSID
```

## Get-ADDomainController

```powershell
Get-ADDomainController
Get-ADDomainController -DomainName vulnableone.local -Discover
```

## Get-ADUser

```powershell
Get-ADUser -Filter * -Properties * |more
Get-ADUser -Identity khan.chanthou -Properties *
Get-ADUser -Filter * -Properties * | select samaccountname, pwdlastset, logoncount
Get-ADUser -Filter * -Properties * | select samaccountname, @{expression={[datetime]::fromFileTime($_.pwdlastset)}}, logoncount
Get-ADUser -Filter * -Properties * | select samaccountname, description
Get-ADUser -Filter 'Description -like "*built*"' -Properties Description | select name, Description
Get-ADUser -Filter 'Name -like "*admin*"' -Properties Description | select name, Description
```

## Get-ADComputer

```powershell
Get-ADComputer -Filter * |select Name
Get-ADComputer -Filter * -Properties Name | ft Name,DNSHostName,IPv4Address
Get-ADComputer -Filter 'OperatingSystem -like "*Windows Server 2019 Standard*"' -Properties OperatingSystem | select name, operatingsystem
Get-ADComputer -Filter 'OperatingSystem -like "*Windows 11*"' -Properties OperatingSystem | select name, operatingsystem
Get-ADComputer -Filter * -Properties DNSHostName | %{Test-Connection -Count 1 -ComputerName $_.DNSHostName}
```

## Get-ADGroup

```powershell
Get-ADGroup -Filter * | select name
Get-ADGroup -Filter * -Properties *
Get-ADGroup -Filter 'Name -like "*admin*"' | select name
Get-ADGroup -Filter 'Name -like "*admin*"' -Server vulnableone.local | select name
Get-ADGroupMember -Identity "Domain Admins" -Recursive
Get-ADGroupMember -Identity "Enterprise Admins" -Server vulnableone.local
Get-ADPrincipalGroupMembership -Identity khan.chanthou
```

## Get-ADOrganizationalUnit

```powershell
Get-ADOrganizationalUnit -Filter * -Properties *
Get-ADOrganizationalUnit -Filter * -Properties *  | select name, gplink
Get-ADOrganizationalUnit -Identity 'OU=Domain Controllers,DC=vulnableone,DC=local' | %{Get-ADComputer -SearchBase $_ -Filter *} | select name
```

## Get-ACL

```powershell
Get-ACL 'AD:\CN=Domain Admins,CN=Users,DC=vulnableone,DC=local' | select -ExpandProperty Access
```

## Get-ADTrust

```powershell
Get-ADTrust -Filter *
Get-ADTrust -Identity vulnableone.local
```

## Get-ADForest

```powershell
Get-ADForest
(Get-ADForest).Domains
Get-ADForest | select -ExpandProperty GlobalCatalogs
Get-ADTrust -Filter 'intraForest -ne $True' -Server (Get-ADForest).Name
(Get-ADForest).Domains | %{Get-ADTrust -Filter '(intraForest -ne $True) -and (ForestTransitive -ne $True)'-Server $_}
Get-ADTrust -Filter * -Server vulnableone.local
```

## Kerberoast

```powershell
Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName
```

## Unconstrained Delegation

```powershell
Get-ADUser -Filter {TrustedForDelegation -eq $True}
Get-ADComputer -Filter {TrustedForDelegation -eq $True}
```

## Constrained Delegation

```powershell
Get-ADObject -Filter {msDS-AllowedToDelegateTo -ne "$null"} -Properties msDS-AllowedToDelegateTo
```
