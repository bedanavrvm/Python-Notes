# Chapter 5: Testing and Comparisons

Testing is central to writing robust Bash scripts. The `test` command (also spelled `[ ]`) and its extended form `[[ ]]` allow you to evaluate conditions and make decisions based on file properties, variable values, and logical combinations.

In programming, **testing** means checking whether a condition is true or false. Based on that result, you execute different code paths. Without testing, scripts are rigid and can't respond to different situations.

---

## 5.1. Understanding Test Syntax

### Three Ways to Test Conditions

Bash provides three different syntaxes for testing:

1. **`[ condition ]`** — POSIX-compatible test command
   ```bash
   if [ -f myfile.txt ]; then
     echo "myfile.txt is a regular file"
   fi
   ```

2. **`[[ condition ]]`** — Bash extended test (more powerful)
   ```bash
   if [[ -f myfile.txt ]]; then
     echo "myfile.txt is a regular file"
   fi
   ```

3. **`(( condition ))`** — Arithmetic test (numbers only)
   ```bash
   if (( count > 5 )); then
     echo "Count is greater than 5"
   fi
   ```

**Key differences:**

| Feature | `[ ]` | `[[ ]]` | `(( ))` |
|---------|-------|---------|---------|
| POSIX compatible | ✓ | ✗ | ✗ |
| Works for file tests | ✓ | ✓ | ✗ |
| Works for string tests | ✓ | ✓ | ✗ |
| Works for arithmetic | ~ | ~ | ✓ |
| Pattern matching | ✗ | ✓ | ✗ |
| Short-circuit `&&`/`\|\|` | ✓ | ✓ | ✓ |

**Rule of thumb:**
- Use `[ ]` when you need maximum portability (scripts that run on many systems)
- Use `[[ ]]` for modern Bash scripts with pattern matching and pattern guards
- Use `(( ))` when doing arithmetic comparisons

### Critical Rule: Always Quote Variables

The most important rule in testing is: **always quote your variables**

```bash
# Dangerous (unquoted)
if [ $var = "test" ]; then
  echo "Matched"
fi
# If $var is unset or contains spaces, this breaks unpredictably

# Correct (quoted)
if [ "$var" = "test" ]; then
  echo "Matched"
fi
# Even if $var is unset or has spaces, this works safely
```

---

## 5.2. File Tests

File tests check the properties and status of files and directories.

### File Existence and Type Tests

| Operator | Test | Meaning |
|----------|------|---------|
| `-f` | File exists and is regular file | Can read/write/execute a regular file |
| `-d` | File exists and is directory | Can navigate into it with `cd` |
| `-e` | File exists (any type) | File exists (less specific than `-f` or `-d`) |
| `-L` or `-h` | File is symbolic link | Points to another file (may be broken) |
| `-b` | File is block device | Hardware device (disk, USB, etc.) |
| `-c` | File is character device | Character-based hardware (terminal, printer) |
| `-p` | File is pipe/FIFO | Named pipe for inter-process communication |
| `-S` | File is socket | Network socket |

**Why these distinctions matter:**

```bash
#!/bin/bash

# Regular file test: safe for reading/writing
if [ -f "$filename" ]; then
  cat "$filename"  # Safely read file
fi

# Directory test: safe for `cd` and listing
if [ -d "$dirname" ]; then
  ls "$dirname"    # Safely list directory
fi

# Symbolic link test: detect broken links
if [ -L "$link" ]; then
  if [ ! -e "$link" ]; then
    echo "$link is a broken symbolic link"
  fi
fi
```

### File Permissions and Properties Tests

| Operator | Test | Meaning |
|----------|------|---------|
| `-r` | File is readable by current user | Can use `cat`, `grep`, etc. |
| `-w` | File is writable by current user | Can use `>`, `>>` redirection |
| `-x` | File is executable by current user | Can run as a script/program |
| `-s` | File exists and size > 0 | Has content (not empty) |
| `-O` | File is owned by current user | Ownership matches your user |
| `-G` | File's group matches current user's group | Group ownership matches your group |

**Practical examples:**

