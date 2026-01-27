# Chapter 12: System and Administrative Commands

System and administrative commands manage user accounts, groups, terminal settings, and system monitoring. These commands are typically invoked by root for system maintenance, user management, and emergency operations.

## Users and Groups Management

### users: List Logged-On Users

Display all currently logged-on users. Equivalent to `who -q`.

```bash
bash$ users
bozo bozo bozo bozo
```

### groups: List User Group Membership

Lists the current user and the groups they belong to.

```bash
bash$ groups
bozita cdrom cdwriter audio xgrp

bash$ echo $GROUPS
501
```

### chown, chgrp: Change File Ownership

**chown** changes file ownership. Only root can use this command to transfer ownership between users.

```bash
root# chown bozo *.txt
# Transfer ownership of all .txt files to user 'bozo'
```

**chgrp** changes group ownership. You must be the owner and a member of the destination group (or root).

```bash
chgrp --recursive dunderheads *.data
# Recursively change group ownership of all *.data files
```

### useradd, userdel: User Account Management

**useradd** adds a user account and creates a home directory:

```bash
root# useradd -m -s /bin/bash newuser
# -m: Create home directory
# -s: Set login shell
```

**userdel** removes a user account and associated files:

```bash
root# userdel -r username
# -r: Remove home directory and mail spool
```

### usermod: Modify User Accounts

Modify user attributes including password, group membership, expiration date, and account status.

```bash
usermod -aG sudo username
# Add user to sudo group

usermod -L username
# Lock user account (disable login)

usermod -U username
# Unlock user account
```

### groupmod: Modify Groups

Change group name and/or ID number:

```bash
groupmod -n newname oldname
# Rename group
```

### id: Show User and Group IDs

List real and effective user IDs, and group IDs of the current process.

```bash
bash$ id
uid=501(bozo) gid=501(bozo) groups=501(bozo),22(cdrom),80(cdwriter),81(audio)
```

### lid: List Group Membership

Show group(s) for a user or users in a group. Only root can use this command.

```bash
root# lid bozo
bozo(gid=500)

root# lid daemon
bin(gid=1)
daemon(gid=2)
```

## User Session Information

### who: List Logged-On Users with Details

Show all users logged into the system with terminal and login time.

```bash
bash$ who
bozo tty1 Apr 27 17:45
bozo pts/0 Apr 27 17:46
bozo pts/1 Apr 27 17:47

bash$ who -m
localhost.localdomain!bozo pts/2 Apr 27 17:49
```

### whoami: Current User Name

Show the current user name.

```bash
bash$ whoami
bozo
```

### w: Extended User and Process List

Show logged-on users and processes belonging to them.

```bash
bash$ w | grep startx
bozo tty1 - 4:22pm 6:41 4.47s 0.45s startx
```

### logname: Login Name

Show current user's login name as found in `/var/run/utmp`.

```bash
bash$ logname
bozo
```

### su: Substitute User

Run a program or shell as a different user.

```bash
su rjones
# Start shell as user 'rjones'

su - username
# Login shell (loads user's environment)
```

### sudo: Execute as Root or Another User

Run commands with elevated privileges. Controlled by `/etc/sudoers` file.

```bash
sudo cp /root/secretfile /home/bozo/secret
# Run copy command as root

sudo -u username command
# Run command as specific user
```

## Password Management

### passwd: Set or Change Passwords

```bash
passwd
# Change current user's password

passwd username
# Root changes another user's password

passwd -l username
# Lock user account

passwd -u username
# Unlock user account
```

### Example: Setting a New Password (Root Only)

```bash
#!/bin/bash
ROOT_UID=0
E_WRONG_USER=65
E_NOSUCHUSER=70

if [ "$UID" -ne "$ROOT_UID" ]; then
    echo "Only root can run this script."
    exit $E_WRONG_USER
fi

username=bozo
NEWPASSWORD=security_violation

grep -q "$username" /etc/passwd
if [ $? -ne 0 ]; then
    echo "User $username does not exist."
    exit $E_NOSUCHUSER
fi

echo "$NEWPASSWORD" | passwd --stdin "$username"
echo "User $username's password changed!"
exit 0
```

## User Activity Logging

### ac: User Login Time

Show users' logged-in time as read from `/var/log/wtmp`.

