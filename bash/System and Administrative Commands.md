# Chapter 12: System and Administrative Commands

## Understanding System Administration through Shells

System and administrative commands manage the core functions of Unix/Linux systems: user accounts, file permissions, terminal behavior, and system monitoring. These commands typically require elevated privileges (root/sudo) and directly interact with system files and kernel interfaces.

**Key insight:** System administration is fundamentally about **identity** (users, groups) and **access control** (permissions, capabilities). Understanding these commands means understanding how systems enforce security and accountability.

### Why These Commands Matter

Most scripting tasks eventually need:
- User authentication (`su`, `sudo`)
- Account creation/management (`useradd`, `usermod`)
- Permission changes (`chown`, `chgrp`)
- Terminal control (`stty`, `tty`)
- Session tracking (`who`, `w`, `last`)

These aren't just utilities—they're your interface to system security and administration.

---

## 12.1. Users and Groups Management

### Understanding Unix Identity Model

Unix security is built on a simple model:
1. **Users**: Individual identity (UID, username, home directory, shell)
2. **Groups**: Collections of users with shared permissions (GID, group members)
3. **Ownership**: Files/processes belong to a user and group
4. **Permissions**: Restrict access based on user/group/other

### users: List Currently Logged-In Users

Display all users currently logged into the system (no details—just names):

```bash
$ users
bozo bozo bozo alice bob
# bozo is logged in 3 times (3 terminal sessions)
```

**Comparison:**
- `users`: Just names, duplicate entries for multiple sessions
- `who`: More detail (terminal, login time)
- `w`: Most detail (terminal, commands running)

### groups: List User Group Memberships

Show current user's group membership:

```bash
$ groups
bozo sudo cdrom audio

$ groups alice
alice : alice sudo docker
# alice belongs to alice, sudo, and docker groups
```

### chown and chgrp: Changing File Ownership

The `chown` command changes file owner. Only the file owner or root can change ownership:

```bash
# Change owner
chown alice file.txt          # alice becomes owner

# Change owner and group
chown alice:staff file.txt    # alice:staff

# Recursive change (directories)
chown -R alice:staff /home/alice/documents

# Verify change
ls -l file.txt
# -rw-r--r-- 1 alice staff 1234 Jan 1 12:00 file.txt
```

The `chgrp` command changes group ownership:

```bash
# Change group
chgrp developers file.txt

# Recursive
chgrp -R developers /project

# Verify
ls -l file.txt
# -rw-r--r-- 1 alice developers 1234 Jan 1 12:00 file.txt
```

### useradd and userdel: User Account Management

**Creating a user account:**

```bash
# Basic user creation
useradd username

# Create with home directory and shell
useradd -m -s /bin/bash -d /home/username username

# Create with specific UID and GID
useradd -u 1001 -g staff -m username

# Options:**
#   -m: Create home directory
#   -s: Specify login shell
#   -d: Specify home directory path
#   -u: Specify UID
#   -g: Specify primary group
#   -G: Specify supplementary groups
```

**Removing a user account:**

```bash
# Remove user (keep home directory)
userdel username

# Remove user AND home directory
userdel -r username

# Verify deletion
id username  # Returns error: no such user
```

### Example 12-1 (Enhanced): New User Creation Script

```bash
#!/bin/bash
# add-user.sh: Create new user with validation

if [ "$EUID" -ne 0 ]; then
  echo "✗ This script requires root privileges"
  exit 1
fi

echo "=== New User Creation ==="
echo

read -p "Username: " username
read -p "Full name: " fullname
read -p "Shell [/bin/bash]: " shell
shell="${shell:-/bin/bash}"

# Validate shell exists
if [ ! -x "$shell" ]; then
  echo "✗ Shell not found or not executable: $shell"
  exit 1
fi

# Check if user already exists
if id "$username" &>/dev/null; then
  echo "✗ User already exists: $username"
  exit 1
fi

# Create user
useradd -m -s "$shell" -c "$fullname" "$username"
if [ $? -ne 0 ]; then
  echo "✗ Failed to create user"
  exit 1
fi

# Set password
echo
echo "Set password for $username"
passwd "$username"

# Verify creation
echo
echo "✓ User created successfully:"
id "$username"

exit 0
```

### usermod and groupmod: Modifying Accounts

**Modifying user accounts:**

```bash
# Add user to supplementary group
usermod -aG sudo alice

# Change login shell
usermod -s /bin/zsh bob

# Change home directory
usermod -d /mnt/newhome charlie

# Change username
usermod -l newname oldname

# Lock account (disable login)
usermod -L username

# Unlock account
usermod -U username

# Set account expiration date
usermod -e 2025-12-31 username
```

**Modifying groups:**

```bash
# Rename group
groupmod -n newname oldname

# Change group GID
groupmod -g 1001 groupname
```

