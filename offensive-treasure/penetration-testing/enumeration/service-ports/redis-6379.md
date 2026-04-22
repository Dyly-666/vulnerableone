---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/enumeration/service-ports/redis-6379
---

# Redis (6379)

## Connect to the Server

```basic
$ redis-cli -h 10.10.10.10
NoAuth Require
```

We need to find password to connect which stored in <mark style="color:red;">**/etc/redis/redis.conf**</mark> file

<figure><img src="../../../../.gitbook/assets/image (231).png" alt=""><figcaption></figcaption></figure>

```basic
└─$ redis-cli -h 10.10.10.10
10.10.10.10:6379> Auth Ready4Redis?
OK
```
