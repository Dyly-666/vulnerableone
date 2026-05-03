---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/credential-access/adversary-in-the-middle/evil-ssdp
---

# Evil SSDP

This tool responds to SSDP multicast discover requests, posing as a generic UPNP device on a local network. Your spoofed device will magically appear in Windows Explorer on machines in your local network. Users who are tempted to open the device are shown a configurable webpage.

Requirement:

<figure><img src="../../../../.gitbook/assets/image (912).png" alt=""><figcaption></figcaption></figure>

Attacker on the same network could spoof Printer

```bash
└─$ evil-ssdp eth0 --template scanner
```

On User network, there is a printer is waiting

<figure><img src="../../../../.gitbook/assets/image (913).png" alt=""><figcaption></figcaption></figure>

If you user open, it will request for credentials

<figure><img src="../../../../.gitbook/assets/image (914).png" alt=""><figcaption></figcaption></figure>

Attacker will obtained **cleartext credential**

<figure><img src="../../../../.gitbook/assets/image (915).png" alt=""><figcaption></figcaption></figure>
