# Chapter 5: Testing and Comparisons

Testing is central to writing robust Bash scripts. The `test` command (also spelled `[ ]`) and its extended form `[[ ]]` allow you to evaluate conditions and make decisions based on file properties, variable values, and logical combinations.

## 5.1. File Tests

File tests check the properties and status of files and directories.

{% tabs %}
{% tab title="File Existence & Type" %}

| Operator | Test |
|----------|------|
| `-f` | File exists and is a regular file |
| `-d` | File exists and is a directory |
| `-e` | File exists (any type) |
| `-L` or `-h` | File is a symbolic link |
| `-b` | File is a block device |
| `-c` | File is a character device |
| `-p` | File is a pipe |
| `-S` | File is a socket |

**Examples:**
```bash
if [ -f "$filename" ]
then
  echo "$filename is a regular file"
fi

if [ -d "$dirname" ]
then
  echo "$dirname is a directory"
fi

if [ -h "$link" ]
then
  echo "$link is a symbolic link"
fi
```

{% endtab %}
{% tab title="File Permissions & Properties" %}

| Operator | Test |
|----------|------|
| `-r` | File is readable by current user |
| `-w` | File is writable by current user |
| `-x` | File is executable by current user |
| `-s` | File exists and has a size greater than 0 |
| `-O` | File is owned by the current user |
| `-G` | File's group matches current user's group |

**Examples:**
```bash
if [ -r "$file" ]
then
  echo "You can read $file"
fi

if [ -x "$script" ]
then
  echo "$script is executable"
fi

if [ -s "$logfile" ]
then
  echo "$logfile has content"
fi
```

{% endtab %}
{% tab title="File Comparisons" %}

| Operator | Test |
|----------|------|
| `f1 -nt f2` | File f1 is **newer** than f2 (based on modification time) |
| `f1 -ot f2` | File f1 is **older** than f2 |
| `f1 -ef f2` | Files f1 and f2 are **hard links** to the same inode |

**Examples:**
```bash
if [ "$file1" -nt "$file2" ]
then
  echo "$file1 was modified after $file2"
fi

if [ "$file1" -ef "$file2" ]
then
  echo "$file1 and $file2 are the same file"
fi
```

{% endtab %}
{% tab title="Negation" %}

The `!` operator reverses the sense of a test (returns true if the condition is absent):

```bash
if [ ! -f "$filename" ]
then
  echo "$filename does not exist"
fi

if [ ! -d "$dirname" ]
then
  echo "$dirname is not a directory"
fi
```

{% endtab %}
{% endtabs %}

### Example 5-1: Testing File Properties

```bash
#!/bin/bash
# Testing file properties

file="/etc/passwd"

echo "Testing $file:"
echo

if [ -e "$file" ]
then
  echo "$file exists"
fi

if [ -f "$file" ]
then
  echo "$file is a regular file"
fi

if [ -r "$file" ]
then
  echo "$file is readable"
fi

if [ -s "$file" ]
then
  echo "$file has content (non-zero size)"
fi

if [ ! -z "$(find "$file" -mmin -60)" ]
then
  echo "$file was modified in the last 60 minutes"
fi
```

### Example 5-2: Comparing File Timestamps

```bash
#!/bin/bash
# Comparing file modification times

file1="$1"
file2="$2"

if [ ! -f "$file1" ] || [ ! -f "$file2" ]
then
  echo "Usage: $0 file1 file2"
  exit 1
fi

if [ "$file1" -nt "$file2" ]
then
  echo "$file1 is newer than $file2"
elif [ "$file1" -ot "$file2" ]
then
  echo "$file1 is older than $file2"
else
  echo "$file1 and $file2 have the same modification time"
fi

if [ "$file1" -ef "$file2" ]
then
  echo "Note: $file1 and $file2 are the same file (hard link)"
fi
```

### Example 5-4: Testing for Broken Links

