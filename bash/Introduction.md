# Chapter 1: Introduction to Bash Scripting

## What is Bash?

Bash scripting is the practical glue of Unix-like systems. As a Linux machine boots it typically runs initialization scripts (for example `/etc/rc.d` on some distributions) to restore system configuration and start services — many of those tasks are performed with small shell scripts. Shell scripts follow the classic Unix philosophy: break a complex task into small steps and chain existing utilities together.

**Bash** stands for "Bourne-Again SHell" and is a command interpreter (or shell) commonly used on Linux systems. Unlike compiled languages such as C or Python, Bash is an **interpreted language** — commands are read and executed line-by-line by the shell without being compiled to machine code first.

### Why Use Bash?

Bash scripting offers several advantages over manually executing commands:

1. **Reproducibility**: Save commands in a script file and run them identically every time
2. **Automation**: Execute complex workflows automatically on schedules or in response to events
3. **Reusability**: Share scripts across systems and with other users
4. **Error Handling**: Include conditional logic and error checking that would be tedious to type manually
5. **Efficiency**: Combine multiple tools and utilities into a single workflow

A Bash script is simply a text file containing shell commands. It runs in a **shell environment**, which is a program that interprets your commands and executes them.

---

## The shebang (`#!`) and Interpreter Selection

### Understanding the Shebang

At the top of most scripts you'll find a line starting with `#!` — called a "shebang" or "hashbang". This special marker tells the operating system kernel which interpreter to use when executing the script:

```bash
#!/bin/bash
# or, for portability to other Unix-like systems:
#!/bin/sh
```

**How it works:**

When you execute a script file (like `./myscript`), the kernel reads the first two bytes. If they are `#!`, the kernel recognizes this as a directive and reads the rest of the line as the path to an interpreter program. The kernel then runs that interpreter with your script as an argument.

**Example workflow:**
```bash
# User executes:
./myscript.sh

# Kernel sees #!/bin/bash and internally does:
/bin/bash ./myscript.sh
```

### Choosing the Right Shebang

**Use `#!/bin/sh`** when you want maximum portability:
- Your script should run on systems where only POSIX-compliant shells are available
- You avoid Bash-specific features (arrays, `[[ ]]`, process substitution, etc.)
- The script runs on BSD, Solaris, and other Unix variants without modification
- Note: On many modern Linux distributions, `/bin/sh` is actually a symbolic link to `/bin/bash`, but explicitly using `#!/bin/sh` signals intent to be portable

**Use `#!/bin/bash`** when you:
- Specifically want Bash features (indexed/associative arrays, extended test syntax)
- Are writing scripts for Linux systems where Bash is guaranteed to be available
- Develop scripts for your own use or controlled environments where Bash is standard
- Benefit from Bash-only improvements in error handling and functionality

### The Shebang Must Be First

The shebang must be the **absolute first line** of the script — no blank lines, comments, or other content before it. The kernel only checks the first two bytes, and if they are not `#!`, it assumes the file is a simple text file to be processed by the default shell.

---

## Making a Script Executable

### File Permissions Basics

When you create a script file, it starts with default permissions that typically don't include the execute permission. The **permission system** in Unix/Linux determines who can read, write, and execute files.

Write your script in a text file (for example `myscript.sh`) and make it executable using the `chmod` command:

```bash
chmod +x myscript.sh
```

**Breaking down `chmod +x`:**
- `chmod` = "change mode" — the command for modifying file permissions
- `+x` = "add execute permission"
- `myscript.sh` = the file to modify

### Verifying Executable Status

Check if a script is executable with `ls -l`:

```bash
# Before making it executable:
$ ls -l myscript.sh
-rw-r--r-- 1 user user 150 Jan 27 10:30 myscript.sh

# The first 10 characters show permissions: -rw-r--r--
# Breaking it down:
# -       = regular file (not directory or link)
# rw-     = owner can read and write (no execute)
# r--     = group can read only
# r--     = others can read only

# After chmod +x:
$ ls -l myscript.sh
-rwxr-xr-x 1 user user 150 Jan 27 10:30 myscript.sh
#  ^       = owner now has execute permission
```

