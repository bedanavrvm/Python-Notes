# Chapter 10: Internal Commands and Builtins

Bash provides numerous internal commands (builtins) that are executed directly by the shell rather than spawning external processes. Understanding these commands is essential for writing efficient and robust scripts.

## 10.1. Script Control Commands

### `exit [n]`

Unconditionally terminates a script, optionally returning an exit status `n` to the shell.

**Key points:**
- `exit 0` indicates successful execution
- `exit 1` (or any non-zero value) indicates failure
- Without an argument, returns the exit status of the last command executed
- Good practice: end all but the simplest scripts with `exit 0`

```bash
#!/bin/bash

if [ ! -f "$1" ]; then
  echo "File not found: $1"
  exit 1  # Exit with error status
fi

echo "Processing $1..."
# ... do something ...

exit 0  # Exit with success status
```

### `exec`

Replaces the current process with a specified command. Unlike normal commands which fork a child process, `exec` directly replaces the shell process itself.

**Key characteristics:**
- The forked child process is **not** created
- When the exec'ed command terminates, the script exits
- Code after `exec` **never executes**
- Can reassign file descriptors (e.g., `exec < file` redirects stdin)

### Example 10-1: Effects of `exec`

```bash
#!/bin/bash
# exec-demo.sh

echo "Before exec"
echo "This message appears."
echo

exec echo "Executed with exec"

# Everything below this line never executes
echo "This line never prints"
exit 99  # This exit never happens
```

**Output:**
```
Before exec
This message appears.

Executed with exec
```

Exit status will not be 99 because `exec` replaced the script process.

### Example 10-2: Self-Referencing Script

```bash
#!/bin/bash
# self-exec.sh: Script that replaces itself

echo
echo "This line repeats (press Ctrl-C to stop)"
echo "Current PID: $$"
echo "==============================================="

sleep 1

# exec replaces this script with a new instance of itself
exec "$0"

# This never executes
echo "This will never print"
```

When run, this script continuously spawns new instances of itself, each with the same PID (because exec replaces the process, not spawning a child).

### Reassigning File Descriptors with `exec`

```bash
# Redirect stdin from a file
exec < input.txt
read line  # Reads from input.txt instead of keyboard

# Redirect stdout to a file
exec > output.txt
echo "This goes to output.txt"

# Redirect stderr to a file
exec 2> errors.log
echo "Error message" >&2  # Goes to errors.log

# Restore stdout
exec > /dev/tty  # /dev/tty is the terminal
echo "Back to terminal"
```

---

## 10.2. Shell Option and Debugging Commands

### `shopt`

Controls shell options on the fly. Common options:

| Option | Effect |
|--------|--------|
| `cdspell` | Correct minor typos in `cd` directory names |
| `nocaseglob` | Pathname expansion is case-insensitive |
| `nullglob` | Allow pattern to expand to nothing (instead of literal) |
| `extglob` | Enable extended pattern matching |

```bash
#!/bin/bash

# Enable cd spelling correction
shopt -s cdspell

cd /hpme  # Typo: /home
pwd       # /home (corrected!)

# Enable extended globbing
shopt -s extglob
files=!(*.txt)  # Match everything except .txt files

# List all set options
shopt
```

**Usage:**
- `shopt -s option` — Set (enable) option
- `shopt -u option` — Unset (disable) option
- `shopt` — List all options

### `caller [expr]`

Outputs information about the calling context of a function.

```bash
#!/bin/bash

function_a() {
  echo "In function_a, caller info:"
  caller 0  # Info about immediate caller
}

function_b() {
  function_a
}

# Called from line 12
function_b

# Output: 12 function_b script.sh
# Line 12: called from function_b
# function_b: the calling function
# script.sh: the script name
```

---

## 10.3. Utility Commands

### `true` and `false`

Simple commands that return specific exit statuses without doing anything else.