```bash
#!/bin/bash
# broken-link.sh: Find dead symlinks in a directory tree
# A symbolic link is "broken" if the target it points to no longer exists

linkchk() {
  for element in "$1"/*; do
    if [ -h "$element" ] && [ ! -e "$element" ]
    then
      echo "Broken link: $element"
    fi
    
    if [ -d "$element" ]
    then
      linkchk "$element"  # Recursively check subdirectories
    fi
  done
}

# Process directories passed as arguments (or current directory)
if [ $# -eq 0 ]
then
  directorys=$(pwd)
else
  directorys="$@"
fi

for directory in $directorys; do
  if [ -d "$directory" ]
  then
    echo "Checking $directory for broken links..."
    linkchk "$directory"
  else
    echo "$directory is not a directory"
  fi
done

exit $?
```

---

## 5.2. Integer Comparison Operators

Integer comparisons work on variables containing numeric values. These operators are used within `[ ]` or `[[ ]]` constructs, or within `(( ))` arithmetic contexts.

{% tabs %}
{% tab title="Integer Operators" %}

| Operator | Meaning |
|----------|---------|
| `-eq` | Is equal to |
| `-ne` | Is not equal to |
| `-gt` | Is greater than |
| `-ge` | Is greater than or equal to |
| `-lt` | Is less than |
| `-le` | Is less than or equal to |

**Usage with `[ ]` (single brackets):**
```bash
if [ "$a" -eq "$b" ]
then
  echo "$a equals $b"
fi
```

**Usage with `(( ))` (arithmetic context):**
```bash
if (( a == b ))
then
  echo "$a equals $b"
fi
```

Arithmetic context allows C-style operators: `<`, `<=`, `>`, `>=`, `==`, `!=`.

{% endtab %}
{% endtabs %}

### Example 5-5: Arithmetic and String Comparisons

```bash
#!/bin/bash
# Demonstrating integer vs. string comparison

a=4
b=5

echo "Comparing a=$a and b=$b"
echo

# Arithmetic (numeric) comparison
if [ "$a" -ne "$b" ]
then
  echo "$a is not equal to $b (numeric comparison with -ne)"
fi

echo

# String comparison
if [ "$a" != "$b" ]
then
  echo "$a is not equal to $b (string comparison with !=)"
fi

echo

# Both work here, but results can differ with non-numeric strings
x="10"
y="9"

echo "Comparing x=$x and y=$y"
echo

if [ "$x" -gt "$y" ]
then
  echo "$x is greater than $y (numeric)"
fi

if [ "$x" \> "$y" ]
then
  echo "$x is greater than $y (string, ASCII order)"
fi

exit 0
```

---

## 5.3. String Comparison Operators

String comparisons test the values of string variables. These use a different set of operators than integer comparisons.

{% tabs %}
{% tab title="String Operators" %}

| Operator | Meaning |
|----------|---------|
| `=` | Is equal to (note: whitespace around `=` is critical) |
| `==` | Is equal to (synonym for `=`) |
| `!=` | Is not equal to |
| `-z` | String is null (zero length) |
| `-n` | String is not null |

**Comparison with `[ ]`:**
```bash
if [ "$a" = "$b" ]
then
  echo "Strings are equal"
fi

if [ "$a" != "$b" ]
then
  echo "Strings are not equal"
fi
```

**Comparison with `[[ ]]` (allows pattern matching):**
```bash
if [[ "$a" == z* ]]
then
  echo "$a starts with 'z'"
fi

if [[ "$a" == "z*" ]]
then
  echo "$a is exactly 'z*'"
fi
```

**Important:** With `[[ ]]`, unquoted patterns use glob matching; with `[ ]`, this can cause unpredictable behavior. Always quote strings in comparisons.

{% endtab %}
{% tab title="Alphabetic Ordering" %}

String comparisons use **ASCII alphabetic order**:

```bash
if [[ "$a" < "$b" ]]
then
  echo "$a comes before $b alphabetically"
fi

# With single brackets, < must be escaped:
if [ "$a" \< "$b" ]
then
  echo "$a comes before $b"
fi
```

