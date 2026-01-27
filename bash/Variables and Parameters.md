# Chapter 3: Introduction to Variables and Parameters

Variables are how programming and scripting languages represent data. A **variable** is nothing more than a **label** — a name assigned to a location (or set of locations) in computer memory holding an item of data. Think of a variable as a named container: the variable name identifies the container, and the variable value is what's inside.

Understanding variables is essential because they enable:
- **Data storage** — Save values for later use
- **Reusability** — Use the same value in multiple places without hardcoding
- **Parameterization** — Make scripts flexible by accepting different inputs
- **Computation** — Perform calculations and transformations on data

---

## 3.1. Variable Substitution

### Understanding Variable Names and Values

The fundamental concept in variable usage is clearly distinguishing between the **name** of a variable and its **value** — the data it holds.

| Concept | Example | Meaning |
|---------|---------|---------|
| **Variable name** | `myvar` | The label itself (no special meaning) |
| **Variable reference** | `$myvar` | "Get the value stored in myvar" |
| **Value** | `42` | The actual data the variable holds |

When you use `$` prefix, you're telling Bash: "I want the value stored in this variable, not the name."

### When NOT to Use `$`

The only times a variable appears "naked" (without the `$` prefix) is when:

1. **Declared or assigned** — Creating or changing the variable:
   ```bash
   myvar=42          # Assignment (no $)
   ```

2. **Unset** — Removing a variable:
   ```bash
   unset myvar       # Remove variable (no $)
   ```

3. **Exported** — Making available to subprocesses:
   ```bash
   export myvar      # Make global (no $)
   ```

4. **In arithmetic expressions** — Within double parentheses:
   ```bash
   ((myvar = myvar + 1))    # Arithmetic context (no $)
   ```

5. **As a function name** — When defining functions:
   ```bash
   myfunction() { ... }     # Function definition (no $)
   ```

### Demonstrating Variable Substitution

Here's a concrete example showing the difference:

```bash
bash$ variable1=23

# Using the name (naked):
bash$ echo variable1
variable1              # Outputs the literal text "variable1"

# Using the value (with $):
bash$ echo $variable1
23                     # Outputs the VALUE stored in variable1

# Both forms:
bash$ echo "The value is: $variable1"
The value is: 23      # $ triggers substitution even in double quotes
```

---

## 3.2. Types of Variables

### Untyped Variables

Unlike languages such as Python, C, or Java, Bash variables are **untyped**. This means:

- **No type declaration needed** — You don't declare whether a variable is a string, number, list, etc.
- **Type is implicit** — The shell infers type based on how you use it
- **Flexible conversion** — Values are automatically converted between strings and numbers as needed
- **Everything is a string** — Internally, Bash stores all variables as strings

**Example showing type flexibility:**

```bash
#!/bin/bash

# Assign a number-looking string
count=5

# Use as string
echo "The count is: $count"      # Works: string concatenation
# Output: The count is: 5

# Use in arithmetic
echo $((count + 10))             # Works: arithmetic (5 + 10 = 15)
# Output: 15

# This is actually stored as a string "5", but Bash converts it for arithmetic
```

### Variable Naming Conventions

**Valid variable names:**
- Start with a letter or underscore
- Contain letters, numbers, and underscores
- Case-sensitive (`MyVar`, `myvar`, and `MYVAR` are three different variables)

**Bash conventions:**
- **UPPERCASE** — Environment variables and constants (e.g., `PATH`, `HOME`)
- **lowercase** — Local variables in scripts
- **snake_case** — Multi-word variables (e.g., `user_input`, `file_count`)
- **camelCase** — Some scripts use this (e.g., `userName`, `fileSize`)

```bash
# Good naming:
username="alice"
file_path="/home/user/documents"
SCRIPT_VERSION="1.0"

# Poor naming (still works, but confusing):
x=5
a_very_long_name_that_is_unclear=10
VAR="value"   # Generic, unhelpful name
```

---

## 3.3. Special Variables

### Special Parameters Set by the Shell

Bash provides **special variables** that contain information about the script and its execution:

| Variable | Meaning |
|----------|---------|
| `$0` | The name of the script or shell itself |
| `$1`, `$2`, ... | Positional parameters (command-line arguments) |
| `$#` | Number of positional parameters (argument count) |
| `$@` | All positional parameters as separate words |
| `$*` | All positional parameters as a single string |
| `$?` | Exit status of the last command (0 = success, non-zero = failure) |
| `$$` | Process ID (PID) of the current shell |
| `$!` | Process ID of the last background process |
| `$-` | Current shell options |

### Understanding $@ vs $*

These two are similar but have important differences:

```bash
#!/bin/bash

# With arguments: script.sh one two three

# $* combines all arguments into a single string
for arg in $*; do
    echo "$arg"
done
# Output: one two three (three separate iterations, but as one string)

# $@ treats each argument as separate
for arg in "$@"; do
    echo "$arg"
done
# Output: one two three (same, but each properly quoted)

# The difference matters when arguments have spaces:
# $ script.sh "one two" three

# "$*" outputs: one two three (loses the space in first argument!)
# "$@" outputs: one two and three (preserves the first argument's space!)
```

