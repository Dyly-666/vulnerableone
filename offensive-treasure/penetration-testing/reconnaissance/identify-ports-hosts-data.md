---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/reconnaissance/identify-ports-hosts-data
---

# Identify Ports/Hosts/Data

## Full TCP Scan

```basic
# Nmap_Scan_TCP
nmap -Pn -n -v $ip -p- --min-rate 5000 -oA $ip

# Grep_PortOpen
cat nmap.txt | grep open | awk -F / '{print $1}' | sed -z "s/\n/,/g" | head -c-1
grep -oP '\d{1,5}/open' $ip.gnmap | awk -F/ '{print $1}' | sed -z 's/\n/,/g' | head -c-1

# Enumerate_Service
nmap -Pn -n -v -pxxx $ip -T4 -sC -sV -oX $ip.xml; xsltproc $ip.xml -o $ip.html
```

## UDP Scan

```basic
# Nmap_Scan_UDP
sudo nmap -Pn -n -v $ip -sU -p- --min-rate 5000 -oA $ip

# Grep_PortOpen
grep -oP '\d{1,5}/open' $ip.gnmap | awk -F/ '{print $1}' | sed -z 's/\n/,/g' | head -c-1

# Enumerate_Service
sudo nmap -Pn -n -v -sU -p137 $ip -T4 -sC -sV -oX $ip.xml
```

## Grep Port Open with Powershell

```powershell
#Save nmap open port into > nmap.txt file

$file = "C:\test.txt"
$filecontent = Get-Content $file
$newfile = Foreach($line in $filecontent){
	($line.split('/'))[0]
}
$newfile | Set-Content $file
(Get-content $file) -join ","

PS C:\> .\split.ps1
135,139,445,3389
```

## Nmap Script

```bash
# List down all the nmap script
grep -r categories /usr/share/nmap/scripts/*.nse 
```

<figure><img src="../../../.gitbook/assets/image (229).png" alt=""><figcaption></figcaption></figure>

```bash
# Search base on category
grep -r categories /usr/share/nmap/scripts/*.nse | grep safe | awk -F: '{print $1}'

# Scan with Safe Script Only
sudo nmap --script safe -Pn -n -v -pxxx $ip -oX $ip-safe.xml

# Scan with "Safe and Default" Script
```

<figure><img src="../../../.gitbook/assets/image (226).png" alt=""><figcaption></figcaption></figure>

```bash
# Search for anything in the between quote 
grep -r categories /usr/share/nmap/scripts/*.nse | grep -oP '".*?"' | sort -u 
```

<figure><img src="../../../.gitbook/assets/image (227).png" alt=""><figcaption></figcaption></figure>

## Host Discover with Fping

```basic
fping -q -a -r 1 < subnet1.txt
fping -a -g 10.10.10.0/24 2>/dev/null
```

## Ping Sweep in Window

```batch
# Powershell
for ($i = 1; $i -lt 255; $i++) {
    Test-Connection "10.10.10.$i" -Count 1 -ErrorAction SilentlyContinue
}

# CMD
for /L %i in (1,1,255) do @ping -n 1 -w 200 10.10.10.%i > nul && echo 10.10.10.%i is up.
```

## Ping Sweep with Bash

```bash
for i in {1..255}; do (ping -c 1 192.168.1.${i} | grep "bytes from" &); done
```

## Ping Sweep with Nmap

```basic
sudo nmap -sP -PE 10.10.10.0/24
sudo nmap -PR -sn 10.10.10.0/24    #Arp
sudo nmap -PE -sn 10.10.10.0/24    #ICMP
```

## Generate IP in Window

```batch
# CMD
for /L %x in (1, 1, 10) do echo 192.168.1.%x >> ip.txt

# Bat File
@echo OFF
for /L %%x in (1, 1, 10) do echo 192.168.1.%%x >> ip.txt
```

## Generate list of IP by bash

```bash
#!/bin/bash

echo "1- Generate Sample IPs [10.10.10.x] "
echo "2- Generate Sample IPs [10.10.x.1] "
echo "3- Generate Sample IPs [10.x.10.1] "
read choice

if [ "$choice" == "1" ];
then
 echo "Enter your IP Range: "
 read IP
	for i in {1..254};
	do
		echo $IP.$i >> $IP.txt;
	done
fi

if [ "$choice" == "2" ];
then
 echo "Enter your first block: "
 read IP
 echo "Enter your fourth block: "
 read IP1
	for i in {1..254};
	do
		echo $IP.$i.$IP1 >> $IP.x.$IP1.txt;
	done
fi

if [ "$choice" == "3" ];
then
 echo "Enter your first block: "
 read IP
 echo "Enter your third block: "
 read IP1
 echo "Enter your Fourth block: "
 read IP2
	for i in {1..254};
	do
		echo $IP.$i.$IP1.$IP2 >> $IP.x.$IP1.$IP2.txt;
	done
fi
```

```bash
└─$ ./gen.sh
1- Generate Sample IPs [10.10.10.x] 
2- Generate Sample IPs [10.10.x.1] 
3- Generate Sample IPs [10.x.10.1] 
1
Enter your IP Range: 
10.10.10
```

## Ipv6 Discovery

```basic
sudo atk6-alive eth0
```
