---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/enumeration/service-ports/ftp-21
---

# FTP (21)

## 4 Points to Check

1. Version number and associated CVEs
2. Anonymous / authenticated login (with discovered creds)
3. Sensitive files that you have read access to
4. File Upload

### Credential Test

```basic
anonymous:anything
admin:admin
admin:password
username:username
guest:null
```

### Nmap Script

```basic
# List all the nse script of ftp service
ls /usr/share/nmap/scripts/ftp*
```

### Connect to Custom Port

```bash
└─$ ftp
ftp> open 10.10.10.10 20001
Connected to 10.10.10.10.
220-FileZilla Server version 0.9.41 beta
220-written by Tim Kosse (Tim.Kosse@gmx.de)
220 Please visit http://sourceforge.net/projects/filezilla/
Name (10.10.10.10:pwned): anonymous
331 Password required for anonymous
Password:
230 Logged on
Remote system type is UNIX.
ftp> dir
200 Port command successful
150 Opening data channel for directory list.
```

### FTP Technique&#x20;

```bash
ftp $IP 2121
ftp> quote PASV
Entering passive mode (192,168,123,123,123,123).
ftp> ls -las
```