**Best practice:** Always use `"$@"` (with quotes) when passing arguments to functions or other commands.

---

## 3.4. Positional Parameters

### Understanding Script Arguments

When you run a script with arguments, Bash automatically assigns them to **positional parameters**:

```bash
# Running a script:
./myscript.sh arg1 arg2 arg3

# Inside the script:
# $0 = ./myscript.sh (the script name)
# $1 = arg1
# $2 = arg2
# $3 = arg3
# $# = 3 (three arguments)
```

### Practical Example with Positional Parameters

```bash
#!/bin/bash
# file_ops.sh - Demonstrate positional parameters

if [ $# -lt 2 ]; then
    echo "Usage: $0 <source> <destination>"
    exit 1
fi

source_file="$1"
dest_file="$2"

echo "Copying $source_file to $dest_file..."
cp "$source_file" "$dest_file"

if [ $? -eq 0 ]; then
    echo "Copy successful"
else
    echo "Copy failed"
    exit 1
fi
```

**Explanation:**
- `$#` checks if at least 2 arguments provided
- `"$1"` gets the first argument (source file)
- `"$2"` gets the second argument (destination file)
- `$0` in error message shows how to run the script
- `$?` checks if the copy succeeded

### Shifting Parameters

The `shift` command removes the first positional parameter and shifts all others down:

```bash
#!/bin/bash

echo "All arguments: $@"        # all arg1 arg2 arg3
shift                           # Remove arg1
echo "After shift: $@"          # all arg2 arg3
shift 2                         # Remove arg2 and arg3
echo "After shift 2: $@"        # all
```

This is useful in loops when processing unknown numbers of arguments:

```bash
#!/bin/bash
# Process each argument

while [ $# -gt 0 ]; do
    echo "Processing: $1"
    shift
done
```

---

## 3.5. Quoting Variables

### Double Quotes: Preserve Value but Allow Substitution

Double quotes (`"..."`) preserve the value of a variable (including spaces) but still allow substitution:

```bash
hello="A B C D"

echo $hello           # A B C D (word-splits on spaces)
echo "$hello"         # A B C D (preserves spaces as one value)
```

**Why this matters:**

```bash
names="Alice Bob Charlie"

# Without quotes (word-splitting):
for name in $names; do
    echo "$name"
done
# Output: Alice (newline) Bob (newline) Charlie

# With quotes (treated as single value):
for name in "$names"; do
    echo "$name"
done
# Output: Alice Bob Charlie (as one value)
```

### Single Quotes: Treat Literally (No Substitution)

Single quotes (`'...'`) prevent variable substitution entirely:

```bash
greeting="Hello"

echo "$greeting"      # Hello (substitution happens)
echo '$greeting'      # $greeting (literal text, no substitution)
```

**Use single quotes when:**
- You want literal `$` signs in output
- Protecting special characters
- Preventing accidental substitution

```bash
echo 'The cost is $50'      # The cost is $50 (literal)
echo "The cost is $50"      # The cost is  (substitution attempted!)
```

### Brace Notation: ${variable}

The brace notation `${variable}` is identical to `$variable` in most contexts, but necessary in some:

```bash
name="Alice"

# These are equivalent:
echo $name          # Alice
echo ${name}        # Alice

# Braces are REQUIRED when variable is followed by alphanumeric:
echo $namehere      # Tries to substitute variable "namehere" (doesn't exist!)
echo ${name}here    # Correctly outputs: Alicehere

# Braces enable advanced parameter expansion:
echo ${name:0:3}    # First 3 characters: Ali
echo ${name^^}      # Uppercase: ALICE
echo ${name,,}      # Lowercase: alice
```

---

## 3.6. Variable Assignment

### Basic Assignment

The most straightforward way to create a variable and give it a value:

```bash
variable_name=value

# No spaces around the = sign!
count=42
message="Hello, World"
empty_var=""
```

**Critical rule: No spaces around `=`**

```bash
# Correct:
myvar=42

# Wrong: Bash tries to run "myvar" as a command!
myvar = 42
# Error: myvar: command not found

# Wrong: Tries to run "=42" with myvar as an environment variable
myvar= 42
# Error: 42: command not found
```

### Reading User Input with `read`

The `read` command waits for user input and stores it in a variable:

```bash
#!/bin/bash

echo "What is your name?"
read username

echo "Hello, $username!"
```

**Reading multiple variables:**

```bash
read firstname lastname age < <(echo "Alice Bob 30")

echo "Name: $firstname $lastname"
echo "Age: $age"
```

**Reading from a file:**

```bash
while read line; do
    echo "Line: $line"
done < myfile.txt
```

---

## 3.7. Environment Variables

### Local vs Environment Variables

**Local variables** exist only in the current shell:

```bash
myvar=42
echo $myvar        # Works: 42
bash               # Start subshell
echo $myvar        # Doesn't work: variable doesn't exist in subshell
exit               # Back to parent shell
```

**Environment variables** are inherited by subprocesses:

