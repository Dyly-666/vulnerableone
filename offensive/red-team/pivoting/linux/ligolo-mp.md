# Ligolo-MP

### Ligolo-MP

Started a Ligolo-MP server listener on port 11601.

{% code overflow="wrap" %}
```bash
sudo ligolo-mp server -laddr 0.0.0.0:11601
```
{% endcode %}

Push Enter/Connect

<figure><img src="../../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```
Push CTRL+N to create new Agent (use TAB for switch to new Linie)
Use you tun0 IP
Activate Ignore env proxy
OS= Windwos ARCH=amd64
Then Submit
```
{% endcode %}

<figure><img src="../../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

The tool create agent.bin File

<figure><img src="../../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

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



## Double Pivoting

First we need to generate agent for linux machine as a first jump box

<figure><img src="../../../../.gitbook/assets/Screenshot 2026-07-03 at 9.38.06 in the morning.png" alt=""><figcaption></figcaption></figure>

Then transfer that file into linux host and run it&#x20;

<figure><img src="../../../../.gitbook/assets/Screenshot 2026-07-03 at 9.39.03 in the morning.png" alt=""><figcaption></figcaption></figure>

Back to our ligolo-mp we got connection from mail then we need to add route

<figure><img src="../../../../.gitbook/assets/Screenshot 2026-07-03 at 9.39.41 in the morning.png" alt=""><figcaption></figcaption></figure>

Then we need to put that internal IP that we want to pivot

<figure><img src="../../../../.gitbook/assets/Screenshot 2026-07-03 at 9.41.12 in the morning.png" alt=""><figcaption></figcaption></figure>

After then we need to stat the relay&#x20;

<figure><img src="../../../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

Now we have connection to the internal IP&#x20;

<figure><img src="../../../../.gitbook/assets/Screenshot 2026-07-03 at 9.42.20 in the morning.png" alt=""><figcaption></figcaption></figure>
