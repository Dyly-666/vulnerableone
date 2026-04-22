---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/enumeration/service-ports/dns-53
---

# DNS (53)

## Nslookup

```bash
nslookup
> server 10.10.10.13
Default server: 10.10.10.13
Address: 10.10.10.13#53
> 10.10.10.13
13.10.10.10.in-addr.arpa        name = ns1.cronos.htb.
> ns1.cronos.htb
Server:         10.10.10.13
Address:        10.10.10.13#53

Name:   ns1.cronos.htb
Address: 10.10.10.13
```

## Dig&#x20;

```bash
# Zone Transfer
└─$ dig axfr cronos.htb @10.10.10.13

; <<>> DiG 9.16.15-Debian <<>> axfr cronos.htb @10.10.10.13
;; global options: +cmd
cronos.htb.             604800  IN      SOA     cronos.htb. admin.cronos.htb. 3 604800 86400 2419200 604800
cronos.htb.             604800  IN      NS      ns1.cronos.htb.
cronos.htb.             604800  IN      A       10.10.10.13
admin.cronos.htb.       604800  IN      A       10.10.10.13
ns1.cronos.htb.         604800  IN      A       10.10.10.13
www.cronos.htb.         604800  IN      A       10.10.10.13
cronos.htb.             604800  IN      SOA     cronos.htb. admin.cronos.htb. 3 604800 86400 2419200 604800
;; Query time: 63 msec
;; SERVER: 10.10.10.13#53(10.10.10.13)
;; WHEN: Wed Nov 03 05:37:37 EDT 2021
;; XFR size: 7 records (messages 1, bytes 203)
```

```bash
# Resolve and Verify domain
└─$ dig @10.10.10.161 forest.htb.local
```

## Host (Zone Transfer)

```bash
└─$ host -l cronos.htb 10.10.10.13                                                                                                                                    127 ⨯
Using domain server:
Name: 10.10.10.13
Address: 10.10.10.13#53
Aliases: 

cronos.htb name server ns1.cronos.htb.
cronos.htb has address 10.10.10.13
admin.cronos.htb has address 10.10.10.13
ns1.cronos.htb has address 10.10.10.13
www.cronos.htb has address 10.10.10.13
```

## BruteForce Subdomain

```bash
└─$ gobuster dns -d cronos.htb -w /usr/share/wordlists/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt -t 50
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Domain:     cronos.htb
[+] Threads:    50
[+] Timeout:    1s
[+] Wordlist:   /usr/share/wordlists/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt
===============================================================
2021/11/03 06:51:16 Starting gobuster in DNS enumeration mode
===============================================================
Found: www.cronos.htb
Found: admin.cronos.htb
                                   
===============================================================
2021/11/03 07:07:26 Finished
===============================================================
```
