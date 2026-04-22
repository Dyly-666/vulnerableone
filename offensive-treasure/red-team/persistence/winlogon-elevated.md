---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/persistence/winlogon-elevated
---

# WinLogon (Elevated)

Winlogon uses some registry keys under that could be interesting to gain persistence:

* **Userinit** points to **userinit.exe**, which is in charge of restoring your user profile preferences.
* **shell** points to the system's shell, which is usually **explorer.exe**

```
Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon
```

<figure><img src="../../../.gitbook/assets/image (208).png" alt=""><figcaption></figcaption></figure>

We can modify the string by adding our shellcode path

<figure><img src="../../../.gitbook/assets/image (209).png" alt=""><figcaption></figcaption></figure>

After doing this, sign out of your current session and log in again, and you should receive a shell

<figure><img src="../../../.gitbook/assets/image (210).png" alt=""><figcaption></figcaption></figure>
