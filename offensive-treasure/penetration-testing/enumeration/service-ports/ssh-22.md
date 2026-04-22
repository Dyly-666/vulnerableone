---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/enumeration/service-ports/ssh-22
---

# SSH (22)

## Nmap Script

```basic
ls /usr/share/nmap/scripts/ssh*
```

## Create SSH Key

If we want to login to other machine, we can generate ssh-keygen, copy public key value <mark style="color:red;">**.pub**</mark> to <mark style="color:red;">**authorized\_keys**</mark>

{% code overflow="wrap" %}
```bash
└─$ ssh-keygen -f theseus
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in theseus
Your public key has been saved in theseus.pub
The key fingerprint is:
SHA256:dbZ/hBR4OAZZ+Qgjl7Zwg6pT0nHMym44tPXFi3+mjf4 pwned@kali
The key's randomart image is:
+---[RSA 3072]----+
|       o ..=.+.  |
|      . B X * .. |
|     o = B.*o=.  |
|    o B  .+o.o.. |
|   . O .So .. . .|
|    * o o .  . . |
|     +   .    . .|
|          .oo  . |
|         .+=E    |
+----[SHA256]-----+


└─$ cat theseus.pub  
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDR10KGNMOMTFEPXzo6dbBh1z0qWqkPi7/lAbY92gnd3t4GLWEQizKaNe/gWpI9g7y9XAD/VFDEKgZG0LcnFgQb0wQ8PZY6oMMr9RCxvnEhbVg7w+tcjJCJQFTI8zkP+tN1b4mwo1OZfsxrIynbMSxsKSaDpel5BICa2BIu3N7l9P20u3+DDxaP2Fn2mky7/ApWC9T82qOE0vbn2Ows2DWCggfxSLMj3YpZdlQez23BpvbMG3PxafOM6L+w0UNTTOq5gFiNyPbEWHSmddqL697cmFIjOuo9hcHgchj22T0uTzIjZdMmdpHWFHaawcVCtG0+VaHABCvfNb8rotzix7kMxwWc9bjRb6ZafqPsPm+XrGQb/cYmgKdAw3wQj1Qp+PA65Ol5y/eftLaGieAnkkAFGQEGupsAggLacap+Mj/1nLcAUUh4O5bvEi0pu91kwo43XnTrg8uUhKv6HVJSQG1XT3xXMAeFUkVj1Mv6Uq1YrnBRP0brDxF7y//Kqb8cbn0= pwned@kali

└─[~/.ssh]$ echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDR10KGNMOMTFEPXzo6dbBh1z0qWqkPi7/lAbY92gnd3t4GLWEQizKaNe/gWpI9g7y9XAD/VFDEKgZG0LcnFgQb0wQ8PZY6oMMr9RCxvnEhbVg7w+tcjJCJQFTI8zkP+tN1b4mwo1OZfsxrIynbMSxsKSaDpel5BICa2BIu3N7l9P20u3+DDxaP2Fn2mky7/ApWC9T82qOE0vbn2Ows2DWCggfxSLMj3YpZdlQez23BpvbMG3PxafOM6L+w0UNTTOq5gFiNyPbEWHSmddqL697cmFIjOuo9hcHgchj22T0uTzIjZdMmdpHWFHaawcVCtG0+VaHABCvfNb8rotzix7kMxwWc9bjRb6ZafqPsPm+XrGQb/cYmgKdAw3wQj1Qp+PA65Ol5y/eftLaGieAnkkAFGQEGupsAggLacap+Mj/1nLcAUUh4O5bvEi0pu91kwo43XnTrg8uUhKv6HVJSQG1XT3xXMAeFUkVj1Mv6Uq1YrnBRP0brDxF7y//Kqb8cbn0= pwned@kali" > authorized_keys
```
{% endcode %}

### Split Private Key

