---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/enumeration/service-ports/tftp-69-udp
---

# TFTP (69/udp)

## TFTP Enumeration

We can connect tftp to the machine. We can verify if the OS is linux or window by grab some files.\
**Linux: /etc/passwd**\
**Window: /Windows/System32/license.rtf**

```bash
└─$ tftp 10.10.10.10  
tftp> get \Windows\System32\license.rtf
Received 61533 bytes in 35.3 seconds
```

```
\Windows\System32\license.rtf	return the path
"/Windows/System32/license.rtf"	specific file
```

## Nmap

```bash
└─$ sudo nmap -n -Pn -sU -p69 -sV --script tftp-enum 10.10.10.10                                                                                                                          1 ⨯
Nmap scan report for 10.11.1.111
Host is up.

PORT   STATE SERVICE VERSION
69/udp open  tftp?
```

## Download

```basic
tftp> get /PROGRA~1/MICROS~1/MSSQL1~1.SQL/MSSQL/Backup/master.mdf
Received 4194541 bytes in 2476.8 seconds
```

Reference: [https://github.com/xpn/Powershell-PostExploitation](https://github.com/xpn/Powershell-PostExploitation)
