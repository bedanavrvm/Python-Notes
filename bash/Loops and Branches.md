# Chapter 9: Loops and Branches

Control flow statements direct the execution path through your script. This chapter covers case/esac for branching, pattern matching, and the select construct for building menus.

## 9.1. The `case/esac` Construct

The `case` construct is Bash's analog to `switch` in C/C++. It permits branching to one of several code blocks depending on condition tests and serves as a shorthand for multiple `if/then/else` statements.

### Syntax

```bash
case "$variable" in
  "pattern1")
    command...
    ;;
  "pattern2")
    command...
    ;;
  *)
    # Default case (optional)
    command...
    ;;
esac
```

**Key points:**
- Variables don't require quoting (no word splitting occurs)
- Each pattern ends with a right parenthesis `)`
- Each block ends with a double semicolon `;;`
- `*` matches any pattern (default case)
- If a pattern matches, associated commands execute and case block terminates
- `esac` (case spelled backwards) ends the entire construct

### Example 11-24: Character Classification

```bash
#!/bin/bash
# Character classification using case

echo "Hit a key, then press Enter:"
read -n1 keypress
echo

case "$keypress" in
  [[:lower:]])
    echo "Lowercase letter"
    ;;
  [[:upper:]])
    echo "Uppercase letter"
    ;;
  [0-9])
    echo "Digit"
    ;;
  *)
    echo "Punctuation, whitespace, or other character"
    ;;
esac

exit 0
```

**Character class support:**
- `[[:lower:]]` — Lowercase letters (POSIX portable)
- `[[:upper:]]` — Uppercase letters (POSIX portable)
- `[0-9]` — Digits
- `[a-z]` — Lowercase range
- `[A-Z]` — Uppercase range
- `[a-zA-Z]` — All letters
- `[!0-9]` — Non-digits (negation)

---

## 9.2. Case with Multiple Patterns

Multiple patterns can match the same code block using the pipe `|` operator.

{% tabs %}
{% tab title="Multiple Patterns" %}

```bash
case "$input" in
  yes|y|YES|Y)
    echo "You said yes"
    ;;
  no|n|NO|N)
    echo "You said no"
    ;;
  *)
    echo "Invalid input"
    ;;
esac
```

{% endtab %}
{% tab title="Glob Patterns" %}

Case patterns support glob wildcard syntax:

```bash
case "$filename" in
  *.txt)
    echo "Text file"
    ;;
  *.sh)
    echo "Shell script"
    ;;
  *.*)
    echo "File with extension"
    ;;
  *)
    echo "File without extension"
    ;;
esac
```

{% endtab %}
{% endtabs %}

### Example 11-25: Contact List Menu

```bash
#!/bin/bash
# Contact list using case menu

clear
echo "========== Contact List =========="
echo
echo "Choose a person:"
echo "[E]vans, Roland"
echo "[J]ones, Mildred"
echo "[S]mith, Julie"
echo "[Z]ane, Morris"
echo

read person

case "$person" in
  E|e)
    echo
    echo "Roland Evans"
    echo "4321 Flash Dr."
    echo "Hardscrabble, CO 80753"
    echo "(303) 734-9874"
    echo "revans@zzy.net"
    ;;
  J|j)
    echo
    echo "Mildred Jones"
    echo "249 E. 7th St., Apt. 19"
    echo "New York, NY 10009"
    echo "(212) 533-2814"
    echo "milliej@loisaida.com"
    echo "Birthday: Feb. 11"
    ;;
  S|s)
    echo
    echo "Julie Smith"
    echo "123 Main St."
    echo "(555) 555-5555"
    echo "julie@example.com"
    ;;
  *)
    echo
    echo "Not found in database"
    ;;
esac

exit 0
```

---

## 9.3. Command-Line Parameter Handling with Case

Case is useful for parsing command-line arguments.

### Example: Simple Parameter Handling

```bash
#!/bin/bash
# Handle filename with leading dash

case "$1" in
  "")
    echo "Usage: $0 <filename>"
    exit 1
    ;;
  -*)
    # If filename starts with dash, prepend ./
    FILENAME="./$1"
    ;;
  *)
    FILENAME="$1"
    ;;
esac

echo "Processing: $FILENAME"
```