```bash
#!/bin/bash

# Check if you can read a file before processing it
if [ -r "$config_file" ]; then
  source "$config_file"
else
  echo "Cannot read configuration file: $config_file" >&2
  exit 1
fi

# Check if file is executable before running it
if [ -x "$script" ]; then
  "$script"
else
  echo "$script is not executable. Make it executable with: chmod +x $script" >&2
fi

# Check if log file has content before analyzing
if [ -s "$logfile" ]; then
  echo "Analyzing $logfile..."
  grep ERROR "$logfile"
else
  echo "$logfile is empty or doesn't exist"
fi

# Check ownership to prevent permission errors
if [ -O "$database" ]; then
  sqlite3 "$database" ".dump"
else
  echo "You don't own $database; cannot backup"
fi
```

### File Comparison Tests

| Operator | Test | Meaning |
|----------|------|---------|
| `f1 -nt f2` | File f1 is **newer** than f2 | Based on modification time |
| `f1 -ot f2` | File f1 is **older** than f2 | Based on modification time |
| `f1 -ef f2` | Files are **hard links** to same inode | Same file with multiple names |

**Understanding file comparison:**

```bash
#!/bin/bash

file1="config.txt"
file2="config.txt.bak"

# Which is more recent?
if [ "$file1" -nt "$file2" ]; then
  echo "$file1 has been modified since the backup was made"
  echo "Consider re-running the backup"
fi

# Are these actually the same file?
if [ "$file1" -ef "$file2" ]; then
  echo "These are the same file (hard link)"
  echo "Modifying one affects the other"
fi

# Backup strategy: only backup if source is newer
if [ "$source" -nt "$dest" ]; then
  echo "Source is newer, updating backup..."
  cp "$source" "$dest"
else
  echo "Backup is current, no update needed"
fi
```

### Negation: Reversing Test Results

The `!` operator reverses the result of a test:

```bash
# Test returns true if condition is FALSE
if [ ! -f "$filename" ]; then
  echo "$filename does not exist"
fi

# Equivalent long form (rarely used)
if [ -f "$filename" ]; then
  # do nothing
else
  echo "$filename does not exist"
fi

# Common pattern: exit if file doesn't exist
if [ ! -f "$required_file" ]; then
  echo "Error: $required_file not found" >&2
  exit 1
fi
```

### Example 5-1: Practical File Testing

```bash
#!/bin/bash
# backup_checker.sh - Verify backup integrity

backup_file="$1"

# Check command-line argument
if [ $# -eq 0 ]; then
  echo "Usage: $0 <backup_file>" >&2
  exit 1
fi

# Check if file exists
if [ ! -e "$backup_file" ]; then
  echo "Error: $backup_file does not exist" >&2
  exit 1
fi

# Check if it's a regular file (not directory or device)
if [ ! -f "$backup_file" ]; then
  echo "Error: $backup_file is not a regular file" >&2
  exit 1
fi

# Check if readable
if [ ! -r "$backup_file" ]; then
  echo "Error: $backup_file is not readable" >&2
  exit 1
fi

# Check if it has content
if [ ! -s "$backup_file" ]; then
  echo "Warning: $backup_file is empty!" >&2
fi

# All checks passed
echo "✓ $backup_file exists, is a regular file, is readable"
echo "✓ File size: $(stat -c%s "$backup_file") bytes"
```

---

## 5.3. Integer Comparison Operators

Integer comparisons work on variables containing numeric values. These are **different** from string comparisons and use different operators.

### Integer Comparison Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `-eq` | Equal to | `[ "$a" -eq "$b" ]` |
| `-ne` | Not equal to | `[ "$a" -ne "$b" ]` |
| `-gt` | Greater than | `[ "$a" -gt "$b" ]` |
| `-ge` | Greater than or equal | `[ "$a" -ge "$b" ]` |
| `-lt` | Less than | `[ "$a" -lt "$b" ]` |
| `-le` | Less than or equal | `[ "$a" -le "$b" ]` |

**Why numeric operators exist:**

```bash
# String comparison vs. numeric comparison

a="10"
b="9"

# String comparison: alphabetical order
if [ "$a" \< "$b" ]; then
  echo "10 < 9 (alphabetically, '1' comes before '9')"
fi

# Numeric comparison: actual numbers
if [ "$a" -gt "$b" ]; then
  echo "10 > 9 (numerically)"
fi
```

### Arithmetic Context with (( ))

Inside `(( ))`, use C-style operators instead of Bash operators:

