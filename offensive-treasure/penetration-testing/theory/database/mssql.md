---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/theory/database/mssql
---

# MSSQL

### DB Version

```
SELECT @@version

EXEC xp_msver
(detailed version info)
```

### Run OS Command

```
EXEC master..xp_cmdshell 'net user'
```

### SELECT commands

```
SELECT HOST_NAME( ) : Hostname and IP

SELECT DB_NAME ( ) : Current DB

SELECT name FROM master..sysdatabases; : List DBs

SELECT user_name ( ) : Current user

SELECT name FROM master..syslogins : List users

SELECT name FROM master..sysobjects WHERE xtype='U'; : List Tables

SELECT name FROM syscolumns WHERE id=(SELECT id FROM sysobjections WHERE name='mytable'); : List columns
```

### List all Tables and Columns

```
SELECT name FROM syscolumns WHERE id = (SELECT id FROM sysobjects WHERE name = 'mytable')
```

### System Table (Info on All Tables)

```
SELECT TOP 1 TABLE_NAME FROM INFORMATION_SCHEMA.TABLES
```

### MS-SQL 2005 Vulnerability (Password Hashes)

```
SELECT name, password_hash FROM master.sys.sql_logins
```
