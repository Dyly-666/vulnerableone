---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/enumeration/service-ports/isakmp-500-udp
---

# isakmp (500/udp)

## ike-scan

```basic
ike-scan -A 10.10.10.10
```

* **--aggressive or -A** Use IKE Aggressive Mode (will return hash of pre-shared key)

```basic
└─$ ike-scan --ikev2 10.10.10.10 -M
```

* **--ikev2 or -2** Use IKE version 2
* **--multiline or -M** Split the payload decode across multiple lines.