```bash
a=10
b=9

# Bash style (works everywhere)
if [ "$a" -gt "$b" ]; then
  echo "10 is greater than 9"
fi

# Arithmetic style (only in (( )))
if (( a > b )); then
  echo "10 is greater than 9"
fi

# More readable for complex arithmetic
if (( (a + b) * 2 > 30 )); then
  echo "Complex calculation result is greater than 30"
fi
```

### Example 5-2: Argument Validation with Integer Tests

```bash
#!/bin/bash
# validate_age.sh - Validate user age argument

if [ $# -ne 1 ]; then
  echo "Usage: $0 <age>" >&2
  exit 1
fi

age="$1"

# Test if argument is a number (basic check)
if ! [[ "$age" =~ ^[0-9]+$ ]]; then
  echo "Error: age must be a number" >&2
  exit 1
fi

# Test age ranges
if [ "$age" -lt 0 ]; then
  echo "Age cannot be negative"
  exit 1
elif [ "$age" -eq 0 ]; then
  echo "Not born yet?"
  exit 1
elif [ "$age" -lt 13 ]; then
  echo "You're a child"
elif [ "$age" -lt 18 ]; then
  echo "You're a teenager"
elif [ "$age" -lt 65 ]; then
  echo "You're an adult"
else
  echo "You're a senior"
fi
```

---

## 5.4. String Comparison Operators

String comparisons test the values of string variables using alphabetical (ASCII) order.

### String Comparison Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `=` | Equal (POSIX) | `[ "$a" = "$b" ]` |
| `==` | Equal (Bash, identical to `=`) | `[ "$a" == "$b" ]` |
| `!=` | Not equal | `[ "$a" != "$b" ]` |
| `-z` | String is null/empty | `[ -z "$a" ]` (true if empty) |
| `-n` | String is not null/empty | `[ -n "$a" ]` (true if not empty) |

### Understanding `-z` and `-n`

These tests check whether a string is empty:

```bash
#!/bin/bash

# Test 1: Uninitialized variable (empty)
unset my_var
if [ -z "$my_var" ]; then
  echo "my_var is empty (uninitialized)"
fi

# Test 2: Initialized but empty
my_var=""
if [ -z "$my_var" ]; then
  echo "my_var is empty (explicitly set to empty string)"
fi

# Test 3: Contains a value
my_var="hello"
if [ -n "$my_var" ]; then
  echo "my_var is not empty: '$my_var'"
fi

# Shorthand: bare variable test (implicit -n)
if [ "$my_var" ]; then
  echo "Shorthand: my_var is not empty"
fi
```

### String Comparison with [[ ]] — Pattern Matching

The `[[ ]]` construct allows pattern matching without globbing:

```bash
#!/bin/bash

filename="document.txt"

# Glob pattern matching (only in [[ ]])
if [[ "$filename" == *.txt ]]; then
  echo "It's a text file"
fi

# Regex matching (only in [[ ]])
if [[ "$filename" =~ ^[a-z]+\.[a-z]+$ ]]; then
  echo "Valid filename format"
fi

# With quoting, patterns are literal
if [[ "$filename" == "*.txt" ]]; then
  echo "Filename is literally '*.txt' (rare)"
fi
```

### String Comparison Order

Strings are compared **alphabetically** using ASCII order:

```bash
#!/bin/bash

# Alphabetical comparison
a="apple"
b="banana"
c="apricot"

if [ "$a" \< "$b" ]; then
  echo "apple comes before banana"  # TRUE
fi

if [ "$c" \< "$b" ]; then
  echo "apricot comes before banana"  # TRUE
fi

if [ "$a" \< "$c" ]; then
  echo "apple comes before apricot"  # TRUE (compare character by character)
fi
```

### Example 5-3: String Validation

```bash
#!/bin/bash
# validate_email.sh - Basic email validation

email="$1"

# Check if argument provided
if [ -z "$email" ]; then
  echo "Usage: $0 <email>" >&2
  exit 1
fi

# Check if contains @ symbol
if [[ "$email" == *@* ]]; then
  echo "✓ Email contains @ symbol"
else
  echo "✗ Email must contain @ symbol"
  exit 1
fi

# Check if contains . after @
if [[ "$email" == *@*.* ]]; then
  echo "✓ Email contains domain"
else
  echo "✗ Email must have proper domain"
  exit 1
fi

# Check format with regex (bash)
if [[ "$email" =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
  echo "✓ Email format is valid"
else
  echo "✗ Email format is invalid"
fi
```

