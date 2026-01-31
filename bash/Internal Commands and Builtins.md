# Chapter 10: Internal Commands and Builtins

## Understanding Bash Builtins

Bash builtins are commands executed directly by the shell itself rather than spawning external processes. This distinction is crucial:

- **External commands** (like `/bin/ls`): Shell must fork a child process, execute the command, wait for it to finish, then resume—expensive
- **Builtins** (like `echo`, `cd`, `read`): Execute directly within the shell process—fast and efficient

Understanding builtins separates casual shell scripting from professional shell programming. You'll write faster scripts, handle processes more effectively, and understand how shells work at a deeper level.

### Why Builtins Matter

Consider a simple loop that reads lines:

```bash
# Slow: Spawns a new `wc` process for each iteration
while read line; do
  words=$(echo "$line" | wc -w)
  echo "$words"
done < file.txt

# Fast: Uses builtin parameter expansion, no external processes
while read line; do
  echo "${#line[@]}"  # Count words without spawning process
done < file.txt
```

The builtin version is **dramatically faster** for large inputs because it avoids process creation overhead.

---

## 10.1. Script Control Commands

### Understanding Exit Status Codes

In Unix philosophy, **every command returns an exit status**—a number from 0 to 255 indicating success (0) or failure (non-zero). Exit codes are fundamental to shell scripting because they enable:

- Error detection and handling
- Conditional execution (`&&`, `||`)
- Script reliability testing
- Process orchestration

### `exit [n]`: Script Termination with Status

The `exit` command unconditionally terminates a script and returns an exit status `n` to the parent shell.

**Exit status conventions:**
- `0` = Success (always)
- `1` = General error
- `2` = Misuse of shell builtin
- `126` = Command invoked cannot execute
- `127` = Command not found
- `128+signal` = Fatal signal N (e.g., `128+9=137` for SIGKILL)

```bash
#!/bin/bash

if [ ! -f "$1" ]; then
  echo "Error: File not found: $1"
  exit 1  # Exit with error status
fi

if [ ! -r "$1" ]; then
  echo "Error: File not readable: $1"
  exit 2  # Different error code for permission issue
fi

echo "Processing $1..."
# ... do something ...

exit 0  # Explicit success (good practice)
```

**Why explicit exit codes matter:**

```bash
# Without explicit exit, script returns last command's status
script.sh
echo $?  # Might be anything (0, 1, etc.)

# With explicit exit, status is predictable
exit 5   # Always 5
```

**Practical pattern: Error handler with cleanup**

```bash
cleanup() {
  rm -f "$TEMP_FILE"
  rm -f "$LOCK_FILE"
}

trap cleanup EXIT  # cleanup runs on exit

if [ ! -f "$1" ]; then
  echo "Error: File not found"
  exit 1  # cleanup runs automatically
fi
```

### `exec`: Process Replacement (Advanced)

The `exec` command replaces the current shell process with another command. Unlike normal command execution which forks a child process, `exec` is a **process replacement**.

**Key semantics:**
- Current shell process is **replaced**, not forked
- When the exec'ed command terminates, the script terminates (no return to script)
- Code after `exec` **never executes** (because shell process no longer exists)
- Can reassign file descriptors before replacement

**Important:** This is rarely used in practice, but understanding it reveals how shells work.

### Example 10-1 (Enhanced): Effects of `exec`

```bash
#!/bin/bash
# exec-demo.sh: Demonstrate process replacement

echo "Before exec (PID: $$)"
echo "This message appears."
echo

# Everything below this point will be replaced
exec echo "Executed with exec (PID: $$)"

# NEVER EXECUTES - shell process has been replaced
echo "This line never prints"
exit 99  # This exit never happens
```

**Output:**
```
Before exec (PID: 1234)
This message appears.

Executed with exec (PID: 1234)
```

Notice: Same PID because `exec` replaced the process, not spawned a child.

### Example 10-2 (Enhanced): Self-Referencing Script with exec

```bash
#!/bin/bash
# self-exec.sh: Infinite self-spawning script (dangerous!)

echo "Script starting (PID: $$)"
echo "This demonstrates process replacement"
echo "=========================================="

sleep 1

# exec replaces this script process with a new instance of itself
# But since it's the same script, it looks infinite
exec "$0"

# This never executes
echo "This will never print"
```

