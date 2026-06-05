# Evil-Winrm

#### Using evil-winrm <a href="#using-evil-winrm" id="using-evil-winrm"></a>

{% code overflow="wrap" %}
```bash
# Basic connection
evil-winrm -i target.com -u administrator -p 'password'

# With domain
evil-winrm -i target.com -u 'DOMAIN\username' -p 'password'

# Using hash (Pass-the-Hash)
evil-winrm -i target.com -u administrator -H 'NTHASH'

# Using SSL (port 5986)
evil-winrm -i target.com -u administrator -p 'password' -S

# With custom port
evil-winrm -i target.com -u administrator -p 'password' -P 5985

# With kerberos ticket
export KRB5CCNAME=jason.caldwell.ccache; evil-winrm -i dc.locktail.martin -r LOCKTAIL.MARTIN
```
{% endcode %}

#### Using PowerShell (from Windows) <a href="#using-powershell-from-windows" id="using-powershell-from-windows"></a>

{% code overflow="wrap" %}
```powershell
# Create credentials
$password = ConvertTo-SecureString "password" -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential("administrator", $password)

# Connect interactively
Enter-PSSession -ComputerName target.com -Credential $cred

# Run command remotely
Invoke-Command -ComputerName target.com -Credential $cred -ScriptBlock { whoami }

# Connect to multiple machines
$computers = "server1", "server2", "server3"
Invoke-Command -ComputerName $computers -Credential $cred -ScriptBlock { hostname }
```
{% endcode %}

#### Using winrs (Windows Remote Shell) <a href="#using-winrs-windows-remote-shell" id="using-winrs-windows-remote-shell"></a>

{% code overflow="wrap" %}
```powershell
# Execute single command
winrs -r:http://target.com:5985 -u:administrator -p:password "whoami"

# Interactive shell
winrs -r:http://target.com:5985 -u:administrator -p:password cmd

# With domain
winrs -r:http://target.com:5985 -u:DOMAIN\username -p:password cmd
```
{% endcode %}