{% code overflow="wrap" %}
```python
└─$ python2
Python 2.7.18 (default, Apr 28 2021, 17:39:59) 
[GCC 10.2.1 20210110] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> str = "b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcnNhAAAAAwEAAQAAAQEAuhHlUvBrrQizdVsrfkyzQPQcaDCEH/5mE+fxiJmwuJ39670pHLJHcP/dXo7ZIWK/eHhkXLPLP6/xFmjZk2vQra8bQiM2iBaUtDrb1wfynNONQSvQkFLpJfmjlJYO7mhn7h2XjEoaBacQ+gjEw9MHvgpYOi4Ho/7yRPpLhQw9I4CSSFcp9/JvaISblBVRXux8O3C2k/i4Rhxzx7Pct6mnsRssArDhDFCjAmmDArKCPJUgn7byM0xVMLu4u+hfa1BV2KAJbfLzjJ7ZuOjpUG68Gtp8EoBUHcyCgK6UpDFB7hoRMXVSIoF1AfRHnZ2VfAVfzOD/jkX/0TQyMO+zw3c6bwAAA8hrkGRBa5BkQQAAAAdzc2gtcnNhAAABAQC6EeVS8GutCLN1Wyt+TLNA9BxoMIQf/mYT5/GImbC4nf3rvSkcskdw/91ejtkhYr94eGRcs8s/r/EWaNmTa9CtrxtCIzaIFpS0OtvXB/Kc041BK9CQUukl+aOUlg7uaGfuHZeMShoFpxD6CMTD0we+Clg6Lgej/vJE+kuFDD0jgJJIVyn38m9ohJuUFVFe7Hw7cLaT+LhGHHPHs9y3qaexGywCsOEMUKMCaYMCsoI8lSCftvIzTFUwu7i76F9rUFXYoAlt8vOMntm46OlQbrwa2nwSgFQdzIKArpSkMUHuGhExdVIigXUB9EednZV8BV/M4P+ORf/RNDIw77PDdzpvAAAAAwEAAQAAAQBjcnshl/PEuHjJyV92kmHf3lhsaznCq8I882OJQbNNCMwUubYGa1Z5k5bqGej8yf1R0u65CTMhJ9TvyDw5aY9PtN4ZvB5CH+d8aFTlGY9WuE6vvU4sRNPtgv4lxQnX7B9YCaLczSIZUVBmglc/3kMuE/NRrRZSVUmBClFgm8j1drD6A3fpfGEP1ozooWvqZ8an2f5pKee6U9NTik3UECyie9rhEEwFFJ/dxD0knUMx5/a1aF1IU4VHoE2JQZS0VaccGKck+PRKiE1CNbEJdsEU7dZnwRNN0QBlS+sEVYZjJ/LK3RnoxIBqbSCiTkMNtcja8JDPGdRSjHK05WQOwdEJAAAAgEHFId3DVBTsGmg/mZe+N1RyEpwLLK+e+PIMhNYECC7EEOrv2lynUsFKSrB30inOm99HW5z4v7R3xeb7d3reLoubdsCDR8Bp0lU2gtjKbeuupn6hhFn/lSys4M4rU4raLEtOVVqUEpSeGCUZSsmAqeO1oCbVjNby0P6CtQB89wz6AAAAgQDuzbiMLPF9QcuOK+E3ZLyXth47M4oArr72xImRvOSOuhgGEE/e1kf1gNSpWCVR8BAgdlCQF1M+kffAgk2uxcWoPSHZdXvza3yPA0pIeb/v6x3NaGvmYMITayIg92TJKoQavKizPyZK6V7m8GpnX7cBbuIfdPjxAadaKlhi2yAMuwAAAIEAx3gLL9eOzYENox+VFy3polncOAMA9Oi8UhBnY/cUOv8XksBQPBQDxifHgdT7iTjUMVEk0dL/YkzWr5ZsWJ81tj2hM4wXbGOM9Kz1kfUCBkVGtgFXbxWCe/W3mY3Kngw5VzHjsDuEb14v5YmMkc7bic/RQbNxak/vgF0CAC63Z90AAAAQcDR5bDBhZEBTWU1CT0xJQwECAw=="
>>> n = 70
>>> for i in range (0, len(str), n):
...  print str[i:i+n]
... 
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn
NhAAAAAwEAAQAAAQEAuhHlUvBrrQizdVsrfkyzQPQcaDCEH/5mE+fxiJmwuJ39670pHLJH
cP/dXo7ZIWK/eHhkXLPLP6/xFmjZk2vQra8bQiM2iBaUtDrb1wfynNONQSvQkFLpJfmjlJ
YO7mhn7h2XjEoaBacQ+gjEw9MHvgpYOi4Ho/7yRPpLhQw9I4CSSFcp9/JvaISblBVRXux8
O3C2k/i4Rhxzx7Pct6mnsRssArDhDFCjAmmDArKCPJUgn7byM0xVMLu4u+hfa1BV2KAJbf
LzjJ7ZuOjpUG68Gtp8EoBUHcyCgK6UpDFB7hoRMXVSIoF1AfRHnZ2VfAVfzOD/jkX/0TQy
MO+zw3c6bwAAA8hrkGRBa5BkQQAAAAdzc2gtcnNhAAABAQC6EeVS8GutCLN1Wyt+TLNA9B
xoMIQf/mYT5/GImbC4nf3rvSkcskdw/91ejtkhYr94eGRcs8s/r/EWaNmTa9CtrxtCIzaI
FpS0OtvXB/Kc041BK9CQUukl+aOUlg7uaGfuHZeMShoFpxD6CMTD0we+Clg6Lgej/vJE+k
uFDD0jgJJIVyn38m9ohJuUFVFe7Hw7cLaT+LhGHHPHs9y3qaexGywCsOEMUKMCaYMCsoI8
lSCftvIzTFUwu7i76F9rUFXYoAlt8vOMntm46OlQbrwa2nwSgFQdzIKArpSkMUHuGhExdV
IigXUB9EednZV8BV/M4P+ORf/RNDIw77PDdzpvAAAAAwEAAQAAAQBjcnshl/PEuHjJyV92
kmHf3lhsaznCq8I882OJQbNNCMwUubYGa1Z5k5bqGej8yf1R0u65CTMhJ9TvyDw5aY9PtN
4ZvB5CH+d8aFTlGY9WuE6vvU4sRNPtgv4lxQnX7B9YCaLczSIZUVBmglc/3kMuE/NRrRZS
VUmBClFgm8j1drD6A3fpfGEP1ozooWvqZ8an2f5pKee6U9NTik3UECyie9rhEEwFFJ/dxD
0knUMx5/a1aF1IU4VHoE2JQZS0VaccGKck+PRKiE1CNbEJdsEU7dZnwRNN0QBlS+sEVYZj
J/LK3RnoxIBqbSCiTkMNtcja8JDPGdRSjHK05WQOwdEJAAAAgEHFId3DVBTsGmg/mZe+N1
RyEpwLLK+e+PIMhNYECC7EEOrv2lynUsFKSrB30inOm99HW5z4v7R3xeb7d3reLoubdsCD
R8Bp0lU2gtjKbeuupn6hhFn/lSys4M4rU4raLEtOVVqUEpSeGCUZSsmAqeO1oCbVjNby0P
6CtQB89wz6AAAAgQDuzbiMLPF9QcuOK+E3ZLyXth47M4oArr72xImRvOSOuhgGEE/e1kf1
gNSpWCVR8BAgdlCQF1M+kffAgk2uxcWoPSHZdXvza3yPA0pIeb/v6x3NaGvmYMITayIg92
TJKoQavKizPyZK6V7m8GpnX7cBbuIfdPjxAadaKlhi2yAMuwAAAIEAx3gLL9eOzYENox+V
Fy3polncOAMA9Oi8UhBnY/cUOv8XksBQPBQDxifHgdT7iTjUMVEk0dL/YkzWr5ZsWJ81tj
2hM4wXbGOM9Kz1kfUCBkVGtgFXbxWCe/W3mY3Kngw5VzHjsDuEb14v5YmMkc7bic/RQbNx
ak/vgF0CAC63Z90AAAAQcDR5bDBhZEBTWU1CT0xJQwECAw==
>>> exit()
```
{% endcode %}

