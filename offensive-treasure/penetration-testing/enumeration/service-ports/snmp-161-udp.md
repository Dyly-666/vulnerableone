---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/enumeration/service-ports/snmp-161-udp
---

# SNMP (161/udp)

## Nmap

```basic
nmap -p161 -sU -sC 10.10.10.116
```

## snmpwalk

```basic
snmpwalk -c public -v1 10.10.10.10
snmpwalk -c public -v2c 10.10.10.10
```

* **-v 1|2c|3** specifies SNMP version to use
* **-c COMMUNITY** set the community string

## snmp-check

```basic
snmp-check 10.10.10.10
```
