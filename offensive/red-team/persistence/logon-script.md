---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/persistence/logon-script
---

# Logon Script

To create an environment variable for a user, you can go to its **HKCU\Environment** in the registry. We will use the <mark style="color:red;">**UserInitMprLogonScript**</mark> entry to point to our payload so it gets loaded when the user logs in:

```
Computer\HKEY_CURRENT_USER\Environment
```

<figure><img src="../../../.gitbook/assets/image (205).png" alt=""><figcaption></figcaption></figure>

After doing this, sign out of your current session and log in again, and you should receive a shell.

<figure><img src="../../../.gitbook/assets/image (206).png" alt=""><figcaption></figcaption></figure>