**Warning:** Each execution replaces the previous one immediately. This is not a true loop—the PID never changes because `exec` replaces the process.

**Safer alternative using a loop:**

```bash
#!/bin/bash
# loop-demo.sh: Use loop instead of exec

for i in {1..3}; do
  echo "Iteration $i (PID: $$)"
  sleep 1
done

echo "Loop complete"
exit 0
```

### Reassigning File Descriptors with `exec`

One practical use of `exec` is reassigning file descriptors before executing a command:

```bash
#!/bin/bash
# fd-redirect.sh: File descriptor manipulation

# Redirect stdin from a file
exec < input.txt
read line  # Reads from input.txt instead of keyboard
echo "Read from file: $line"

# Save original stdout (file descriptor 1)
exec 3>&1  # Copy stdout to FD 3

# Redirect stdout to a file
exec > output.txt
echo "This goes to output.txt"
echo "This also goes to output.txt"

# Restore stdout
exec 1>&3  # Copy FD 3 back to FD 1
echo "Back to terminal"

# Redirect stderr to a file
exec 2> errors.log
echo "Error message" >&2  # Goes to errors.log

# List open file descriptors
ls -l /proc/self/fd
```

---

## 10.2. Shell Option and Debugging Commands

### `shopt`: Dynamic Shell Configuration

The `shopt` command controls shell options on the fly, allowing you to enable/disable features without restarting Bash.

**Common useful options:**

| Option | Effect | Use Case |
|--------|--------|----------|
| `cdspell` | Correct minor typos in `cd` directory names | Interactive shell |
| `nocaseglob` | Pathname expansion is case-insensitive | Inconsistent naming |
| `nullglob` | Pattern expands to nothing instead of literal | Safe globbing |
| `extglob` | Enable extended pattern matching (`@()`, `?()`, etc.) | Complex patterns |
| `dotglob` | Include hidden files (.*) in glob patterns | Script safety |
| `nocasematch` | Pattern matching is case-insensitive | Input validation |

```bash
#!/bin/bash

# Enable typo correction for cd
shopt -s cdspell

# This works now even with typo
cd /hom  # Corrected to /home
pwd       # /home

# Enable extended globbing for complex patterns
shopt -s extglob

# Match everything except .txt files
files=(!(*.txt))
for f in "${files[@]}"; do
  echo "$f"
done

# Enable case-insensitive globbing
shopt -s nocaseglob

# These now all match the same files
echo *.TXT
echo *.txt
echo *.Txt

# Show current settings
shopt
```

**Usage patterns:**

```bash
# Set (enable) option
shopt -s option_name

# Unset (disable) option
shopt -u option_name

# Check if option is set
if shopt -q option_name; then
  echo "Option is enabled"
fi

# List all options and their current state
shopt
```

### `caller [expr]`: Debugging Context Information

The `caller` command outputs information about the calling context—which line in which function called the current function. Essential for debugging.

**Output format:** `line_number function_name source_filename`

```bash
#!/bin/bash

debug_context() {
  # caller 0 = immediate caller
  # caller 1 = caller of the caller
  # etc.
  caller 0
}

function_a() {
  debug_context
}

function_b() {
  function_a
}

# Call from line 18
function_b

# Output: 18 function_b script.sh
```

**Practical: Debug tracer**

```bash
#!/bin/bash

trace_calls() {
  local level=0
  while true; do
    local caller_info=$(caller $level)
    [ -z "$caller_info" ] && break
    echo "  → $caller_info"
    ((level++))
  done
}

function_c() {
  echo "In function_c:"
  trace_calls
}

function_b() {
  function_c
}

function_a() {
  function_b
}

# Call stack will show full path to current function
function_a
```

---

## 10.3. Utility Commands

### `true` and `false`: Simple Success/Failure

These commands return specific exit statuses without doing anything:

- `true` returns 0 (success)
- `false` returns 1 (failure)

**Uses:**

```bash
# Placeholder for stub functions (returns success)
process_item() {
  true  # Not yet implemented
}

# Endless loop pattern
while true; do
  read -p "Enter command (or 'q' to quit): " cmd
  [ "$cmd" = "q" ] && break
  eval "$cmd"
done

# Alternative to 'while true' for clarity
if true; then
  echo "Code block that is always executed"
fi
```