---

## 5.5. Compound Comparison Operators

Combine multiple tests with logical operators to create complex conditions.

### Logical AND and OR

**Using `[ ]` with `-a` and `-o`:**

```bash
# AND: both conditions must be true
if [ -f "$file" -a -r "$file" ]; then
  echo "$file exists and is readable"
fi

# OR: at least one condition must be true
if [ ! -f "$file" -o ! -r "$file" ]; then
  echo "$file doesn't exist OR is not readable"
fi
```

**Using `[[ ]]` with `&&` and `||` (preferred):**

```bash
# AND: both conditions must be true
if [[ -f "$file" && -r "$file" ]]; then
  echo "$file exists and is readable"
fi

# OR: at least one condition must be true
if [[ ! -f "$file" || ! -r "$file" ]]; then
  echo "$file doesn't exist OR is not readable"
fi
```

### Understanding Short-Circuit Evaluation

Short-circuit evaluation means the second condition is **not evaluated** if the result is already determined:

```bash
#!/bin/bash

# Short-circuit AND (&&)
# If first is false, second is never checked
if [ "$count" -gt 0 ] && [ 1 -eq "$array[$count]" ]; then
  echo "Array has elements and first is 1"
fi

# Short-circuit OR (||)
# If first is true, second is never checked
if [ "$user" = "root" ] || [ "$user" = "admin" ]; then
  echo "User has privileges"
fi

# Practical example: exit early if condition fails
if [ ! -f "$config" ]; then
  echo "Config not found" >&2
  exit 1
fi
# Only executed if $config file exists
source "$config"
```

### Combining Multiple Conditions

```bash
#!/bin/bash

file="$1"

# Multiple AND conditions: all must be true
if [ -f "$file" ] && [ -r "$file" ] && [ -s "$file" ]; then
  echo "$file exists, is readable, and has content"
  wc -l "$file"
fi

# Mixing AND and OR
if [ "$user" = "root" ] || ([ -w "$file" ] && [ ! -L "$file" ]); then
  echo "You have permission to modify this file"
fi

# Nested conditions (same as AND)
if [ -d "$dir" ]; then
  if [ -w "$dir" ]; then
    echo "You can write to $dir"
  fi
fi
```

### Example 5-4: Complex Decision Making

```bash
#!/bin/bash
# backup.sh - Selective backup based on conditions

source_file="$1"
backup_dir="$2"

# Validate arguments
if [ -z "$source_file" ] || [ -z "$backup_dir" ]; then
  echo "Usage: $0 <source_file> <backup_dir>" >&2
  exit 1
fi

# Check source exists and is readable
if [ ! -f "$source_file" ] || [ ! -r "$source_file" ]; then
  echo "Error: Cannot read $source_file" >&2
  exit 1
fi

# Check backup dir exists and is writable
if [ ! -d "$backup_dir" ] || [ ! -w "$backup_dir" ]; then
  echo "Error: Cannot write to $backup_dir" >&2
  exit 1
fi

# Determine backup filename
backup_file="$backup_dir/$(basename "$source_file").bak"

# Only backup if source is newer or backup doesn't exist
if [ ! -f "$backup_file" ] || [ "$source_file" -nt "$backup_file" ]; then
  echo "Backing up $source_file..."
  cp "$source_file" "$backup_file"
  echo "✓ Backup complete: $backup_file"
else
  echo "Backup is current, skipping"
fi
```

---

## Programming Keywords and Concepts

### Essential Testing Keywords

| Keyword/Operator | Type | Purpose |
|---|---|---|
| `[ ]` | Syntax | POSIX test construct |
| `[[ ]]` | Syntax | Bash extended test (pattern matching) |
| `(( ))` | Syntax | Arithmetic test and evaluation |
| `-f` | File test | Regular file existence |
| `-d` | File test | Directory existence |
| `-r`, `-w`, `-x` | Permission test | Read, write, execute permissions |
| `-z`, `-n` | String test | Empty/non-empty string tests |
| `-eq`, `-ne`, `-gt`, `-lt` | Numeric test | Equality and comparison |
| `&&` | Logical operator | Short-circuit AND |
| `\|\|` | Logical operator | Short-circuit OR |
| `!` | Logical operator | Negation (NOT) |

### Core Programming Concepts