```bash
# true returns 0 (success)
if true; then
  echo "true succeeded"
fi

# false returns 1 (failure)
if false; then
  echo "This won't print"
else
  echo "false failed (as expected)"
fi

# Endless loop pattern
while true
do
  read -p "Enter command (or 'q' to quit): " cmd
  [ "$cmd" = "q" ] && break
  eval "$cmd"
done
```

### `type [cmd]`

Identifies what a command is (builtin, function, alias, or external command).

```bash
bash$ type echo
echo is a shell builtin

bash$ type -a echo
echo is a shell builtin
echo is /bin/echo

bash$ type ls
ls is aliased to 'ls --color=auto'

bash$ type -a cd
cd is a shell builtin
cd is /usr/bin/cd

# Check if command exists
if type python3 &>/dev/null; then
  echo "Python 3 is installed"
fi
```

### `hash [cmds]`

Records command paths in the shell hash table for faster lookup.

```bash
# Show all hashed commands
hash

# Hash a specific command
hash -p /usr/bin/python3 python

# Clear the hash table
hash -r

# Remove specific command from hash
hash -d python
```

### `help [builtin]`

Display brief usage information for shell builtins.

```bash
help exit
help read
help for
```

---

## 10.4. Job Control Commands

Job control allows you to manage foreground and background processes.

| Command | Purpose |
|---------|---------|
| `jobs` | List background jobs |
| `fg [%job]` | Bring job to foreground |
| `bg [%job]` | Resume suspended job in background |
| `wait [job\|PID]` | Wait for job/process to complete |
| `suspend` | Suspend shell (like Ctrl-Z) |
| `disown [job]` | Remove job from job table |

### Job Identifiers

| Notation | Meaning |
|----------|---------|
| `%N` | Job number N |
| `%string` | Job whose command begins with string |
| `%?string` | Job whose command contains string |
| `%%` or `%+` | Current job (most recent) |
| `%-` | Previous job |
| `$!` | PID of last background process |

### Example 10-3: Job Management

```bash
#!/bin/bash
# job-demo.sh

echo "Starting background jobs..."

# Start background jobs
find / -name "*.log" > /tmp/logs.txt 2>&1 &
job1=$!

sleep 2 &
job2=$!

echo "Job list:"
jobs -l  # -l shows PIDs

echo
echo "Waiting for all background jobs to complete..."
wait $job1 $job2

echo "All jobs completed"
exit 0
```

### Example 10-4: Preventing Orphan Processes

```bash
#!/bin/bash
# wait-example.sh: Use wait to prevent orphan processes

ROOT_UID=0
E_NOTROOT=65

if [ "$UID" -ne "$ROOT_UID" ]; then
  echo "Must be root to run this script"
  exit $E_NOTROOT
fi

echo "Updating database (may take a while)..."

# Run background job
updatedb / &

# Don't exit until background job completes
wait

# Now it's safe to proceed
echo "Database update complete"
exit 0
```

### Background Job Hanging Issue

When a script runs a background command that writes to stdout, it may appear to hang:

```bash
#!/bin/bash
# BAD: May appear to hang
ls -l &
echo "Done."  # Prompt appears before ls output
```

**Solution 1: Use `wait`**
```bash
#!/bin/bash
ls -l &
echo "Done."
wait  # Wait for background process
```

**Solution 2: Redirect output**
```bash
#!/bin/bash
ls -l > /tmp/listing.txt 2>&1 &
echo "Done."
```

---

## 10.5. Process Management Commands

### `kill [signal] PID|job`

Send a signal to a process, usually to terminate it.

```bash
# Graceful termination (SIGTERM, signal 15)
kill $PID

# Force kill (SIGKILL, signal 9)
kill -9 $PID

# List all signals
kill -l

# Kill by job number
kill %1  # Kill job 1
```

### Example 10-5: Self-Terminating Script

```bash
#!/bin/bash
# self-destruct.sh: Script that kills itself

echo "Starting script..."
sleep 2

echo "About to self-terminate..."
kill $$  # Kill the script's own process

# This never executes
echo "This message never prints"
exit 0
```

**Output:**
```
Starting script...
About to self-terminate...
Terminated
```