### Example: Flag Processing

```bash
#!/bin/bash
# Parse multiple command-line flags

DEBUG=0
CONFFILE="/etc/default.conf"

while [ $# -gt 0 ]; do
  case "$1" in
    -d|--debug)
      DEBUG=1
      ;;
    -c|--conf)
      CONFFILE="$2"
      shift  # Consume argument
      if [ ! -f "$CONFFILE" ]; then
        echo "Error: Config file not found"
        exit 1
      fi
      ;;
    -h|--help)
      echo "Usage: $0 [-d|--debug] [-c|--conf FILE]"
      exit 0
      ;;
    *)
      echo "Unknown option: $1"
      exit 1
      ;;
  esac
  shift  # Move to next argument
done

[ $DEBUG -eq 1 ] && echo "Debug mode enabled"
echo "Config file: $CONFFILE"
```

---

## 9.4. Case with Command Substitution

The case variable can be generated dynamically using command substitution.

### Example 11-26: Machine Architecture Detection

```bash
#!/bin/bash
# Detect machine architecture

case $(arch) in
  i386)
    echo "80386-based machine"
    ;;
  i486)
    echo "80486-based machine"
    ;;
  i586)
    echo "Pentium-based machine"
    ;;
  i686)
    echo "Pentium2+-based machine"
    ;;
  x86_64)
    echo "x86_64 (64-bit Intel/AMD)"
    ;;
  *)
    echo "Other architecture: $(uname -m)"
    ;;
esac

exit 0
```

---

## 9.5. String Validation Functions

### Example 11-27: Simple String Matching

```bash
#!/bin/bash
# String matching function using case

match_string() {
  # Return 0 if strings match, 90 otherwise
  
  [ $# -eq 2 ] || return 91  # Require exactly 2 arguments
  
  case "$1" in
    "$2")
      return 0   # Match
      ;;
    *)
      return 90  # No match
      ;;
  esac
}

# Test the function
match_string "apple" "apple"
echo "Match 'apple' to 'apple': $?"    # 0

match_string "apple" "orange"
echo "Match 'apple' to 'orange': $?"   # 90

match_string "test"
echo "Match with 1 arg: $?"             # 91

exit 0
```

### Example 11-28: Character Type Validation

```bash
#!/bin/bash
# Validate string content using case patterns

SUCCESS=0
FAILURE=1

isalpha() {
  # Test if first character is alphabetic
  [ -z "$1" ] && return $FAILURE
  
  case "$1" in
    [a-zA-Z]*)
      return $SUCCESS
      ;;
    *)
      return $FAILURE
      ;;
  esac
}

isalpha_full() {
  # Test if entire string is alphabetic
  [ $# -eq 1 ] || return $FAILURE
  
  case "$1" in
    *[!a-zA-Z]*|"")
      return $FAILURE
      ;;
    *)
      return $SUCCESS
      ;;
  esac
}

isdigit() {
  # Test if entire string is numeric
  [ $# -eq 1 ] || return $FAILURE
  
  case "$1" in
    *[!0-9]*|"")
      return $FAILURE
      ;;
    *)
      return $SUCCESS
      ;;
  esac
}

test_value() {
  local val="$1"
  
  if isalpha "$val"; then
    echo "\"$val\" starts with a letter"
    
    if isalpha_full "$val"; then
      echo "\"$val\" is all letters"
    else
      echo "\"$val\" contains non-letters"
    fi
  else
    echo "\"$val\" does not start with a letter"
  fi
  
  if isdigit "$val"; then
    echo "\"$val\" is all digits"
  else
    echo "\"$val\" contains non-digits"
  fi
  echo
}

# Test various strings
test_value "hello"       # All letters
test_value "hello123"    # Mixed
test_value "123"         # All digits
test_value "123abc"      # Starts with digit
test_value "-test"       # Starts with non-alphanum

exit 0
```

---

## 9.6. The `select` Construct

The `select` construct creates an interactive menu for user selection. It prompts the user to choose from a numbered list.

### Syntax

```bash
select variable [in list]
do
  command...
  break  # Important: break to exit select
done
```

**Key points:**
- Displays a numbered menu of choices
- Stores user selection in `variable`
- Uses `$PS3` prompt (default: `#? `)
- Requires explicit `break` to exit
- If `[in list]` omitted, uses `$@` (command-line arguments)

