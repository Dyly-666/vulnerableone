---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/execution/wsh
---

# WSH

<figure><img src="../../../.gitbook/assets/image (222).png" alt=""><figcaption></figcaption></figure>

Double click vbs file and it will execute

```javascript
Dim shell
Set shell = WScript.CreateObject("WScript.Shell")
shell.Run("calc.exe")
```

<figure><img src="../../../.gitbook/assets/image (220).png" alt=""><figcaption></figcaption></figure>

If the VBS files are blacklisted, we can rename the file to **.txt** file and run it using wscript as follows,

```javascript
C:\Users\sieng.chantrea\Desktop>wscript /e:VBScript test.txt
```

<figure><img src="../../../.gitbook/assets/image (221).png" alt=""><figcaption></figcaption></figure>
