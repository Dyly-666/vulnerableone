---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/theory/windows
---

# Windows

### SAM

{% code overflow="wrap" %}
```
Stores Windows users' passwords in a hashed format (in LM hash and NTLM hash). These are backups of C:\windows\system32\config\SAM

%SYSTEMROOT%\repair\SAM
%SYSTEMROOT%\System32\config\RegBack\SAM
```
{% endcode %}

### SystemInfo

```
ver : OS Version
sc query state=all : Services
tasklist /svc : Processes and Services
echo %USERNAME% : Current user
```

### Find Files of Type

```
dir /a /s /n c:\.pdf
```

### Add User, Make Admin

```
net user <user> <pass> /add
net localgroup "Administrators" <user> /add
```

### Disable Firewall

```
netsh advfirewall set currentprofile state off
netsh advfirewall set allprofiles state off
```