### id: Show User and Group IDs

Display current user's UIDs, GIDs, and groups:

```bash
$ id
uid=501(alice) gid=501(alice) groups=501(alice),4(adm),24(cdrom),27(sudo)

# Show specific user
$ id bob
uid=502(bob) gid=502(bob) groups=502(bob)

# Show only effective UID
$ id -u
501

# Show only effective GID
$ id -g
501
```

---

## 12.2. User Session Information

### who: Logged-In Users with Details

Show detailed information about logged-in users:

```bash
$ who
alice    tty1        2025-01-28 09:30
bob      pts/0       2025-01-28 10:15
charlie  pts/1       2025-01-28 10:20

# With detailed output
$ who -l
alice    tty1        2025-01-28 09:30 (00:45)
bob      pts/0       2025-01-28 10:15 (00:20)

# Show current terminal
$ who -m
alice    pts/5       2025-01-28 10:45
```

**Output columns:**
- Username
- Terminal (tty# or pts/#)
- Login date and time
- Idle time (with `-l`)

### whoami: Current Username

Simple command to show current user:

```bash
$ whoami
alice

# Useful in scripts
if [ "$(whoami)" != "root" ]; then
  echo "This script requires root"
  exit 1
fi
```

### w: Extended User and Process List

Show logged-in users and what they're doing:

```bash
$ w
 10:45:30 up 2:30, 3 users, load average: 0.12, 0.15, 0.18
USER    TTYP    FROM          LOGIN@  IDLE   JCPU   PCPU  WHAT
alice   tty1    -             09:30   15m    1:20   0.45s bash
bob     pts/0   192.168.1.100 10:15   5m     0.30s  0.02s sshd
charlie pts/1   remote.com    10:20   2m     0.15s  0.01s man

# Show only specific user
$ w alice
```

### last: Login History

Show login and logout history:

```bash
$ last
alice    pts/0                        Tue Jan 28 10:45 - 10:50 (00:05)
bob      pts/1                        Tue Jan 28 10:30 - 10:40 (00:10)
charlie  tty1                         Tue Jan 28 09:00 - 09:45 (00:45)
reboot   system boot                  Tue Jan 28 08:00 - 10:50 (02:50)

# Show specific user's history
$ last alice

# Show last 10 entries
$ last -n 10

# Show failed login attempts
$ lastb
```

### Example 12-2 (Enhanced): User Activity Monitor

```bash
#!/bin/bash
# user-activity.sh: Monitor and report user sessions

echo "=== User Activity Report ==="
echo "Generated: $(date)"
echo

echo "--- Currently Logged In Users ---"
w -h | awk '{print $1}' | sort | uniq -c

echo
echo "--- Last 5 Logins ---"
last -n 5 | grep -v "^$"

echo
echo "--- User Login Time Summary ---"
ac -p | tail -10

exit 0
```

---

## 12.3. Password Management

### passwd: User Password Control

Change password for current user or another user (if root):

```bash
# Change own password
passwd
# Prompts for current password, then new password

# Root change another user's password
passwd alice

# Options:
#   -l: Lock account (disable login)
#   -u: Unlock account
#   -d: Delete password (no login required)
#   -e: Force password change on next login
#   -i: Disable after days of inactivity
```

### Example 12-3 (Enhanced): Secure Password Setting Script

```bash
#!/bin/bash
# set-password.sh: Set password with validation (root only)

if [ "$EUID" -ne 0 ]; then
  echo "✗ This script requires root privileges"
  exit 1
fi

if [ $# -ne 2 ]; then
  echo "Usage: $0 username password"
  exit 1
fi

username="$1"
newpassword="$2"

# Verify user exists
if ! id "$username" &>/dev/null; then
  echo "✗ User not found: $username"
  exit 1
fi

# Validate password strength
if [ ${#newpassword} -lt 8 ]; then
  echo "✗ Password too short (minimum 8 characters)"
  exit 1
fi

# Set password via stdin
echo "$newpassword" | passwd --stdin "$username" 2>/dev/null

if [ $? -eq 0 ]; then
  echo "✓ Password changed for $username"
else
  echo "✗ Failed to change password"
  exit 1
fi

# Force password change on next login
passwd -e "$username"
echo "✓ User must change password on next login"

exit 0
```

---

## 12.4. Terminal Management and Control

### tty: Show Terminal Device

Display the terminal device associated with current session:

```bash
$ tty
/dev/pts/1

# In a non-interactive context (e.g., cron)
$ tty
not a tty

# Use in scripts to detect if interactive
if [ -t 0 ]; then
  echo "Running interactively"
else
  echo "Running non-interactively"
fi
```

### stty: Terminal Settings Control

View and modify terminal behavior—critical for scripts that read sensitive input or control terminal behavior:

```bash
# View all current settings
stty -a

# Save current settings
saved_settings=$(stty -g)

# Restore settings
stty "$saved_settings"

# Disable echo (for password input)
stty -echo
read -p "Password: " password
stty echo

# Disable canonical mode (read keystrokes immediately)
stty -icanon
read -n1 key
stty icanon

# Set erase character
stty erase '#'

# Reset to defaults
stty sane
```

### Example 12-4 (Enhanced): Safe Password Input Function

```bash
#!/bin/bash
# read-password.sh: Read password without echoing

read_password() {
  local prompt="${1:-Password: }"
  local password
  
  # Save terminal settings
  local saved_settings=$(stty -g)
  
  # Disable echo
  stty -echo -echonl
  
  read -p "$prompt" password
  
  # Restore settings
  stty "$saved_settings"
  
  echo "$password"
}

echo "Enter credentials:"
username=$(read -p "Username: " username; echo "$username")
password=$(read_password "Password: ")

echo
echo "Username: $username"
echo "Password length: ${#password}"

exit 0
```

### Example 12-5 (Enhanced): Interactive Key Reading

```bash
#!/bin/bash
# keypress-menu.sh: Read single keypress for menu selection

show_menu() {
  echo "=== Menu ==="
  echo "1. Option One"
  echo "2. Option Two"
  echo "3. Option Three"
  echo "q. Quit"
}

read_single_key() {
  local saved_settings=$(stty -g)
  stty -icanon -echo  # Raw mode: read immediately, no echo
  
  local key=$(head -c1)
  
  stty "$saved_settings"
  
  echo "$key"
}

while true; do
  show_menu
  
  echo -n "Your choice: "
  choice=$(read_single_key)
  
  echo  # Newline after single character
  
  case "$choice" in
    1) echo "You chose: Option One" ;;
    2) echo "You chose: Option Two" ;;
    3) echo "You chose: Option Three" ;;
    q) echo "Goodbye"; exit 0 ;;
    *) echo "Invalid choice" ;;
  esac
  
  echo
done
```

### Terminal Modes

**Canonical mode (default):**
- Characters buffered until Enter pressed
- Backspace, Ctrl-U (kill line) work automatically
- Line discipline handles editing

**Raw mode (-icanon):**
- Every keypress sent immediately to program
- No automatic line editing
- Program responsible for all input handling

```bash
# Canonical mode (line buffered)
stty icanon
read line  # Reads until Enter

# Raw mode (character buffered)
stty -icanon
read -n1 char  # Reads single character immediately
```

### setterm: Terminal Appearance Control

Modify terminal appearance and attributes:

```bash
# Hide/show cursor
setterm -cursor off
# ... commands ...
setterm -cursor on

# Text formatting
setterm -bold on
echo "Bold text"
setterm -bold off

setterm -reverse on
echo "Reverse video"
setterm -reverse off

# Colors (if terminal supports)
setterm -foreground white -background black
```

---

## 12.5. User Activity Logging

### ac: Accumulated Connect Time

Show total login time for users:

```bash
$ ac
total       68.08

$ ac -p
user1        25.30
user2        42.78

$ ac -d
Jan  1       15.50
Jan  2       52.58
```

### logname: Login Name

Show the name the user logged in with (from `/var/run/utmp`):

```bash
$ logname
alice

# Different from whoami in some cases (after su)
$ su bob
$ whoami
bob

$ logname
alice  # Still shows original login name
```

---

## Best Practices and Common Patterns

### 1. Always Check Privileges Before Administrative Operations

```bash
#!/bin/bash

require_root() {
  if [ "$EUID" -ne 0 ]; then
    echo "✗ This script requires root privileges"
    echo "   Run with: sudo $0"
    exit 1
  fi
}

require_root

# Now safe to do privileged operations
```

### 2. Verify User Existence Before Operations

```bash
# Good: Check before modification
if ! id "$username" &>/dev/null; then
  echo "User not found: $username"
  exit 1
fi

usermod -aG sudo "$username"

# Bad: Assumes user exists
usermod -aG sudo "$username"  # May fail silently
```

### 3. Always Save and Restore Terminal Settings

```bash
#!/bin/bash

cleanup() {
  stty "$saved_tty"  # Restore no matter how script exits
}

trap cleanup EXIT

saved_tty=$(stty -g)

stty -echo  # Disable echo
read password
# cleanup runs automatically on exit
```

### 4. Use sudo for Privilege Escalation Instead of su

```bash
# Good: sudo logs commands, respects sudoers
sudo useradd newuser

# Avoid: su requires password, no audit log
su - root -c "useradd newuser"
```

### 5. Handle Multi-User Environments

```bash
# Check for running processes before deleting user
if pgrep -u "$username" &>/dev/null; then
  echo "User has running processes. Kill them first."
  pkill -u "$username"
  sleep 2
fi

userdel -r "$username"
```

---

## 10 Core Programming Concepts from System Administration

### 1. **Identity and Access Control**
Users and groups implement the fundamental security model—identity defines access rights. Understanding UID/GID is essential to system security.

```bash
uid=501(alice) gid=501(alice) groups=501(alice),4(adm),27(sudo)
# Identity hierarchy: user → primary group → supplementary groups
```

### 2. **Privilege Escalation and Capability Dropping**
`sudo` and `su` demonstrate controlled privilege elevation—granting temporary elevated access while maintaining audit trails and capability restrictions.

### 3. **Session Management**
`who`, `w`, `last` show how systems track sessions—each login is a session with identity, terminal, duration, and activity.

### 4. **Terminal as State Machine**
`stty` reveals terminals as finite state machines with modes (canonical/raw), settings (echo on/off), and behaviors that scripts must understand and manage.

### 5. **TTY and Pseudo-Terminal Concepts**
`tty`, `pts`, `/dev/tty` demonstrate how terminal I/O is abstracted—real terminals (ttyN) vs. pseudo-terminals (ptsN) for SSH, screen, etc.

### 6. **Interactive vs. Non-Interactive Execution**
Detecting whether running interactively (`[ -t 0 ]`) determines what prompts/interactions are appropriate—fundamental to robust scripts.

### 7. **State Preservation and Restoration**
Saving terminal settings before modifications and restoring on exit is a general pattern for managing stateful resources.

```bash
saved_state=$(stty -g)
trap "stty "$saved_state"" EXIT
```

### 8. **Auditing and Accountability**
Login history (`last`, `ac`, `w`) provides accountability—tracking who did what when. This is critical for security and forensics.

### 9. **Password and Authentication Handling**
Secure password input (disabling echo, immediate reading) demonstrates security best practices for sensitive data.

### 10. **User and Group Hierarchies**
The multi-level identity model (user → primary group → supplementary groups → other) creates flexible permission hierarchies used throughout Unix systems.

---

## Summary Table: Common Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `useradd` | Create user account | `useradd -m -s /bin/bash alice` |
| `userdel` | Remove user account | `userdel -r alice` |
| `usermod` | Modify user account | `usermod -aG sudo alice` |
| `passwd` | Change password | `passwd alice` |
| `chown` | Change file owner | `chown alice file.txt` |
| `chgrp` | Change group owner | `chgrp staff file.txt` |
| `id` | Show user/group IDs | `id alice` |
| `groups` | Show group membership | `groups alice` |
| `who` | List logged-in users | `who` |
| `whoami` | Current username | `whoami` |
| `w` | User and process list | `w` |
| `last` | Login history | `last` |
| `tty` | Show terminal device | `tty` |
| `stty` | Terminal settings | `stty -a` |
| `setterm` | Terminal appearance | `setterm -bold on` |
| `su` | Switch user | `su - alice` |
| `sudo` | Execute as root | `sudo useradd alice` |

---

## Exercises

1. **User Creation Script**: Write a script that creates multiple users from a CSV file, setting passwords and group memberships.

2. **Account Auditor**: Build a script that checks for disabled accounts, verifies home directories exist, and reports anomalies.

3. **Terminal Control Form**: Create an interactive form using `stty` that reads menu selections without requiring Enter.

4. **Session Monitor**: Write a script that tracks user logins/logouts by monitoring `who` output over time.

5. **Password Validator**: Build a password strength checker that enforces complexity requirements (length, mixed case, numbers, special chars).

6. **Cleanup Automation**: Create a script that safely removes user accounts, killing running processes and archiving home directories.

7. **SSH Key Manager**: Write a script to manage SSH authorized_keys for multiple users with validation.

8. **Audit Report**: Generate a daily report of user activities, failed login attempts, and sudo usage from system logs.

---

## Summary

**User and Group Management:**
- **`useradd`, `userdel`, `usermod`**: Account lifecycle management
- **`passwd`**: Password control (setting, locking, unlocking)
- **`chown`, `chgrp`**: File ownership and group assignment
- **`id`, `groups`**: Identity information and verification

**Session Tracking:**
- **`who`, `w`, `last`**: User session monitoring
- **`ac`**: Accumulated login time
- **`logname`**: Show login identity

**Terminal Control:**
- **`tty`**: Detect terminal and interaction mode
- **`stty`**: Terminal settings (echo, canonical mode, characters)
- **`setterm`**: Terminal appearance and attributes

**Security:**
- **`sudo`**: Privilege escalation with auditing
- **`su`**: User switching (less preferred than sudo)
- **Session management**: Tracking identity and access

**Key insight**: System administration commands are your interface to Unix identity and access control. Understanding them means understanding how systems enforce security, track accountability, and manage the relationships between users, files, and processes. These concepts are fundamental not just to shell scripting, but to all systems programming.