**Anti-pattern to avoid:**

```bash
# Bad: Invokes external /bin/true command
if /bin/true; then ...

# Good: Uses builtin
if true; then ...  # Much faster
```

### `type [cmd]`: Command Type Identification

The `type` command identifies whether a command is a builtin, function, alias, or external executable. Critical for understanding command resolution order.

**Bash command resolution order:**
1. Aliases
2. Functions
3. Builtins
4. External commands (in PATH)

```bash
# Identify what 'echo' is
$ type echo
echo is a shell builtin

# Show ALL definitions of 'echo'
$ type -a echo
echo is a shell builtin
echo is /bin/echo

# Show type of alias
$ type ls
ls is aliased to 'ls --color=auto'

# Show type of function
$ type my_function
my_function is a function

# Show function definition
$ type -a my_function
my_function is a function
my_function () 
{ 
  echo "hello"
}

# Check if command exists (safe in scripts)
if type python3 &>/dev/null; then
  echo "Python 3 is available"
fi
```

**Practical: Safe command invocation**

```bash
#!/bin/bash

safe_invoke() {
  local cmd="$1"
  shift
  
  if ! type "$cmd" &>/dev/null; then
    echo "Error: Command '$cmd' not found"
    return 1
  fi
  
  "$cmd" "$@"
}

safe_invoke ls -la /tmp
```

### `hash [cmds]`: Command Path Caching

The shell maintains a hash table of external command paths for faster lookup. The `hash` command manipulates this table.

```bash
# Show all cached commands
$ hash
  /bin/ls
  /usr/bin/python3
  /usr/bin/grep

# Hash a specific command
$ hash -p /usr/local/bin/python3 python

# Clear entire hash table (forces re-lookup)
$ hash -r

# Remove specific command from cache
$ hash -d python
```

**Use case:** After installing a new version of a command, you may need to clear the hash table:

```bash
#!/bin/bash

# Update Python
install_python_update

# Clear hash table so new python is found
hash -r

# Now using new Python version
python3 --version
```

### `help [builtin]`: Builtin Documentation

Display brief but comprehensive help for shell builtins:

```bash
help exit    # Show exit documentation
help read    # Show read options
help for     # Show for loop syntax
help [       # Show [ (test) documentation
```

---

## 10.4. Job Control Commands

### Understanding Background Jobs and Process Management

Job control allows you to manage multiple foreground and background processes within a single shell session. This is essential for long-running scripts and interactive work.

**Key concepts:**
- **Foreground job**: Blocks your input, has control of terminal
- **Background job**: Runs independently, you can continue typing
- **Job number**: `%1`, `%2` (different from PID)
- **Suspended job**: Stopped with Ctrl-Z, can resume

### Job Management Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `jobs` | List all background jobs | `jobs -l` (with PIDs) |
| `fg [%job]` | Bring job to foreground (blocking) | `fg %1` |
| `bg [%job]` | Resume suspended job in background | `bg %2` |
| `wait [job\|PID]` | Wait for job/process to complete | `wait $PID` |
| `suspend` | Suspend current shell | `suspend` |
| `disown [job]` | Remove job from job table | `disown %1` |

### Job Reference Notation

```bash
%N        # Job number N
%string   # Job whose command starts with string
%?string  # Job whose command contains string
%%  or %+ # Current/most recent job
%-        # Previous job
$!        # PID of last background process
```

**Example:**

```bash
find / -name "*.log" > results.txt &    # Background job 1
sleep 300 &                               # Background job 2
echo "waiting" &                          # Background job 3

jobs                      # Lists all 3 jobs
jobs -l                   # Lists with PIDs
fg %1                     # Bring job 1 to foreground
bg %2                     # Resume job 2 in background
wait %1                   # Wait for job 1 only
kill %3                   # Kill job 3
```

### Example 10-3 (Enhanced): Comprehensive Job Management