Exit status will be 143 (128 + 15, where 15 is SIGTERM).

### `killall [signal] command`

Kill all processes matching a command name.

```bash
# Kill all instances of firefox
killall firefox

# Send SIGTERM first, then SIGKILL if needed
killall -9 node

# List signals
killall -l
```

---

## 10.6. Command Directives

### `command`

Disable aliases and functions, using only the builtin or external command.

```bash
#!/bin/bash

# If 'ls' is aliased to 'ls --color=auto'
ls  # Uses alias

# Bypass alias, use actual command
command ls  # Uses external /bin/ls
```

### `builtin`

Execute a builtin command, ignoring functions and external commands with the same name.

```bash
# If you have a function named 'cd'
cd /tmp  # Uses the function

# Force use of builtin
builtin cd /tmp  # Uses shell builtin
```

### `enable`

Enable or disable shell builtins.

```bash
# List all builtins
enable

# Disable the 'kill' builtin (use external /bin/kill instead)
enable -n kill

# Re-enable it
enable kill

# Load a builtin from a shared object file
enable -f ./builtin.so mybuiltin
```

---

## 10.7. Process Introspection

### `times`

Display CPU time statistics for the shell and its children.

```bash
bash$ times
0m0.020s 0m0.020s
0m1.234s 0m0.456s
```

Format: `user_time system_time user_child_time system_child_time`

Limited usefulness for shell scripts; more useful for profiling in general programming.

---

## 10.8. Script Recursion and Sourcing

### Self-Sourcing vs. Self-Calling

**Self-calling (recursion):**
```bash
#!/bin/bash
. "$0"  # Source the script itself
```

This causes the script to expand itself recursively, useful for clever tricks but not true recursion.

**Example: Pass Counter**

```bash
#!/bin/bash
# recursive-counter.sh

MAXPASS=5
pass_count=0  # Must initialize!

while [ "$pass_count" -lt "$MAXPASS" ]; do
  echo "Pass $pass_count"
  . "$0"  # Script sources itself
  let "pass_count += 1"
done

exit 0
```

---

## Summary Table: Common Builtins

| Command | Purpose |
|---------|---------|
| `exit [n]` | Exit script with status n |
| `exec cmd` | Replace shell with command |
| `shopt` | Set/unset shell options |
| `caller` | Show calling function info |
| `true` | Return success (0) |
| `false` | Return failure (1) |
| `type cmd` | Identify command type |
| `hash` | Manage command hash table |
| `help` | Show builtin documentation |
| `jobs` | List background jobs |
| `fg [%job]` | Foreground a job |
| `bg [%job]` | Background a job |
| `wait [job]` | Wait for process/job |
| `kill [signal] pid` | Terminate process |
| `killall cmd` | Kill by command name |
| `command cmd` | Disable aliases/functions |
| `builtin cmd` | Force builtin execution |
| `enable [-n] cmd` | Enable/disable builtin |
| `times` | Show CPU time stats |
| `suspend` | Suspend shell |
| `disown [job]` | Remove from job table |

---

## Exercises

1. **Write a script** that uses `exit` status codes to report different types of errors (file not found, permission denied, invalid argument).

2. **Create a function** that uses `caller` to print debugging information showing which line called the function.

3. **Implement a process monitor** using `jobs` and `wait` to manage multiple background tasks and ensure they complete.

4. **Build a command wrapper** using `command` and `type` that checks if a command exists before executing it.

5. **Write a script** that uses `shopt` to enable various shell options and demonstrate their effects.

6. **Create a self-sourcing script** (using `. "$0"`) that cleverly performs a task through recursive expansion.

---

## Best Practices

- **Always end scripts with `exit 0`** (or appropriate error code)
- **Use `wait` for background jobs** to prevent orphan processes
- **Check command existence with `type`** before using it
- **Use `command` to bypass aliases** when needed in scripts
- **Initialize variables** before incrementing (even with `+=`)
- **Use job control `%notation`** for clarity in scripts managing multiple processes
- **Document unusual techniques** like self-sourcing for maintainability
