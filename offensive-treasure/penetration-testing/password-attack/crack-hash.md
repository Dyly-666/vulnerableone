---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/password-attack/crack-hash
---

# Crack Hash

## Hashcat Example

```basic
hashcat --example-hashes | grep -i unix -B 1 -A 1
```

## Kerberoast&#x20;

Crack the ticket with Hashcat

```basic
hashcat -m 13100 -a 0 hash.txt /usr/share/wordlist/rockyou.txt
```

Crack the ticket with John

```basic
kirb2j0hn ticket.kirbi > crackfile
john --format=krb5tgs crackfile --wordlist=10k-worst-pass.txt
```

Crack with kerberos tool

{% code overflow="wrap" %}
```basic
python tgsrepcrack.py wordlist.txt 1-40a5@HTTP~Service.com
```
{% endcode %}

## ASREP roast

Mode: **Kerberos 5, etype 23, AS-REP - 18200**

```
└─$ hashcat -m 18200 -a 0 hash.txt password.txt 
```

```basic
└─$ john --format=krb5asrep --wordlist=/usr/share/wordlists/rockyou.txt asreproast-hash.txt 
```

## Crack private key

```basic
└─$ locate ssh2john                                                                                                                                                                       1 ⨯
/usr/share/john/ssh2john.py

└─$ /usr/share/john/ssh2john.py id_rsa > hash.txt
                                                                                                 
└─$ john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt                                                                                                                           255 ⨯
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 1 for all loaded hashes
Cost 2 (iteration count) is 2 for all loaded hashes
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
computer2008     (id_rsa)
1g 0:00:00:22 DONE (2021-12-03 21:25) 0.04403g/s 631514p/s 631514c/s 631514C/s *7¡Vamos!
Session completed
```

## PDF Crack

```basic
sudo apt install pdfcrack

└─$ pdfcrack -f Infrastructure.pdf --w /usr/share/wordlists/rockyou.txt                                                                                                                   1 ⨯
PDF version 1.7
Security Handler: Standard
V: 2
R: 3
P: -1060
Length: 128
Encrypted Metadata: True
FileID: 14350d814f7c974db9234e3e719e360b
U: 6aa1a24681b93038947f76796470dbb100000000000000000000000000000000
O: d9363dc61ac080ac4b9dad4f036888567a2d468a6703faf6216af1eb307921b0
Average Speed: 68625.4 w/s. Current Word: 'pumar31'
Average Speed: 68448.6 w/s. Current Word: 'windblo jr'
```

## Rar crack

Cracking the hash with hashcat. Let check the hash type example

```basic
└─$ hashcat --example-hashes

MODE: 13000
TYPE: RAR5
HASH: $rar5$16$38466361001011015181344360681307$15$00000000000000000000000000000000$8$cc7a30583e62676a
PASS: hashcat
```

```basic
└─$ rar2john MSSQL_BAK.rar > crack.txt
                                                                                              
└─$ cat crack.txt      
MSSQL_BAK.rar:$rar5$16$53b1acf5cd3d02dafdf50f1cb79e46e5$15$a8761ee8f467302d9ee19284f60713dd$8$514688ceb07cab7b
```

```basic
hashcat-6.2.3>hashcat.exe -m 13000 -a 0 hash-to-crack.txt rockyou.txt

Dictionary cache hit:
* Filename..: rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

$rar5$16$53b1acf5cd3d02dafdf50f1cb79e46e5$15$a8761ee8f467302d9ee19284f60713dd$8$514688ceb07cab7b:letmeinplease
```

## Zip Crack

```basic
zip2john file.zip > hash.txt 
john --wordlist=/usr/share/wordlist/rockyou.txt hash.txt
```

## Crack Passwd and Shadow File

```basic
└─$ unshadow passwd.txt shadow.txt 
bob:$1$Rrhb4lzg$Ee8/JYZjv.NimwyrSEL6R/:500:500::/home/bob:/bin/bash
```

## Crack apache hash

Let check the hash type example

```basic
└─$ hashcat --example-hashes

MODE: 1600
TYPE: Apache $apr1$ MD5, md5apr1, MD5 (APR)
HASH: $apr1$62722340$zGjeAwVP2KwY6MtumUI1N/
PASS: hashcat
```

We are able to crack the password

```basic
C:\>hashcat.exe -m 1600 -a 0 hash-to-crack.txt rockyou.txt --show
$apr1$oRfRsc/K$UpYpplHDlaemqseM39Ugg0:elite
```