```bash
#!/bin/bash
# job-management.sh: Multi-task coordination

echo "Job Management Example"
echo "======================================"

# Start multiple background tasks
echo "Starting background tasks..."

# Task 1: Find operation
find / -name "*.log" > /tmp/logs.txt 2>&1 &
job1=$!
echo "Task 1 (PID $job1): Finding log files"

# Task 2: Sleep (simulates long operation)
sleep 5 &
job2=$!
echo "Task 2 (PID $job2): Sleeping 5 seconds"

# Task 3: Another operation
ping -c 3 example.com > /tmp/ping.txt 2>&1 &
job3=$!
echo "Task 3 (PID $job3): Pinging example.com"

echo
echo "Jobs in background:"
jobs -l

echo
echo "Waiting for all jobs to complete..."
wait $job1
echo "✓ Task 1 (logs) complete"

wait $job2
echo "✓ Task 2 (sleep) complete"

wait $job3
echo "✓ Task 3 (ping) complete"

echo
echo "All background jobs completed"
echo "======================================"
exit 0
```

### Example 10-4 (Enhanced): Preventing Orphan Processes

```bash
#!/bin/bash
# background-task.sh: Safe background job handling

# Check if running as root (required for updatedb)
if [ "$EUID" -ne 0 ]; then
  echo "This script requires root privileges"
  exit 1
fi

echo "Database Update Script"
echo "======================================"
echo "Starting database update (may take a while)..."

# Run background job with output redirection
updatedb / > /tmp/updatedb.log 2>&1 &
update_pid=$!

echo "Database update running (PID: $update_pid)"
echo "You can continue working while this runs..."

# Optional: Show progress
while kill -0 $update_pid 2>/dev/null; do
  echo "  ... still running ..."
  sleep 5
done

# Critical: MUST wait before exiting to prevent orphan process
wait $update_pid
exit_code=$?

echo "Database update complete (exit code: $exit_code)"

if [ $exit_code -eq 0 ]; then
  echo "✓ Update successful"
else
  echo "✗ Update failed"
fi

exit $exit_code
```

### Background Job Hanging Pattern and Solutions

When a script runs a background command that writes to stdout, it can appear to hang:

```bash
# Problem: Output from background job appears after prompt
#!/bin/bash
ls -l &                # Background job writes to terminal
echo "Done."           # User prompt appears first
# Then ls output appears, confusing user
```

**Solution 1: Use `wait` for all background jobs**

```bash
#!/bin/bash
ls -l &
job1=$!

echo "Starting cleanup..."
sleep 1 &
job2=$!

# Wait for all before showing completion
wait $job1 $job2
echo "Done."
```

**Solution 2: Redirect background job output**

```bash
#!/bin/bash
ls -l > /tmp/listing.txt 2>&1 &
echo "Done."  # No hang
```

**Solution 3: Use `disown` to prevent waiting**

```bash
#!/bin/bash
nohup long_task.sh &
disown  # Remove from job table, doesn't wait for completion
echo "Done."
```

---

## 10.5. Process Management Commands

### `kill [signal] PID|job`: Sending Signals to Processes

The `kill` command sends signals to processes. While its name suggests termination, it's actually for sending any signal.

**Common signals:**

| Signal | Number | Meaning | Default Action |
|--------|--------|---------|-----------------|
| SIGTERM | 15 | Graceful termination | Terminate |
| SIGKILL | 9 | Forced kill (cannot be caught) | Terminate |
| SIGSTOP | 19 | Pause process (cannot be caught) | Stop |
| SIGCONT | 18 | Resume paused process | Continue |
| SIGHUP | 1 | Hangup (terminal closed) | Terminate |
| SIGINT | 2 | Interrupt (Ctrl-C) | Terminate |

```bash
# Graceful termination (gives process chance to cleanup)
kill $PID

# Force kill (immediate termination)
kill -9 $PID

# Send specific signal
kill -SIGTERM $PID

# Kill by job number
kill %1

# Kill multiple processes
kill $PID1 $PID2 $PID3

# List all available signals
kill -l
```

**Best practice: Try SIGTERM first, then SIGKILL**

```bash
#!/bin/bash

terminate_process() {
  local pid="$1"
  
  # Try graceful termination
  kill -TERM "$pid" 2>/dev/null
  
  # Give it 5 seconds to exit
  sleep 5
  
  # Check if still running
  if kill -0 "$pid" 2>/dev/null; then
    echo "Process still running, forcing kill..."
    kill -KILL "$pid"
  fi
}

terminate_process $some_pid
```

