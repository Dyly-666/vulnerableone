---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/theory/database/page-7
---

# Page 7

## Default Credential:

```
--Username | Password--
SYSTEM | MANAGER
ANONYMOUS | ANONYMOUS
SCOTT | TIGER
OLAPSYS | MANAGER
SYS | CHANGE_ON_INSTALL
```

### SELECT Commands

```
SELECT * FROM v$version; : DB Version
(SELECT version FROM v$instance;)

SELECT instance_name FROM v$instance : Current DB
(SELECT name FROM v$database;)

SELECT DISTINCT owner FROM all_tables; : List DBs

SELECT user FROM dual; : Current User

SELECT username FROM all_users ORDER BY username; : List users

SELECT column_name FROM all_tab_columns; : List Columns

SELECT table_name FROM all_tables; : List Tables

SELECT name, password, astatus FROM sys.user$; : List password hashes
```
