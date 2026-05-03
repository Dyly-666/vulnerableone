---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/theory/database/mysql
---

# MySQL

### SELECT Commands

```
SELECT @@version; : DB Version

SELECT @@hostname; : Hostname and IP

SELECT database(); : Current DB

SELECT distinct (db) FROM mysql.db; : List DBs

SELECT user(); : Current user

SELECT user FROM mysql.user; : List Users

SELECT host,user,password FROM mysql.user; : List password hashes
```

### List Tables (and Columns)

```
SHOW TABLES (only works for current database)

SELECT * FROM information_schema.columns (full dump)
```