### Example 10-5 (Enhanced): Self-Terminating Script

```bash
#!/bin/bash
# self-destruct.sh: Script that terminates itself

echo "Script starting (PID: $$)"
echo "Script will self-terminate in 2 seconds..."
sleep 2

echo "Sending SIGTERM to own process..."
kill -TERM $$

# This executes but shell is already terminating
echo "This message may or may not appear"
exit 0  # This exit never happens
```

**Output:**
```
Script starting (PID: 12345)
Script will self-terminate in 2 seconds...
Sending SIGTERM to own process...
Terminated
```

### `killall [signal] command`: Kill by Command Name

Kill all running processes matching a command name:

```bash
# Kill all instances of a process
killall firefox
killall python3

# Send specific signal
killall -TERM node

# Force kill
killall -9 java

# List available signals
killall -l
```

---

## 10.6. Command Directives: Controlling Resolution Order

### `command`: Disable Aliases and Functions

The `command` directive forces the shell to bypass aliases and functions, using only the builtin or external command:

```bash
# If 'ls' is aliased to 'ls --color=auto'
ls         # Uses the alias
command ls # Uses external /bin/ls

# Practical: script that doesn't want alias behavior
#!/bin/bash
command ls -a | grep "\.txt$"  # Guaranteed to use real ls
```

**Use case:** Preventing scripts from being affected by user's aliases

```bash
#!/bin/bash
# backup.sh: Backup script that ignores aliases

# Even if user has aliased 'rm' to something weird
command rm -f "$temp_file"  # Uses actual /bin/rm

# Even if user defined 'cp' function
command cp "$source" "$dest"  # Uses /bin/cp
```

### `builtin`: Force Builtin Execution

The `builtin` directive executes a builtin command, ignoring functions and external commands:

```bash
# If you have a function named 'cd'
cd /tmp       # Calls the function

# Force builtin
builtin cd /tmp  # Calls shell builtin
```

**Use case:** Override function that shadows builtin

```bash
#!/bin/bash

# You've defined your own 'read' function
read() {
  echo "My read function"
}

read -p "Enter name: " name  # Calls your function
builtin read -p "Enter name: " name  # Calls shell builtin
```

### `enable`: Control Builtin Availability

The `enable` command enables/disables shell builtins:

```bash
# List all builtins and their status
enable

# Disable a builtin (use external version instead)
enable -n kill   # Now /bin/kill is used

# Re-enable a builtin
enable kill

# Load builtin from shared object file
enable -f ./custom.so my_builtin
```

---

## 10.7. Process Introspection

### `times`: CPU Usage Statistics

Display CPU time consumed by the shell and its children:

```bash
bash$ times
0m0.020s 0m0.020s    # Shell user time, Shell system time
0m1.234s 0m0.456s    # Children user time, Children system time
```

**Format:**
- `shell_user_time`
- `shell_system_time`
- `children_user_time`
- `children_system_time`

**Use case:** Performance profiling (though typically use `time` command instead)

```bash
#!/bin/bash

# Profile a script
time ./heavy_computation.sh

# Show total CPU used so far
times
```

---

## 10.8. Advanced: Script Recursion and Sourcing

### Self-Sourcing vs. Self-Calling

There's a difference between **sourcing** (`. "$0"`) and **calling** (`"$0"`):

- **Sourcing** (`. "$0"`): Current shell executes script's code—shares variables, functions, etc.
- **Calling** (`"$0"`): Spawns child shell—isolated environment

**Example: Self-sourcing recursion**

```bash
#!/bin/bash
# recursive-counter.sh: Self-sourcing for variable state

MAXPASS=3
pass_count=0  # Initialize!

while [ "$pass_count" -lt "$MAXPASS" ]; do
  echo "Pass $((pass_count + 1)) of $MAXPASS"
  
  . "$0"  # Source self in same shell
  
  ((pass_count += 1))
done

echo "Complete"
exit 0
```

**Warning:** Self-sourcing can cause infinite loops if not careful:

