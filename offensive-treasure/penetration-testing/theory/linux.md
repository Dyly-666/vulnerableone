---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/theory/linux
---

# Linux

### Find File Type

```
find . -type f -iname '*.pdf'
locate '*.pdf'
```

### File Structure

```
/bin - User Binaries
/boot - Bootup related files
/dev - Interface for system devices
/etc - System Config Files
/home - Base directory for user files
/lib - Critical software libraries
/opt - Third party software
/proc - System and running processes
/root - Home for root
/sbin - Sys Admin binaries
/tmp - Temporary Files
/usr - Less critical files
/var - Variable system files
```

### Add User, Make Sudoer

```
useradd <user> (adduser <user>)
passwd <user>
sudo useradd <user> 
sudo (sudo adduser <user> sudo)
```
