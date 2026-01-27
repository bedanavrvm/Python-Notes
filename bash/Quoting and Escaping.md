# Chapter 5: Quoting and Escaping

Quoting and escaping are fundamental techniques for controlling how the shell interprets special characters and spaces in your scripts. Proper use of quotes and escapes prevents word splitting, variable expansion, and command interpretation in unintended ways.

## 5.1. Double Quotes (Partial Quoting / Weak Quoting)

Double quotes allow variable and command substitution, but prevent globbing (wildcard expansion) and most special character interpretation.

{% tabs %}
{% tab title="Double Quotes Rules" %}

Within double quotes (`"..."`):
- **Variable substitution occurs:** `$var` is expanded to its value
- **Command substitution occurs:** `` $(...)`` or `` `...` `` is executed
- **Escape character works:** `\` can escape special characters
- **Globbing is prevented:** `*`, `?`, `[...]` are treated literally
- **Whitespace is preserved:** Spaces and tabs are kept intact

**Example:**
```bash
var="hello world"
echo "$var"                # hello world (spaces preserved)
echo "Today is $(date)"    # Command substitution works
echo "Cost: \$5.99"        # \$ escapes the dollar sign
```

{% endtab %}
{% tab title="Nested Quotes" %}

You can nest quotes of different types within double quotes:

```bash
echo "$(echo '"')"         # Outputs: "
echo "\$var = "$var""      # Variable inside and outside quotes
```

This allows flexible formatting of complex strings.

{% endtab %}
{% endtabs %}

### Escaping Within Double Quotes

The backslash `\` can escape the following within double quotes:

- **`\"`** — Literal double quote
- **`\$`** — Literal dollar sign (prevents variable expansion)
- **`\\`** — Literal backslash
- **`\n`** — Newline (with `echo -e` or `$'...'` syntax)

```bash
echo "\"Hello\" he said."   # "Hello" he said.
echo "\$5"                   # $5
echo "\\"                    # \
```

### Example 5-1: Complex Quoting Scenarios

```bash
#!/bin/bash
# Nested quoting and complex variable expansion

# Escaping within double quotes
var2="\\\\\""
echo $var2      # " (two backslashes become one, \" becomes ")
echo "$var2"    # \\" (preserves the literal backslash sequence)

# Strong quoting with backslashes
var3='\\\\'
echo "$var3"    # \\\\ (single quotes treat backslash literally)

# Nesting quotes
echo "$(echo '"')"  # " (single quote inside double quote inside command substitution)
var1="Two bits"
echo "\$var1 = "$var1""  # $var1 = Two bits

# File comparison with proper quoting
if [[ "$(du "$My_File1")" -gt "$(du "$My_File2")" ]]; then
  echo "File1 is larger"
fi
```

---

## 5.2. Single Quotes (Full Quoting / Strong Quoting)

Single quotes operate strictly: they prevent **all** variable expansion, command substitution, and escape processing. Every character within single quotes is treated literally except the single quote itself.

{% tabs %}
{% tab title="Single Quotes Rules" %}

Within single quotes (`'...'`):
- **No variable substitution:** `$var` is printed literally
- **No command substitution:** `` $(...)`` and `` `...` `` are not executed
- **No escape processing:** `\` is treated as a literal character
- **All special characters are literal:** `*`, `?`, `#`, `&`, etc. are all literal

**Example:**
```bash
var="hello"
echo '$var'      # $var (not expanded)
echo '$HOME'     # $HOME (literal, not home directory)
echo '\t\n'      # \t\n (escape sequences not processed)
```

{% endtab %}
{% tab title="Including Single Quotes" %}

Since the escape character has no special meaning within single quotes, including a single quote requires ending the quoted string, escaping the quote, and starting a new quoted string:

```bash
echo 'Why can'\''t I write'"'"'s between single quotes'
# |-------| |----------| |-----------------------|
# Three single-quoted strings, with escaped and quoted single quotes between.

# Equivalent (clearer) approach:
echo "Why can't I write 's between single quotes"
```

The first method breaks into three parts:
1. `'Why can'` — string up to the apostrophe
2. `\'` — escaped single quote (outside quotes)
3. `'t I write...'` — rest of string

{% endtab %}
{% endtabs %}

### Key Difference: Single vs. Double Quotes