```bash
bash$ ac
total 68.08
```

### last: List Last Logins

Show last logged-in users, including remote logins and system reboots.

```bash
bash$ last reboot
reboot system boot 2.6.9-1.667 Fri Feb 4 18:18 (00:02)
reboot system boot 2.6.9-1.667 Fri Feb 4 15:20 (01:27)
```

### newgrp: Switch Group Membership

Change user's group ID without logging out.

```bash
newgrp groupname
# Switch to different group for file permissions
```

## Terminal Management

### tty: Show Terminal Device

Echo the filename of the current user's terminal.

```bash
bash$ tty
/dev/pts/1
```

### stty: Terminal Settings

Show and modify terminal settings.

```bash
stty -a            # Show all settings
stty -echo         # Disable terminal echo
stty echo          # Enable terminal echo
stty -icanon       # Disable canonical mode
stty erase '#'     # Set erase character
stty -g            # Save all settings
```

### Example: Secret Password Input

```bash
#!/bin/bash
echo
echo -n "Enter password "
read passwd
echo "password is $passwd"

stty -echo
# Turn off terminal echo

echo -n "Enter password again "
read passwd
echo
echo "password is $passwd"
echo

stty echo
# Restore terminal echo

exit 0
```

### Example: Keypress Detection

```bash
#!/bin/bash
old_tty_settings=$(stty -g)
stty -icanon

Keypress=$(head -c1)

echo
echo "Key pressed was \"$Keypress\"."
echo

stty "$old_tty_settings"
exit 0
```

### Terminal Modes

**Canonical Mode (default):**
- Characters buffered until ENTER is pressed
- Terminal has built-in line editor
- Supports backspace, line-kill (^U), etc.

**Raw Mode (-icanon):**
- Every keypress sent immediately to program
- No terminal line editing
- Special keys sent as characters

### setterm: Terminal Attributes

Set terminal attributes, controlling appearance and behavior.

```bash
setterm -cursor off
# Hide cursor

setterm -bold on
echo "This text is bold"
setterm -bold off
```

### tset: Initialize Terminal

Show or initialize terminal settings.

```bash
bash$ tset -r
Terminal type is xterm-xfree86.
Kill is control-U (^U).
Interrupt is control-C (^C).
```

### mesg: Terminal Write Access

Enable or disable write access to current terminal.

```bash
mesg n
# Disable write access

mesg y
# Enable write access

mesg
# Show current status
```

## Best Practices

### User Management Security

1. Use `sudo` for privilege escalation rather than `su` when possible
2. Always verify user exists before modifying with `grep /etc/passwd`
3. Lock unused accounts with `usermod -L` rather than deleting
4. Set appropriate group memberships using `usermod -aG group user`
5. Regular password audits with `lastlog` and `last` commands

### Terminal Safety in Scripts

1. Always save and restore terminal settings
2. Disable echo for sensitive input
3. Use canonical mode by default unless raw input is needed
4. Test terminal operations carefully before production deployment

## Exercises

1. **User Account Script:** Write a script to add a new user with home directory, set password from stdin, and add to specific groups.

2. **Login Monitor:** Create a script that logs all user logins/logouts by parsing output from `last` and `who`.

3. **Terminal Control:** Build an interactive form using `stty -icanon` that reads menu selections without ENTER.

4. **Session Tracker:** Write a script that reports which users are currently logged in and how long they've been idle.

5. **Password Security:** Create a wrapper for `passwd` that enforces password complexity requirements before setting.

6. **Terminal Saver:** Build a function that automatically saves/restores terminal settings in all scripts.

7. **Group Manager:** Write a script to bulk-add users to groups with validation and error handling.

8. **Activity Report:** Generate a daily report of user logins, logouts, and failed attempts.

## Summary

System and administrative commands provide essential user and group management capabilities:

- **User Management:** `useradd`, `userdel`, `usermod`, `chown`, `chgrp`
- **User Information:** `who`, `whoami`, `id`, `groups`, `last`
- **Password Control:** `passwd` with options for locking, unlocking, and deletion
- **Terminal Control:** `stty` and `setterm` for terminal behavior
- **Terminal Modes:** Canonical (buffered) vs. raw (immediate) input modes
- **Session Tracking:** `w`, `logname`, `ac` for user activity monitoring