### Understanding the Permission Groups

Linux file permissions are divided into **3 groups**, each with exactly 3 slots:

```
-rwxr-xr-x
 -  rwx   r-x   r-x
 |  |     |     |
 |  |     |     └── others (everyone else)
 |  |     └──────────── group (users in the file's group)
 |  └───────────────────── owner (the user who created the file)
 └────────────────────────── file type (- = regular file, d = directory, l = link)
```

**Each group has exactly 3 positions:**
- **r** (read) = can view file contents
- **w** (write) = can modify/delete file
- **x** (execute) = can run the file as a program

**Critical rule**: Each group can have **at most one r, one w, and one x**. The extra r's and w's you see belong to the **next group**:

```bash
-rwxr-xr-x
  └└└ owner (rwx)
     └└└ group (r-x, meaning read and execute only)
        └└└ others (r-x, meaning read and execute only)
```

**What r and x together mean:**
- **r-x** (read + execute) = You can **read AND run the file**, but cannot **modify it**
- **rwx** (read + write + execute) = You have **full control**: read, modify, and execute
- **r--** (read only) = You can only **view** the contents; cannot modify or execute
- **--x** (execute only) = You can **run** the program but cannot read its source code

**Common permission examples:**

| Permission | Meaning |
|------------|----------|
| `-rw-r--r--` | Owner can read/write; group and others can only read |
| `-rwxr-xr-x` | Owner has full control; group and others can read and execute |
| `-rwx------` | Only owner can do anything; group and others have no access |
| `-r--r--r--` | Everyone can read; nobody can modify or execute |

### Running Your Script

Once executable, run the script with:

```bash
./myscript.sh          # Execute from current directory
/path/to/myscript.sh   # Execute from any directory using absolute path
bash myscript.sh       # Execute with explicit interpreter (doesn't require x permission)
```

**Why `./` prefix is needed:**
The `./` tells the shell to look for the command in the current directory. Without it, the shell searches the directories listed in the `$PATH` environment variable, which typically doesn't include the current directory (for security reasons).

---

## Your First Bash Script

### A Simple Example

Create a file named `hello.sh` with the following content:

```bash
#!/bin/bash

# This is a comment - it starts with # and is ignored by the shell
echo "Hello, World!"
```

Now make it executable and run it:

```bash
chmod +x hello.sh
./hello.sh
```

**Output:**
```
Hello, World!
```

### Understanding the Components

| Component | Meaning |
|-----------|---------|
| `#!/bin/bash` | Shebang: tells the kernel to use `/bin/bash` interpreter |
| `# This is a comment` | Comments explain code; ignored by the shell |
| `echo` | **Command**: prints text to standard output (the terminal) |
| `"Hello, World!"` | **String**: text enclosed in quotes |

### Comments Explained

A **comment** is text that the shell ignores. Comments are essential for code documentation:

```bash
#!/bin/bash

# This entire line is a comment and won't be executed
echo "This will print"  # This is an inline comment
# The next line demonstrates variable use
name="Alice"           # Assign value to variable
echo $name             # Use the variable
```

---

## Understanding Scripts as Sequences of Commands

### The Shell as Command Interpreter

A Bash script is essentially a list of commands executed sequentially, line by line. You can run the exact same commands either interactively at the shell prompt or in a script file:

**Interactive (typing commands one at a time):**
```bash
$ name="Bob"
$ echo "Hello, $name"
Hello, Bob
```

**In a script file (`greet.sh`):**
```bash
#!/bin/bash
name="Bob"
echo "Hello, $name"
```

When you run `./greet.sh`, the shell reads and executes each line in order, just as if you had typed them at the prompt.

### Variables and Substitution

A **variable** is a named container that holds a value. When you reference a variable using `$variablename`, the shell replaces it with the variable's value:

