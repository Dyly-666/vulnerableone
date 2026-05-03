---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/theory/ip-address
---

# IP Address

## IPv4 Address

### Reversed Internal IPs

```
10.0.0.0/8    (10.0.0.0 - 10.255.255.255)    : Private
127.0.0.0/8   (127.0.0.1 - 127.255.255.255)    : Local Host
172.16.0.0/12    (172.16.0.0 - 172.31.255.255)    : Private
169.254.1.0 - 169.254.254.255    : APIPA
240.0.0.1 - 255.255.255.254    : Reserved
```

<table><thead><tr><th></th><th width="134.33333333333331"></th><th></th></tr></thead><tbody><tr><td>1-126</td><td>Class A</td><td>/8</td></tr><tr><td>128-191</td><td>Class B</td><td>/16</td></tr><tr><td>192-223</td><td>Class C</td><td>/24</td></tr><tr><td>224-239</td><td>Class D</td><td>Multicast</td></tr><tr><td>240-254</td><td>Class E</td><td>Reserved</td></tr><tr><td>127.0.0.1 - 127.255.255.254</td><td>Loopback</td><td></td></tr><tr><td>240.0.0.1 - 255.255.255.254</td><td>Reserved</td><td></td></tr><tr><td>169.254.1.0 - 169.254.254.255</td><td>Reversed</td><td>APIPA</td></tr></tbody></table>

## IPv6 Address

```
BROADCAST ADDRESSES 
ff02::1- link-local nodes 
ff05::1- site-local nodes 
ff01::2- node-local routers 
ff02::2- link-local routers 
ff05::2- site-local routerd

fe80:: -link-local 
2001:: - routable 
```

<figure><img src="../../../.gitbook/assets/image (951).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (952).png" alt=""><figcaption></figcaption></figure>
