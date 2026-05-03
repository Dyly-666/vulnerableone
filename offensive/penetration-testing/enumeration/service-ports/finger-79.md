---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/enumeration/service-ports/finger-79
---

# Finger (79)

Finger displays information about users on a specified remote computer (typically a computer running UNIX) that is running the finger service or daemon. The remote computer specifies the format and output of the user information display.

Let check if there is any user logged on.

```basic
└─$ finger @10.10.10.10
No one logged on
```

On finger service could allow us to enumerate user.

```bash
└─$ finger root@10.10.10.10
Login       Name               TTY         Idle    When    Where
root     Super-User            pts/3        <Apr 24, 2018> sunday 

└─$ finger any@10.10.10.10
Login       Name               TTY         Idle    When    Where
any                   ???
```

We can download the tool [finger-enum](http://pentestmonkey.net/tools/user-enumeration/finger-user-enum) from pentest monkey.

```basic
./finger-user-enum.pl -U /usr/share/seclists/Usernames/Names/names.txt -t 10.10.10.10 
```

```basic
└─$ ./finger-user-enum.pl -U /usr/share/seclists/Usernames/Names/names.txt -t 10.10.10.10                                                                                                 1 ⨯
Starting finger-user-enum v1.0 ( http://pentestmonkey.net/tools/finger-user-enum )

 ----------------------------------------------------------
|                   Scan Information                       |
 ----------------------------------------------------------

Worker Processes ......... 5
Usernames file ........... /usr/share/seclists/Usernames/Names/names.txt
Target count ............. 1
Username count ........... 10177
Target TCP port .......... 79
Query timeout ............ 5 secs
Relay Server ............. Not used

######## Scan started at Thu Nov 11 06:17:00 2021 #########
sammy@10.10.10.10: sammy                 console      <Jul 31, 2020>..
sunny@10.10.10.10: sunny                 pts/2         11 Thu 17:04  10.10.14.31         .
```