```bash
#!/bin/bash

# Assignment: Create a variable and give it a value
year=2026
message="Welcome to Bash"

# Substitution: Use the variable by prefixing with $
echo $year              # Output: 2026
echo $message           # Output: Welcome to Bash
echo "Year is $year"    # Output: Year is 2026
```

**Key points:**
- No spaces around `=` in variable assignment
- Variable names are case-sensitive (`Name` and `name` are different)
- Access variables with `$variablename`

### Understanding Variable Persistence

Shell variables have **limited lifetime** — they exist only in their shell environment:

**Interactive shell example:**

```bash
$ name="Alice"
$ echo $name
Alice

# Variables persist in the same shell session
$ echo $name
Alice

# But exit the shell and start a new one:
$ exit
$ echo $name
# (nothing — variable doesn't exist in new shell)
```

**Why variables don't persist across shells:**

Variables live in the **current shell's memory** (RAM). When the shell exits, that memory is freed and all variables disappear. A new shell session gets its own separate memory space.

**Making variables persist:**

1. **Export to child processes** (they inherit, but don't persist after exit):
   ```bash
   export name="Alice"
   bash                    # Start a subshell
   echo $name              # Works! "Alice" is inherited
   exit                    # Return to parent shell
   ```
   The variable is only available to child processes, not saved permanently.

2. **Save to `~/.bashrc`** (persists across terminal sessions):
   ```bash
   # Add this line to ~/.bashrc
   export MY_VAR="Alice"
   ```
   Now every time you open a terminal, `MY_VAR` will be available in that shell.

3. **Save to `~/.bash_profile`** (persists for login shells on some systems):
   ```bash
   # Add to ~/.bash_profile for login shell persistence
   export MY_VAR="Alice"
   ```

**In short:**
- Variables created in a shell session **exist only in that session's memory**
- **`export`** makes variables available to child processes, but they're still lost when all shells exit
- **`~/.bashrc`** is the way to save variables permanently across different terminal sessions
- Without saving to a config file or exporting, variables are **temporary** and disappear when you close the terminal

### Functions for Reusable Code

A **function** is a reusable block of commands. Define a function once and call it multiple times:

```bash
#!/bin/bash

# Function definition
greet() {
    echo "Hello, $1!"    # $1 means the first argument passed to the function
}

# Function calls (using the function)
greet "Alice"           # Output: Hello, Alice!
greet "Bob"             # Output: Hello, Bob!
```

In this example:
- `greet()` is the function name
- `$1` refers to the first argument passed when the function is called
- The function can be called multiple times with different arguments

---

## Execution Modes

### Interactive Mode

In interactive mode, you type commands and the shell immediately executes them:

```bash
$ echo "Testing"
Testing
$ name="Alice"
$ echo $name
Alice
```

This mode is useful for quick tasks and exploration, but becomes tedious for complex workflows.

### Script Mode (Non-Interactive)

In script mode, the shell reads commands from a file and executes them all in sequence:

```bash
#!/bin/bash
# complex_task.sh

# Multiple commands executed in order
echo "Starting backup..."
cp important_file.txt backup_important_file.txt
echo "Backup complete!"
```

Run it once, and all commands execute automatically without needing to type anything.

### Sourcing (Including Other Scripts)

You can include commands from another script using the `source` command (or its alias `.`):

```bash
#!/bin/bash
# main.sh

source ./setup.sh      # Execute setup.sh in the current shell
# or equivalently:
. ./setup.sh

# Now commands and variables from setup.sh are available here
```

This is useful for organizing code into modules and reusing common setup logic.

---

## Example: A Practical Script

Here's a script that demonstrates several concepts:

```bash
#!/bin/bash
# backup.sh - Simple backup script

# Variables
source_dir="$HOME/Documents"
backup_dir="$HOME/Backups"
timestamp=$(date +%Y-%m-%d_%H-%M-%S)

# Function to perform backup
backup_files() {
    local filename="backup_$timestamp.tar.gz"
    
    echo "Starting backup at $(date)"
    tar -czf "$backup_dir/$filename" "$source_dir"
    
    if [ $? -eq 0 ]; then
        echo "Backup successful: $filename"
    else
        echo "Backup failed!"
        return 1
    fi
}

# Main script execution
backup_files
```

**Concepts demonstrated:**
- **Shebang** and initial setup
- **Variables** (`source_dir`, `backup_dir`, `timestamp`)
- **Command substitution** (`$(date +%Y-%m-%d_%H-%M-%S)` runs date command and uses output)
- **Functions** (`backup_files()` groups related commands)
- **Conditionals** (`if [ $? -eq 0 ]` checks if the previous command succeeded)

---

## Programming Keywords and Concepts

### Essential Bash Keywords

| Keyword | Type | Purpose |
|---------|------|---------|
| `echo` | Built-in Command | Print text to standard output |
| `read` | Built-in Command | Read input from user or standard input |
| `if/then/else` | Control Structure | Execute commands conditionally |
| `for` | Loop | Repeat commands for each item in a list |
| `while` | Loop | Repeat commands while condition is true |
| `function` | Declaration | Define a reusable block of commands |
| `return` | Control Flow | Exit a function and optionally return a value |
| `export` | Declaration | Make a variable available to subshells |
| `local` | Declaration | Restrict variable scope to a function |
| `source` or `.` | Command | Execute commands from another script file |

### Core Programming Concepts in Bash

**1. Variables**
- Named containers that hold values (strings, numbers, arrays)
- Created with assignment: `name="value"`
- Accessed with `$name` (variable substitution)
- Case-sensitive and can contain letters, numbers, underscores

**2. Quoting**
- **Double quotes `"..."` **: Allow variable substitution; `echo "$name"` prints the value
- **Single quotes `'...'` **: Treat content literally; `echo '$name'` prints the literal string `$name`
- Essential for handling values with spaces or special characters

**3. Command Substitution**
- Running a command and using its output as a value
- Syntax: `$(command)` or `` `command` `` (backticks)
- Example: `today=$(date +%Y-%m-%d)` stores today's date in a variable

**4. Exit Codes**
- Every command returns a **status code** (0 = success, non-zero = failure)
- Access with `$?` immediately after a command
- Used in conditionals: `if [ $? -eq 0 ]` means "if the previous command succeeded"

**5. Arguments/Parameters**
- `$0` = name of the script itself
- `$1`, `$2`, `$3`... = arguments passed to script or function
- `$@` = all arguments as separate items
- `$#` = number of arguments

**6. Redirection**
- `>` = send output to a file (overwrites)
- `>>` = append output to a file
- `<` = read input from a file
- `2>` = redirect error messages
- `|` = pipe output from one command to another as input

**7. Conditionals**
- Execute commands only if a condition is true
- Test conditions with `[ expression ]` or `[[ expression ]]` (Bash-only, more powerful)
- Common tests: file existence (`-f`), equality (`=`), numeric comparison (`-eq`)

**8. Loops**
- **for loop**: Iterate over items in a list
- **while loop**: Repeat while a condition is true
- Essential for automating repetitive tasks

**9. Functions**
- Reusable blocks of code with local scope
- Can accept arguments via `$1`, `$2`, etc.
- Return values with `return` command or via output

**10. Pipes and Filters**
- Chain commands together: `command1 | command2`
- Output of first command becomes input to second
- Enables powerful one-liners combining multiple tools

---

## Next Steps

Now that you understand the basics of Bash scripting, the next chapters will cover:

- **Chapter 2**: Bash Basics — special characters, stdin/stdout, filters
- **Chapter 3**: Variables and Parameters — deeper exploration of data storage
- **Chapter 4**: Quoting and Escaping — mastering string handling
- **Chapter 5**: Testing and Comparisons — conditional logic
- **Chapter 6-12**: Advanced features, functions, scripting patterns, and system administration

Start by writing small scripts to practice these concepts. The best way to learn Bash is through hands-on experimentation!
