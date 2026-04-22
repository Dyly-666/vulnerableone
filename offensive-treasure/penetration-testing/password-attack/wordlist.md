---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/password-attack/wordlist
---

# Wordlist

## Generate 4 PINs

```python
from pwn import *
for i in range(0,9999):
    pin = str(i)
    code = pin.zfill(4)
    print(code)
    
pip3 install pwntools

python3 script.py
```

## Username Wordlist

```basic
/usr/share/metasploit-framework/data/wordlists/common_users.txt
/usr/share/seclists/Usernames/Names/names.txt
/usr/share/seclists/Usernames/xato-net-10-million-usernames.txt
```

## Password Wordlist

```basic
usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
/usr/share/wfuzz/wordlist/others/common_pass.txt
/usr/share/wordlists/rockyou.txt
```

## Generate PW with cewl

```basic
└─$ cewl $url -m 5 -d 4 -w $PWD/cewl.txt 2>/dev/null
```

* <mark style="color:red;">**-m**</mark> extract minimum word length
* <mark style="color:red;">**-w**</mark> write output to&#x20;
* <mark style="color:red;">**-d**</mark> depth to spider (How many page you want to check)
* <mark style="color:red;">**$PWD**</mark> current directory

## Web application

```basic
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
/usr/share/wordlists/dirb/common.txt 
/usr/share/wordlists/dirbuster/directory-list-2.3-large.txt
/usr/share/wordlists/dirb/big.txt
/usr/share/wordlists/dirb/small.txt
```

### Create Password List

```basic
└─$ cat pass.txt       
January
Febuary
March
April
May
Password
Secret

└─$ for i in $(cat pass.txt); do echo $i; echo ${i}2020; echo ${i}2021; done                                                                                                              1 ⨯
January
January2020
January2021
Febuary
Febuary2020
Febuary2021
March
March2020
March2021
April
April2020
April2021
May
May2020
May2021
Password
Password2020
Password2021
Secret
Secret2020
Secret2021

└─$ for i in $(cat test.txt); do echo $i; echo ${i}\!; done
January
January!
January2020
January2020!
January2021
January2021!
Febuary
Febuary!
Febuary2020
Febuary2020!
Febuary2021
Febuary2021!
March
March!
March2020
March2020!
March2021
March2021!
April
April!
April2020
April2020!
April2021


sudo hashcat --force --stdout test.txt -r /usr/share/hashcat/rules/best64.rule | awk 'length($0) > 7'
```

### Generate hash with PHP

```basic
└─$ php -a                                                      
Interactive mode enabled

php > echo password_hash('password', PASSWORD_DEFAULT);
$2y$10$cloAITtRNIM8u3Uu8pxg2eCymqPoTKXYyw4fz9Da5wZ12w9trk2m.
php > 
```