**1. Testing Conditions**
- Conditions evaluate to true or false
- Every command in Bash returns an exit code (0 = success/true, non-zero = failure/false)
- Tests are commands that return exit codes based on their condition
- Use exit codes to make decisions with `if`, `while`, etc.

**2. Exit Codes and $?**
- Every command sets `$?` to its exit code
- `$?` = 0 means success/true
- `$?` != 0 means failure/false
- Allows chaining commands based on success/failure

**3. File Testing Importance**
- Always verify files exist before using them
- Check permissions before attempting operations
- Validate file types (regular file vs. directory) before processing
- Prevents cryptic errors from invalid operations

**4. String vs. Numeric Comparison**
- `-eq`, `-ne`, `-gt`, `-lt` are for **numbers**
- `=`, `!=`, `<`, `>` are for **strings** (alphabetical order)
- Using wrong operator gives wrong results (e.g., "10" < "9" alphabetically!)
- Must choose correct operator for your data type

**5. Quoting Variables in Tests**
- Always quote variables: `[ "$var" = "test" ]` not `[ $var = test ]`
- Unquoted variables undergo word-splitting and glob expansion
- Word-splitting can cause unpredictable test behavior
- Quoting prevents subtle bugs in production scripts

**6. Short-Circuit Evaluation**
- `&&` stops evaluating if first condition is false
- `||` stops evaluating if first condition is true
- Allows "guard" patterns: `[ -f "$file" ] && read data < "$file"`
- Prevents errors from evaluating invalid conditions

**7. Test Construct Selection**
- `[ ]` — POSIX portable, works everywhere
- `[[ ]]` — Modern Bash, enables pattern matching and safer variable handling
- `(( ))` — Pure arithmetic, can't use file/string tests
- Choose based on portability needs and feature requirements

**8. Logical Combination Patterns**
- AND (&&): All conditions must be true
- OR (||): At least one must be true
- NOT (!): Reverses truth value
- Combine to express complex business logic

**9. File Integrity Checking**
- `-e` — File exists (any type)
- `-f` — Is regular file (safe for reading)
- `-d` — Is directory (safe for `cd` and listing)
- `-r/-w/-x` — Permission checks before operations
- `-s` — Has content (not empty)

**10. Defensive Programming**
- Validate all inputs and files before using them
- Use tests to prevent errors before they happen
- Check file existence, readability, ownership
- Fail fast with clear error messages
- Use exit codes to signal success/failure

---

## Common Testing Patterns

### Pattern 1: Guard Clauses (Fail Early)

```bash
#!/bin/bash

script_init() {
  # Exit immediately if any required file missing
  [ -f "$config" ] || { echo "Config not found" >&2; exit 1; }
  [ -d "$data_dir" ] || { echo "Data dir not found" >&2; exit 1; }
  [ -x "$binary" ] || { echo "Binary not executable" >&2; exit 1; }
  # Only reach here if all checks passed
}
```

### Pattern 2: Selective Processing Based on Tests

```bash
# Only process if file exists AND is readable
if [ -f "$logfile" ] && [ -r "$logfile" ]; then
  grep ERROR "$logfile"
else
  echo "Cannot read logfile"
fi
```

### Pattern 3: Conditional Backup

```bash
# Only backup if source is newer than backup
backup_file="$1.backup"
if [ ! -f "$backup_file" ] || [ "$1" -nt "$backup_file" ]; then
  cp "$1" "$backup_file"
  echo "Backup created"
fi
```

---

## Summary

Testing is fundamental to robust scripting:

- **File tests** check existence (`-e`, `-f`, `-d`), permissions (`-r`, `-w`, `-x`), and properties (`-s`, `-L`)
- **Integer comparisons** use `-eq`, `-ne`, `-gt`, `-lt`, `-ge`, `-le` (or `==`, `!=`, `>`, `<` in `(( ))`)
- **String comparisons** use `=`, `!=`, `-z` (null), `-n` (not null), with alphabetical ordering
- **Always quote variables** in tests: `[ "$var" ]` not `[ $var ]`
- **Use `[[ ]]`** for modern Bash with pattern matching and better handling of quoting
- **Combine conditions** with `&&` (AND) and `||` (OR) for complex logic
- **Short-circuit evaluation** allows guard patterns to fail fast
- **Test before using** — always verify files exist and have correct properties before processing

Master testing, and your scripts will be more robust, reliable, and maintainable.
