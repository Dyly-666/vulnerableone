---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/theory/database/postgresql
---

# PostgreSQL

### SELECT commands

```
SELECT version(); : DB Version

SELECT inet_server)addr(); : Hostname and IP

SELECT current_database(); : Current DB

SELECT datname FROM pg_database; : List DBs

SELECT user; : Current user

SELECT username FROM pg_user; : List Users

SELECT username,passwd FROM pg_shadow : List password hashes
```