```bash
var="HELLO"

# Double quotes: variable substitution
echo "The value is $var"     # The value is HELLO

# Single quotes: no substitution
echo 'The value is $var'     # The value is $var

# Double quotes: command substitution
echo "Today: $(date +%Y)"    # Today: 2026

# Single quotes: no command substitution
echo 'Today: $(date +%Y)'    # Today: $(date +%Y)
```

---

## 5.3. Escaping

Escaping is a method of quoting a **single character**. The backslash `\` preceding a character tells the shell to interpret that character literally.

### Special Escape Sequences

Certain commands like `echo` and `sed` recognize escape sequences (especially with `echo -e`):

| Sequence | Meaning |
|----------|---------|
| `\n` | Newline |
| `\r` | Carriage return |
| `\t` | Tab |
| `\v` | Vertical tab |
| `\b` | Backspace |
| `\a` | Alert (beep or flash) |
| `\0xx` | Octal ASCII value |

```bash
echo -e "Line 1\nLine 2"    # Prints two lines
echo -e "Col1\tCol2"        # Prints with tab separator
echo -e "\a"                # Beep
```

### The `$'...'` Construct (ANSI-C Quoting)

Bash 2.0+ supports the `$'...'` construct, which interprets escape sequences without needing `echo -e`:

```bash
echo $'\n'                  # Newline
echo $'\t\042\t'            # Tab, quote (octal 042), tab
echo $'\x22'                # Double quote (hex 22)
echo $'\033'                # Escape character (octal 033)
```

**Assigning ASCII characters to variables:**

```bash
quote=$'\042'               # " (octal 042 for double quote)
echo "$quote Hello $quote"  # "Hello"

triple_underline=$'\137\137\137'  # Octal 137 = underscore
echo "$triple_underline SECTION $triple_underline"

ABC=$'\101\102\103'         # Octal 101, 102, 103 = A, B, C
echo $ABC                   # ABC
```

### Example 5-2: Escaped Characters

```bash
#!/bin/bash
# escaped.sh: demonstrating escape sequences

# Escaping a newline (line continuation)
echo "This will print \
as one line."               # Newline escaped; prints as single line

# Without escape, newline in string splits across lines
echo "This will print
as two lines."

# Using -e flag with echo for escape sequences
echo -e "\v\v\v\v"          # Four vertical tabs
echo -e "\042"              # Double quote character

# Using $'...' construct (cleaner)
echo $'\n'                  # Newline
echo $'\a'                  # Alert/beep

echo "Demonstrating \$'...' construct:"
echo $'\t\042\t'            # Quote framed by tabs
echo $'\t\x22\t'            # Same, using hex notation

# Variable assignment with special characters
escape=$'\033'              # Escape character
echo "Invisible escape: $escape (no visible output)"
```

### Example 5-3: Detecting Key Presses

```bash
#!/bin/bash
# Advanced example: detecting special keys
# Requires Bash 4.2+

key="no value yet"

while true; do
  clear
  echo "Bash Key Detection Demo"
  echo "Try arrow keys, Insert, Delete, Home, End, etc."
  echo
  
  case "$key" in
    $'\x1b\x5b\x41')  # Up arrow
      echo "Up arrow pressed"
      ;;
    $'\x1b\x5b\x42')  # Down arrow
      echo "Down arrow pressed"
      ;;
    $'\x1b\x5b\x43')  # Right arrow
      echo "Right arrow pressed"
      ;;
    $'\x1b\x5b\x44')  # Left arrow
      echo "Left arrow pressed"
      ;;
    $'\x09')          # Tab
      echo "Tab key pressed"
      ;;
    $'\x1b')          # Escape
      echo "Escape key pressed"
      exit 0
      ;;
    d)
      date
      ;;
    *)
      echo "You pressed: $key"
      ;;
  esac
  
  echo "Press a key (d=date, q=quit):"
  read -s -N1 -p "> " K1
  read -s -N2 -t 0.001 K2
  read -s -N1 -t 0.001 K3
  key="$K1$K2$K3"
