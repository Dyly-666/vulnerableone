---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/theory/windows/wmic
---

# WMIC

**Command : Execute Process**

```
wmic process call create "process_name"
```

### Command : Uninstall Software

```
wmic product get name /value
wmic product where name="XX" call uninstall /nointeractive
```