```bash
export myvar=42
echo $myvar        # Works: 42
bash               # Start subshell
echo $myvar        # Works: 42 (inherited from parent!)
exit               # Back to parent shell
```

### Common Environment Variables

| Variable | Meaning | Example |
|----------|---------|---------|
| `HOME` | User's home directory | `/home/alice` |
| `USER` | Current username | `alice` |
| `PATH` | Directories to search for commands | `/usr/bin:/bin:/usr/sbin` |
| `PWD` | Current working directory | `/home/alice/projects` |
| `SHELL` | Current shell | `/bin/bash` |
| `LANG` | Language/locale | `en_US.UTF-8` |
| `TERM` | Terminal type | `xterm-256color` |
| `EDITOR` | Default text editor | `vim` |

---

## Programming Keywords and Concepts

### Essential Variable-Related Keywords

| Keyword | Type | Purpose |
|---------|------|---------|
| `export` | Built-in | Make a variable available to subprocesses |
| `unset` | Built-in | Remove a variable |
| `read` | Built-in | Read user input into a variable |
| `declare` | Built-in | Declare variables with special properties |
| `local` | Built-in | Create function-local variables |
| `$` | Operator | Prefix for variable substitution |
| `${}` | Operator | Brace notation for advanced substitution |
| `=` | Operator | Variable assignment |
| `shift` | Built-in | Shift positional parameters left |
| `$@` | Special | All positional parameters (properly quoted) |

### Core Programming Concepts in Variables and Parameters

**1. Variable Scope**
- **Global**: Variables in main script accessible everywhere
- **Local**: Variables in functions with `local` keyword
- **Environment**: Variables exported to subshells and child processes
- Understanding scope prevents bugs and unintended side effects

**2. Parameter Expansion**
- `${variable}` basic form
- `${variable:-default}` use default if unset
- `${variable:=default}` assign default if unset
- `${variable:0:5}` substring extraction
- `${variable^^}` uppercase conversion
- Allows sophisticated variable manipulation without subprocesses

**3. Positional Parameters**
- `$0`, `$1`, `$2`, etc. for script/function arguments
- `$#` for argument count
- `$@` and `$*` for all arguments
- Essential for writing flexible, reusable scripts

**4. Special Variables**
- `$$` process ID (useful for temporary files)
- `$?` exit code (critical for error handling)
- `$!` background process ID
- Enable script to introspect itself and its environment

**5. Quoting Context**
- Double quotes allow substitution but preserve structure
- Single quotes prevent all substitution
- Backticks/`$()` for command substitution
- Choosing correct quoting prevents subtle bugs

**6. Untyped Nature**
- Everything is a string internally
- Type conversions happen automatically in context
- Flexibility vs. the need for care and validation
- String comparisons vs. numeric comparisons

**7. Parameter Shift**
- `shift` removes and shifts positional parameters
- Enables processing unknown numbers of arguments
- Alternative to indexed array access

**8. Command Substitution in Variables**
- `var=$(command)` executes command, stores output
- `var=\`command\`` older syntax (still works)
- Enables dynamic values based on system state

**9. Here-Strings and Input Redirection**
- `read var < <(echo "value")` assigns from command output
- `<` redirects input from file
- `<<< ` here-string operator
- Enables reading into variables from various sources

**10. Variable Naming and Convention**
- Names should be descriptive and follow conventions
- Case convention indicates scope and purpose
- Good naming improves readability and maintainability
- Variable names form part of the script's documentation

---

## Common Patterns and Best Practices

### Safe Variable Usage

**Always quote variables in double quotes:**

```bash
# Good: quotes prevent word-splitting
cp "$source_file" "$destination"

# Bad: spaces in filename cause problems
cp $source_file $destination    # If filename has spaces, breaks!
```

**Use ${variable} when adjacent to text:**

```bash
# Good: braces separate variable from text
echo ${name}here

# Bad: tries to substitute "namehere" variable
echo $namehere
```

**Check parameter count:**

```bash
if [ $# -lt 2 ]; then
    echo "Usage: $0 <arg1> <arg2>"
    exit 1
fi
```

**Use meaningful names:**

```bash
# Good: clear what the variable contains
source_file="$1"
destination_dir="$2"
backup_count=3

# Bad: unclear purpose
f="$1"
d="$2"
n=3
```

---

## Summary

Variables are the foundation of Bash scripting:

- **Container concept**: Variables hold data (always as strings internally)
- **Naming conventions**: Use descriptive names that indicate purpose
- **Quoting matters**: Double quotes preserve value; single quotes prevent substitution
- **Positional parameters**: Enable flexible, reusable scripts
- **Special variables**: Provide introspection and control
- **Scope matters**: Local vs. environment affects behavior in subshells
- **Always quote**: Prevents subtle bugs with spaces and special characters

Master variable usage and you'll write more robust and maintainable scripts.

---

## Next Steps

The next chapters build on variables:
- **Chapter 4**: Quoting and Escaping — Advanced string handling with variables
- **Chapter 5**: Testing and Comparisons — Using variables in conditions
- **Chapter 6+**: Advanced features and practical scripting patterns
