[译]利用 SystemPropertiesAdvanced.exe 和 DLL 劫持绕过 UAC

> Auto-elevating binaries are a good source of bypasses for Windows User Account Control (UAC). “SystemPropertiesAdvanced.exe” and other SystemProperties* binaries can be used to bypass UAC on Windows Server 2019 via DLL hijacking.
> 
> 自动提权二进制（可执行文件）是用来绕过 Windows User Account Control (UAC) 的好东西。“SystemPropertiesAdvanced.exe” and 其他 SystemProperties* 二进制（可执行文件）可在 Windows Server 2019 上通过 DLL 劫持被用来绕过 UAC。

findstr can used to examine the auto-elevate behaviour within the embedded manifest:

```
findstr /C:"<autoElevate>true" C:\Windows\SysWOW64\SystemPropertiesAdvanced.exe
```

Alternatively, sigcheck can be used to dump the manifest.

```
sigcheck.exe -m C:\Windows\SysWOW64\SystemPropertiesAdvanced.exe
```

After setting the Procmon filter, the auto-elevating SysWOW64 binaries are executed.

This results in a fair amount of output, and so additional Procmon filters to exclude DLL paths starting with “C:\Windows” and “C:\Program Files” can be applied. Of note, the binary “C:\Windows\SysWOW64\SystemPropertiesAdvanced.exe” attempts to load the DLL “srrstr.dll” from the WindowsApps folder. As this folder is included in the PATH variable, Windows will search this location for the required DLL.

This folder is writeable by unpriviledged users, in order to allow installation of Apps from the Microsoft Store.

A DLL to spawn calc.exe is crafted (tested with DllMain) and saved to the WindowsApps folder. “SystemPropertiesAdvanced.exe” is executed again (from a Medium Integrity Level command prompt), and calc.exe is spawned as a High Integrity Level process.

In testing, this DLL hijack / UAC bypass also affects other SysWOW64 SystemProperties* binaries:

+ SystemPropertiesComputerName.exe
+ SystemPropertiesHardware.exe
+ SystemPropertiesProtection.exe
+ SystemPropertiesRemote.exe

Setting UAC policy to “Always Notify” mitigates this issue.