**Example:**
```bash
a="apple"
b="banana"

if [ "$a" \< "$b" ]
then
  echo "apple comes before banana"
fi
```

{% endtab %}
{% endtabs %}

### Example 5-6: Testing for Null Strings

```bash
#!/bin/bash
# Testing null, uninitialized, and empty strings

# Test 1: Uninitialized variable (dangerous without quoting)
if [ -n $uninitialized ]
then
  echo "Uninitialized is not null (wrong!)"
else
  echo "Uninitialized is null"
fi

echo

# Test 2: Uninitialized variable with proper quoting
if [ -n "$uninitialized" ]
then
  echo "Uninitialized is not null"
else
  echo "Uninitialized is null (correct!)"
fi

echo

# Test 3: Bare variable test (works, but less explicit)
if [ "$uninitialized" ]
then
  echo "Uninitialized is not null"
else
  echo "Uninitialized is null (correct)"
fi

echo

# Test 4: After initialization
string="initialized"
if [ -n "$string" ]
then
  echo "String \"$string\" is not null"
fi

echo

# Test 5: String with spaces (demonstrates quoting importance)
string="a = b"
if [ $string ]      # Unquoted: word splitting occurs
then
  echo "Unquoted: String detected as non-null (misleading)"
fi

if [ "$string" ]    # Quoted: correct
then
  echo "Quoted: String detected as non-null (correct)"
fi

exit 0
```

**Key Insight:** Always quote variables in test brackets: `[ "$var" ]` rather than `[ $var ]`. An unquoted variable may undergo word splitting and cause unexpected results.

### Example 5-7: A Practical Utility Script

```bash
#!/bin/bash
# zmore: View gzipped files with 'more' filter
# Usage: zmore filename.gz

E_NOARGS=85
E_NOTFOUND=86
E_NOTGZIP=87

# Check for command-line argument
if [ $# -eq 0 ]
then
  echo "Usage: $(basename $0) filename.gz" >&2
  exit $E_NOARGS
fi

filename="$1"

# Check if file exists
if [ ! -f "$filename" ]
then
  echo "File $filename not found!" >&2
  exit $E_NOTFOUND
fi

# Check if file is gzipped (extension .gz)
if [ "${filename##*.}" != "gz" ]
then
  echo "File $filename is not a gzipped file!" >&2
  exit $E_NOTGZIP
fi

# Decompress and display
zcat "$filename" | more

exit $?
```

---

## 5.4. Compound Comparison Operators

Combine multiple test conditions using logical operators.

{% tabs %}
{% tab title="Logical AND / OR" %}

**Single Brackets `[ ]`:**
```bash
if [ condition1 -a condition2 ]
then
  echo "Both conditions are true"
fi

if [ condition1 -o condition2 ]
then
  echo "At least one condition is true"
fi
```

**Double Brackets `[[ ]]` (preferred):**
```bash
if [[ condition1 && condition2 ]]
then
  echo "Both conditions are true"
fi

if [[ condition1 || condition2 ]]
then
  echo "At least one condition is true"
fi
```

**Important:** `-a` and `-o` do **not** short-circuit (evaluate all conditions), while `&&` and `||` do short-circuit (stop evaluating once result is determined).

{% endtab %}
{% tab title="Nested if/then" %}

Nested conditions are equivalent to compound logical AND:

```bash
a=3

# Nested approach
if [ "$a" -gt 0 ]
then
  if [ "$a" -lt 5 ]
  then
    echo "a is between 0 and 5"
  fi
fi

# Equivalent compound approach
if [ "$a" -gt 0 ] && [ "$a" -lt 5 ]
then
  echo "a is between 0 and 5"
fi

# Equivalent with double brackets
if [[ "$a" -gt 0 && "$a" -lt 5 ]]
then
  echo "a is between 0 and 5"
fi
```

{% endtab %}
{% endtabs %}

### Example 5-8: Compound Tests