done
```

---

## 5.4. Practical Escaping Scenarios

### Escaping Special Characters

```bash
echo \z                     # z (backslash escapes z, but z has no special meaning)
echo \\z                    # \z (first \ escapes second \)
echo '\z'                   # \z (single quotes prevent interpretation)
echo '\\z'                  # \\z (two backslashes, literal)
echo "\z"                   # \z (\ without special meaning in double quotes)
echo "\\z"                  # \z (escaped backslash)
```

### Line Continuation

Ending a line with `\` escapes the newline and continues the command on the next line:

```bash
# Multi-line command with line continuation
(cd /source/directory && tar cf - . ) | \
(cd /dest/directory && tar xpvf -)

# Alternative: pipe at end of line doesn't require backslash
tar cf - -C /source/directory . |
tar xpvf - -C /dest/directory
```

### Escaping Spaces in Arguments

Escaping a space prevents word splitting:

```bash
file_list="/bin/cat /bin/gzip /bin/more"
ls -l /usr/bin/xsetroot /sbin/dump $file_list

# Escaping spaces concatenates arguments
ls -l /usr/bin/xsetroot\ /sbin/dump\ $file_list
# Now the escaped spaces prevent splitting, so the first three
# files are concatenated into a single argument to ls
```

### Behavior in Different Contexts

The behavior of `\` depends on its context:

```bash
# Simple escaping
echo \z                     # z
echo \\z                    # \z

# In single quotes (literal)
echo '\z'                   # \z
echo '\\z'                  # \\z

# In double quotes
echo "\z"                   # \z
echo "\\z"                  # \z

# In command substitution
echo `echo \\z`             # z (one level of backslash processing)
echo `echo \\\\z`           # \z (two levels)

# In here document
cat <<EOF
\z
EOF
# \z (literal backslash and z)
```

---

## 5.5. Assignment and Escaping

### Assigning Escaped Values

You can escape characters when assigning to variables:

```bash
variable=\\                 # Single backslash
echo "$variable"            # \

variable=\\\\               # Two backslashes
echo "$variable"            # \\
```

### Naked Escape Issues

A "naked" escape (backslash at end of line in assignment) cannot safely be assigned and will likely cause an error:

```bash
variable=\
echo "$variable"
# Error: command not found
# The \ escapes the newline, making the next line invalid syntax

# But this works:
variable=\
23skidoo
echo "$variable"            # 23skidoo (valid assignment on next line)

variable=\\
echo "$variable"            # \ (escaped backslash is valid)
```

---

## 5.6. Quoting Behavior Summary

| Feature | Double Quotes | Single Quotes | Escape (`\`) |
|---------|---------------|---------------|--------------|
| Variable substitution | ✓ | ✗ | N/A |
| Command substitution | ✓ | ✗ | N/A |
| Globbing | ✗ | ✗ | N/A |
| Whitespace preserved | ✓ | ✓ | N/A |
| Escape processing | ✓ | ✗ | ✓ |
| Special char literal | Partial | All | Per char |

### Choosing the Right Quoting

- **Use double quotes** when you need variable/command substitution but want to prevent globbing and preserve whitespace
- **Use single quotes** when you need complete literal interpretation and no substitution
- **Use escaping** for precise control over individual characters
- **Use `$'...'`** for readable ANSI-C escape sequences and special characters

---

## Exercises

1. **Write a script** that uses both double and single quotes to print a string containing variables and literal dollar signs. Show the difference.

2. **Use the `$'...'` construct** to assign a newline character to a variable and use it to format multi-line output.

3. **Demonstrate line continuation** with `\` by splitting a long command (e.g., `tar` or `find`) across multiple lines.

4. **Create a script** that safely handles filenames with spaces by properly quoting or escaping them.

5. **Use `echo -e` and escape sequences** to create a simple formatted table with tabs and alignment.

6. **Write a function** that takes a string containing special characters and safely echoes it using appropriate quoting techniques.

---

## Summary

- **Double quotes** (`"..."`) allow substitution but prevent globbing; use for most variable-heavy strings
- **Single quotes** (`'...'`) prevent all substitution and special processing; use for literal strings
- **Escaping** (`\`) quotes single characters; use `\\` for literal backslash
- **`$'...'` construct** (ANSI-C quoting) interprets escape sequences like `\n`, `\t`, `\xHH` (hex), `\OOO` (octal)
- **Line continuation** with `\` at end of line allows multi-line commands
- **Nested quoting** is possible: combine quote types strategically to handle complex strings
- Context matters: `\` behavior differs in single quotes, double quotes, command substitution, and here documents
