---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/persistence/startup-folder
---

# Startup Folder

Applications, files and shortcuts within a user's startup folder are launched automatically when they first log in.

```
C:\Users\UserName\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
```

<figure><img src="../../../.gitbook/assets/image (907).png" alt=""><figcaption></figcaption></figure>

Every time user restart, we will got a shell pop up again.

<figure><img src="../../../.gitbook/assets/image (908).png" alt=""><figcaption></figcaption></figure>

This path will required elevated privilege:

```
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup
```
