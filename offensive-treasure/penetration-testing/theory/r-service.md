---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/theory/r-service
---

# R\* Service

### host.equiv (or .rhosts file) Structure

```bash
Allow any user to log in from any host:
+


Allow any user from host with a matching local account to log in:
host


Allow any user from host to log in:
host +


Allow user from host to log in as any non-root user:
host user


Allow all users with matching local accounts from host to log in except for baduser:
host -baduser
host


Deny all users from host:
-host


Allow all users with matching local accounts on all hosts in a netgroup:
+@netgroup


Disallow all users on all hosts in a netgroup:
-@netgroup


Allow all users in a netgroup to log in from host as any non-root user:
host +@netgroup


Allow all users with matching local accounts on all hosts in a netgroup except baduser:
+@netgroup -baduser
+@netgroup
```