```bash
#!/bin/bash
# Demonstrating compound comparison operators

a=5
b=10

echo "Testing a=$a and b=$b:"
echo

# Using && (short-circuit AND)
if [ "$a" -gt 0 ] && [ "$b" -gt "$a" ]
then
  echo "a is positive AND b is greater than a"
fi

echo

# Using || (short-circuit OR)
if [ "$a" -lt 0 ] || [ "$a" -gt 0 ]
then
  echo "a is either negative or positive (not zero)"
fi

echo

# Checking file conditions
if [ -f "$HOME/.bashrc" ] && [ -r "$HOME/.bashrc" ]
then
  echo "Your .bashrc exists and is readable"
fi

echo

# Complex condition
if [ -d "$HOME" ] && [ -w "$HOME" ] || [ "$USER" = "root" ]
then
  echo "You have write access to your home directory (or you're root)"
fi

exit 0
```

---

## 5.5. Test Constructs Comparison

| Test Form | Usage | Short-circuits | Pattern Matching |
|-----------|-------|----------------|-----------------|
| `[ ]` | POSIX compatible | `&&`, `\|\|` only | No |
| `[[ ]]` | Bash extended | `&&`, `\|\|` yes | Yes (unquoted) |
| `(( ))` | Arithmetic only | `&&`, `\|\|` yes | N/A |

**Choosing the right construct:**
- Use `[ ]` for POSIX portability
- Use `[[ ]]` for modern Bash with pattern matching
- Use `(( ))` for pure arithmetic comparisons
- Always quote variables: `[ "$var" ]`

---

## 5.6. Real-World Example: xinitrc Configuration

```bash
#!/bin/bash
# Simplified xinitrc: Launch X Window System with fallback options

if [ -f "$HOME/.Xclients" ]
then
  exec "$HOME/.Xclients"
elif [ -f /etc/X11/xinit/Xclients ]
then
  exec /etc/X11/xinit/Xclients
else
  # Fallback: launch default components
  xclock -geometry 100x100-5+5 &
  xterm -geometry 80x50-50+150 &
  
  # Launch Netscape if both browser and documentation exist
  if [ -f /usr/bin/netscape ] && [ -f /usr/share/doc/HTML/index.html ]
  then
    netscape /usr/share/doc/HTML/index.html &
  fi
fi
```

**Explanation:**
- Line 3: Check if user has custom `.Xclients` file
- Line 6: Check if system has default `.Xclients`
- Lines 9-14: If neither exists, launch defaults
- Line 13: Compound test using `-a` (and) to verify both files exist

---

## Exercises

1. **Write a script** that checks if a file exists, is readable, and has a size greater than zero. If all conditions are true, display the file size.

2. **Create a backup utility** that compares the modification times of two files and copies the newer one to a backup location using `-nt`.

3. **Write a script** that tests whether a variable is null and handles both quoted and unquoted versions to show the difference.

4. **Implement a permission checker** that verifies if the current user can read, write, and execute a given file, displaying appropriate messages.

5. **Create a recursive directory scanner** that finds and reports all broken symbolic links (targets that no longer exist).

6. **Write a script** that demonstrates the difference between integer comparison (`-eq`, `-lt`) and string comparison (`=`, `<`) with the same variables.

---

## Summary

- **File tests** check existence (`-e`, `-f`, `-d`), permissions (`-r`, `-w`, `-x`), and properties (`-s`, `-L`)
- **File comparisons** test modification time (`-nt`, `-ot`) and hard links (`-ef`)
- **Integer comparisons** use `-eq`, `-ne`, `-gt`, `-lt`, `-ge`, `-le` (or `==`, `!=`, `>`, `<` in `(( ))`)
- **String comparisons** use `=`, `==`, `!=`, `-z` (null), `-n` (not null)
- **Always quote variables** in test brackets: `[ "$var" ]`
- **Use `[[ ]]`** for modern Bash with pattern matching and short-circuit behavior
- **Compound tests** combine conditions with `&&`, `||`, `-a`, `-o`
- **Short-circuit evaluation:** `&&` and `||` stop evaluating once result is determined; `-a` and `-o` do not
