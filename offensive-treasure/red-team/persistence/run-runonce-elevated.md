---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/persistence/run-runonce-elevated
---

# Run / RunOnce (Elevated)

You can use the following registry entries to specify applications to run at logon:

```
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce
HKLM\Software\Microsoft\Windows\CurrentVersion\Run
HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce
```

Let's then create a **REG\_EXPAND\_SZ** registry entry under

```
HKLM\Software\Microsoft\Windows\CurrentVersion\Run
```

<figure><img src="../../../.gitbook/assets/image (211).png" alt=""><figcaption></figcaption></figure>

After doing this, sign out of your current session and log in again, and you should receive a shell.

<figure><img src="../../../.gitbook/assets/image (212).png" alt=""><figcaption></figcaption></figure>