```bash
# Dangerous: infinite loop
#!/bin/bash
echo "Processing..."
. "$0"  # Sources script again, again, and again...
```

**Safe version with counter:**

```bash
#!/bin/bash
: ${RECURSION_DEPTH:=0}

if [ "$RECURSION_DEPTH" -ge 3 ]; then
  echo "Recursion limit reached"
  exit 0
fi

echo "Recursion depth: $RECURSION_DEPTH"
export RECURSION_DEPTH=$((RECURSION_DEPTH + 1))
. "$0"
```

---

## Best Practices and Common Patterns

### 1. Always Explicitly Exit with Status Code

```bash
# Good: Clear, predictable exit
if process_file; then
  exit 0
else
  exit 1
fi

# Avoid: Relying on last command status
process_file
# exit status unclear
```

### 2. Use `wait` for All Background Jobs

```bash
# Good: Explicit wait ensures cleanup
task1.sh &
pid1=$!

task2.sh &
pid2=$!

wait $pid1
wait $pid2

echo "All tasks complete"
```

### 3. Check Command Existence Before Using

```bash
# Good: Safe invocation
if type python3 &>/dev/null; then
  python3 script.py
else
  echo "Python 3 not installed"
  exit 1
fi

# Avoid: Assuming command exists
python3 script.py  # May fail with confusing error
```

### 4. Use `command` in Scripts to Bypass Aliases

```bash
# Good: Ignores user's aliases
#!/bin/bash
command rm -rf "$temp_dir"

# Avoid: Uses alias if it exists
rm -rf "$temp_dir"
```

### 5. Trap Signals for Cleanup

```bash
# Good: Cleanup on exit
cleanup() {
  rm -f "$TEMP_FILE"
  kill $BACKGROUND_PID 2>/dev/null
}

trap cleanup EXIT
trap cleanup SIGINT SIGTERM

# Script now cleans up properly even on Ctrl-C
```

### 6. Initialize Variables Before Use

```bash
# Good: Clear initial value
counter=0
((counter++))

# Avoid: Uninitialized variable
((counter++))  # First iteration has undefined behavior
```

---

## 10 Core Programming Concepts from Builtins

### 1. **Process Control and Lifecycle**
`exit`, `exec`, and background job management demonstrate process lifecycle—creation, state changes, and termination. Understanding this is crucial for systems programming.

```bash
task.sh &      # Process creation (forking)
job_id=$!      # Process tracking
wait $job_id   # Process synchronization
exit $?        # Process termination with status
```

### 2. **Exit Status Codes as Control Flow**
Exit codes are not just status reporting—they're a **mechanism for control flow** between programs. This is fundamental to Unix pipes and process composition.

```bash
if command; then    # Using exit code as condition
  next_command
fi
```

### 3. **Shell Features vs. External Commands**
Understanding the distinction between builtins and external commands reveals how operating systems abstract functionality. Builtins can't fork (too expensive), so shells provide them.

```bash
type cd        # Builtin (can't be external—affects current shell)
type ls        # External (spawns child process)
command cd     # Explicitly use builtin
```

### 4. **Process Replacement via `exec`**
`exec` demonstrates process replacement—the fundamental mechanism that allows Unix to run completely different programs. It's the basis for `execve()` system call.

```bash
exec python script.py  # Shell process becomes Python process
```

### 5. **Background Processes and Concurrency**
The `&` operator and job control (`jobs`, `fg`, `bg`, `wait`) are mechanisms for concurrent execution—essential for multitasking and parallel processing.

```bash
task1 &    # Concurrent execution
task2 &    # Both run in parallel
wait       # Synchronization point
```

### 6. **Signal Handling and Process Communication**
`kill` and signal handling (`trap`, `SIGTERM`, `SIGKILL`) demonstrate inter-process communication—how programs signal each other asynchronously.

```bash
kill -TERM $pid    # Asynchronous signal to process
trap cleanup EXIT  # Handler for signals
```

### 7. **Command Resolution and Namespace**
`type`, `command`, `builtin`, and `hash` reveal how shells resolve commands through a priority order (aliases → functions → builtins → external). This is a namespace resolution pattern.

```bash
type echo      # Reveals resolution order
command echo   # Forces specific resolution
```