## BruteForce

```basic
hydra -L user.txt -P pass.txt ssh://10.10.10.10
crackmapexec ssh 10.10.10.10 -u patrick -p /usr/share/wordlists/rockyou.txt
```

## Crack Weak SSH Key

[https://0xdf.gitlab.io/2020/04/08/htb-lame-more.html#weak-ssh-key](https://0xdf.gitlab.io/2020/04/08/htb-lame-more.html#weak-ssh-key)

### SSH Error

```basic
└─$ ssh admin@10.10.10.10
Unable to negotiate with 10.10.10.10 port 22: no matching key exchange method found. Their offer: diffie-hellman-group-exchange-sha1,diffie-hellman-group14-sha1,diffie-hellman-group1-sha1

└─$ ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 admin@10.10.10.10
```

## Port Knocking

{% code overflow="wrap" %}
```bash
└─$ for i in 571 290 911; do nmap -Pn --max-retries 0 -p $i 10.10.10.10 && sleep 1; done
```
{% endcode %}

Let knock the port to open with for loop. [https://wiki.archlinux.org/title/Port\_knocking](https://wiki.archlinux.org/title/Port_knocking)

## Rbash Bypass

[https://www.hacknos.com/rbash-escape-rbash-restricted-shell-escape/](https://www.hacknos.com/rbash-escape-rbash-restricted-shell-escape/)

[https://www.exploit-db.com/docs/english/44592-linux-restricted-shell-bypass-guide.pdf](https://www.exploit-db.com/docs/english/44592-linux-restricted-shell-bypass-guide.pdf)