### Example 11-29: Vegetable Selection Menu

```bash
#!/bin/bash
# Simple select menu

PS3='Choose your favorite vegetable: '

select vegetable in "beans" "carrots" "potatoes" "onions" "rutabagas"
do
  echo
  echo "Your favorite vegetable is: $vegetable"
  echo
  break  # Important: exit select without break creates infinite loop
done

exit 0
```

**Output example:**
```
1) beans
2) carrots
3) potatoes
4) onions
5) rutabagas
Choose your favorite vegetable: 2

Your favorite vegetable is: carrots
```

### Example 11-30: Select in a Function

```bash
#!/bin/bash
# Using select within a function

PS3='Choose a vegetable: '

choose_vegetable() {
  # If [in list] is omitted, select uses function arguments
  select vegetable
  do
    echo
    echo "You chose: $vegetable"
    echo
    break
  done
}

# Pass arguments to function, they become the select list
choose_vegetable beans rice carrots radishes spinach

exit 0
```

### Example: Select with Validation

```bash
#!/bin/bash
# Select with validation and custom handling

PS3='Select an option: '

while true; do
  select option in "Option A" "Option B" "Option C" "Quit"
  do
    case $option in
      "Option A")
        echo "You selected Option A"
        break
        ;;
      "Option B")
        echo "You selected Option B"
        break
        ;;
      "Option C")
        echo "You selected Option C"
        break
        ;;
      "Quit")
        echo "Goodbye!"
        exit 0
        ;;
      *)
        echo "Invalid selection"
        continue 2  # Continue outer loop
        ;;
    esac
  done
  
  read -p "Continue? (y/n) " -n1 response
  echo
  [ "$response" != "y" ] && break
done

exit 0
```

---

## 9.7. Combining Case and Loops

### Example: Loop with Case Input Validation

```bash
#!/bin/bash
# Repeat menu until user chooses to exit

while true; do
  echo
  echo "===== Main Menu ====="
  echo "1. List files"
  echo "2. Create file"
  echo "3. Remove file"
  echo "4. Exit"
  echo
  read -p "Choose an option: " choice
  
  case "$choice" in
    1)
      echo "Listing files:"
      ls -l
      ;;
    2)
      read -p "Filename: " filename
      touch "$filename"
      echo "File created: $filename"
      ;;
    3)
      read -p "Filename to remove: " filename
      rm -f "$filename"
      echo "File removed: $filename"
      ;;
    4)
      echo "Goodbye!"
      exit 0
      ;;
    *)
      echo "Invalid option"
      ;;
  esac
done
```

---

## Exercises

1. **Create a case-based menu** that displays options for file operations (view, copy, delete, rename) and executes the selected operation.

2. **Write a script** that uses case to validate and process command-line options (`-d`, `-f`, `-v`, etc.).

3. **Build a character classifier** using case with character classes that identifies alphabetic, numeric, and special characters from user input.

4. **Implement a select menu** that lets users choose from a list of system commands and executes the chosen command with appropriate arguments.

5. **Create string validation functions** (like `isalpha`, `isdigit`, `isalnum`) using case patterns for negative matching (`[!...]`).

6. **Write an interactive script** that uses a combination of `select` and `case` to build a multi-level menu system.

---

## Summary

- **`case/esac`:** Branching based on pattern matching; cleaner than multiple `if/then/else` statements
- **Patterns:** Support glob wildcards (`*`, `?`, `[...]`), ranges, and POSIX character classes (`[[:alpha:]]`, `[[:digit:]]`)
- **Multiple patterns:** Use `|` to match multiple patterns to same block
- **Default case:** Use `*` for catch-all/default behavior
- **Command substitution:** Case variable can be dynamically generated with `$(command)`
- **Parameter handling:** Useful for parsing command-line arguments and flags
- **String validation:** Case patterns enable functions to check if strings are alphabetic, numeric, etc.
- **`select` construct:** Creates interactive menus with numbered options
- **`$PS3`:** Customizable prompt for select (default: `#? `)
- **Select requires `break`:** Without it, select creates an infinite loop
- **Combine case and loops:** For multi-level menus and repeating operations