### 8. **Caching and Performance Optimization**
The `hash` table demonstrates the **memoization pattern**—caching expensive lookups (external command paths) for faster repeated access.

```bash
hash                    # View cache
hash -r                 # Clear cache when needed
```

### 9. **Introspection and Reflection**
`caller`, `type`, and `enable` enable **runtime introspection**—programs examining themselves and other programs. This is the basis for debuggers, profilers, and reflective systems.

```bash
caller 0       # Introspection: who called me?
type $cmd      # Introspection: what is this?
enable         # Introspection: what's available?
```

### 10. **Delayed Execution and Lazy Evaluation**
Functions, sourcing, and command evaluation patterns demonstrate **lazy evaluation**—delaying execution until needed. This is fundamental to functional programming and metaprogramming.

```bash
. "$0"              # Delayed self-execution
eval "$command"     # Runtime command generation
```

---

## Summary Table: Common Builtins

| Command | Purpose | Example |
|---------|---------|---------|
| `exit [n]` | Terminate script with status | `exit 1` |
| `exec cmd` | Replace shell with command | `exec python script` |
| `shopt` | Control shell options | `shopt -s extglob` |
| `caller [n]` | Show calling function info | `caller 0` |
| `true` | Return success (0) | `if true; then ...` |
| `false` | Return failure (1) | `if false; then ...` |
| `type cmd` | Identify command type | `type ls` |
| `hash [cmds]` | Manage command cache | `hash -r` |
| `help [builtin]` | Show builtin help | `help read` |
| `jobs [opts]` | List background jobs | `jobs -l` |
| `fg [%job]` | Foreground a job | `fg %1` |
| `bg [%job]` | Background a job | `bg %2` |
| `wait [job\|PID]` | Wait for completion | `wait $pid` |
| `kill [signal] pid` | Terminate process | `kill -9 $pid` |
| `killall cmd` | Kill by command name | `killall firefox` |
| `command cmd` | Disable aliases/functions | `command rm file` |
| `builtin cmd` | Force builtin execution | `builtin cd /tmp` |
| `enable [-n] cmd` | Enable/disable builtin | `enable -n kill` |
| `times` | Show CPU time stats | `times` |
| `suspend` | Suspend shell | `suspend` |
| `disown [job]` | Remove from job table | `disown %1` |

---

## Exercises

1. **Write a script** that uses different `exit` status codes to report specific errors (file not found=1, permission denied=2, invalid argument=3).

2. **Create a background task manager** using `jobs`, `wait`, and `kill` that monitors multiple long-running tasks and ensures clean termination.

3. **Implement a debugging wrapper** function that uses `caller` to print the call stack whenever an error occurs.

4. **Build a command availability checker** using `type` that detects whether required commands are available and suggests alternatives.

5. **Write a script** that uses `shopt` to enable/disable various shell options and demonstrate their effects.

6. **Create a process monitor** that uses `kill -0` to check if processes are still running and restarts them if necessary.

7. **Implement a signal handler** using `trap` that catches SIGTERM and SIGINT to perform cleanup before exiting.

8. **Build a concurrent task executor** that starts multiple background jobs, monitors progress, and waits for completion.

---

## Summary

**Script control:**
- **`exit [n]`**: Always explicitly exit with status code for reliability
- **`exec`**: Process replacement (rarely used directly, but critical for understanding shells)

**Shell configuration:**
- **`shopt`**: Dynamic option control for shell behavior
- **`caller`**: Essential for debugging—shows function call stack

**Utilities:**
- **`type`**: Identify command type (critical for script safety)
- **`hash`**: Command path caching (automatic, but important to understand)

**Process management:**
- **`jobs`, `fg`, `bg`, `wait`**: Essential for managing concurrent tasks
- **`kill`, `killall`**: Signal-based process control
- **`disown`**: Release job from shell's management

**Command directives:**
- **`command`**: Bypass aliases/functions (use in scripts)
- **`builtin`**: Force builtin execution
- **`enable`**: Control builtin availability

**Key insight**: Builtins are not just conveniences—they're critical for shell functionality and script performance. Understanding them reveals how shells work at a fundamental level and enables writing robust, efficient scripts that interact properly with Unix processes and signals.
