---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/resource-development/c2-infrastructure
---

# C2 Infrastructure

## C2 Setup Diagram: Soon !!!

<figure><img src="../../../.gitbook/assets/image (950).png" alt=""><figcaption></figcaption></figure>

{% code title="KaliBox" %}
```bash
└─$ ssh -N -R 8443:localhost:443 ubuntu@10.10.10.10
```
{% endcode %}

This command will allocate port 8443 on the Public Cloud. Any incoming traffic on that port will be directed to 127.0.0.1:443 on the Kali Box. As the C2 listener binds to all interfaces (0.0.0.0), this will ensure that the traffic is routed to the listener.
