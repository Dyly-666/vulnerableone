# Enable AD Account

If account locked we can unlock with this command&#x20;

{% code overflow="wrap" %}
```bash
bloodyAD -H 192.168.1.10 -d corp.local -u adminuser -p 'P@ssw0rd' set object john.doe lockoutTime -v 0
```
{% endcode %}

If account no active we can enable with this command&#x20;

{% code overflow="wrap" %}
```bash
bloodyAD --host DC-CC.city.local -d 'city.local' -u 'emma.hayes' -p '!Gemma4James!' remove uac 'sam.brooks' -f ACCOUNTDISABLE                     
```
{% endcode %}

If we want to disable it back

{% code overflow="wrap" %}
```bash
bloodyAD --host DC-CC.city.local -d 'city.local' -u 'emma.hayes' -p '!Gemma4James!' add uac 'sam.brooks' -f ACCOUNTDISABLE                     
```
{% endcode %}

if accountExpires we can set it to never&#x20;

{% code overflow="wrap" %}
```bash
bloodyAD -H 192.168.1.10 -d corp.local -u adminuser -p 'P@ssw0rd' \
  set object john.doe accountExpires -v 9223372036854775807
```
{% endcode %}
