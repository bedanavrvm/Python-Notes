# Chapter 9: Loops and Branches

## Understanding Control Flow in Shell Scripts

Control flow statements direct the execution path through your script, allowing you to make decisions based on conditions and repeat code blocks. This chapter focuses on two critical control structures:

1. **The `case/esac` construct**: A cleaner, more efficient alternative to multiple `if/then/else` statements for matching a value against multiple patterns
2. **The `select` construct**: An interactive menu system that prompts users to choose from numbered options

These constructs are more than syntactic sugar—they represent fundamental patterns for decision-making found in every programming language. Understanding them deeply makes you a better programmer overall.

### Why These Constructs Matter

Consider validating user input against multiple choices:

```bash
# Without case (verbose and error-prone)
if [ "$choice" = "A" ] || [ "$choice" = "a" ]; then
  handle_a
elif [ "$choice" = "B" ] || [ "$choice" = "b" ]; then
  handle_b
elif [ "$choice" = "C" ] || [ "$choice" = "c" ]; then
  handle_c
else
  echo "Invalid"
fi

# With case (clean and efficient)
case "$choice" in
  A|a) handle_a ;;
  B|b) handle_b ;;
  C|c) handle_c ;;
  *)   echo "Invalid" ;;
esac
```

The `case` version is:
- **Faster**: No unnecessary condition evaluations
- **Clearer**: Pattern intent is obvious at a glance
- **Maintainable**: Easy to add or remove patterns
- **Less error-prone**: Can't accidentally omit a comparison operator

---

## 9.1. The `case/esac` Construct

### Understanding Pattern Matching

The `case` construct is Bash's implementation of pattern matching—selecting code to execute based on whether a value matches certain patterns. It's Bash's analog to `switch` statements in C/C++ or pattern matching in functional languages.

The fundamental idea is simple: **Compare a value against multiple patterns and execute code for the first matching pattern.**

### Syntax and Semantics

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

**Key points to understand:**

- **No quoting required**: While `"$variable"` is idiomatic, Bash doesn't require quotes around variables in `case`. There's no word splitting because patterns are treated specially.
- **Pattern matching operators:**
  - `pattern)` ends each pattern specification
  - `;;` (double semicolon) terminates the code block for that pattern
  - Matches proceed in order—the **first matching pattern** wins
  - Subsequent patterns are not evaluated once a match is found
  - `*` matches any value (default case, always matches if reached)
- **`esac`**: Literally "case" backwards; ends the entire construct
- **Exit behavior**: Once a pattern matches and its code executes, the entire `case` statement terminates (unlike C's `switch` which requires `break`)

### Pattern Types Supported

Bash `case` patterns support:

1. **Glob wildcards** (same as filename expansion):
   - `*` matches any sequence of characters
   - `?` matches any single character
   - `[abc]` matches any character in the set
   - `[a-z]` matches any character in the range

2. **Literal strings**: Exact text matching (recommended for clarity when not using wildcards)

3. **POSIX character classes** (portable across different systems):
   - `[[:alpha:]]` — Letters (a-z, A-Z)
   - `[[:digit:]]` — Digits (0-9)
   - `[[:alnum:]]` — Alphanumeric (letters + digits)
   - `[[:space:]]` — Whitespace characters
   - `[[:lower:]]` — Lowercase letters
   - `[[:upper:]]` — Uppercase letters
   - `[[:punct:]]` — Punctuation characters

4. **Negation**: `[!pattern]` matches anything NOT matching the pattern

### Example 11-24 (Enhanced): Character Classification

```bash
#!/bin/bash
# classify-char.sh: Classify input character using case patterns

echo "Hit a key, then press Enter:"
read -n1 keypress
echo

case "$keypress" in
  [[:lower:]])
    echo "Lowercase letter: $keypress"
    ;;
  [[:upper:]])
    echo "Uppercase letter: $keypress"
    ;;
  [0-9])
    echo "Digit: $keypress"
    ;;
  [[:space:]])
    echo "Whitespace character (space, tab, newline)"
    ;;
  [[:punct:]])
    echo "Punctuation: $keypress"
    ;;
  *)
    echo "Other character (control code or extended ASCII)"
    ;;
esac

exit 0
```

**Character class comparison:**

```bash
# POSIX classes (portable, recommended)
case "$char" in
  [[:lower:]])  echo "Lowercase" ;;
  [[:digit:]])  echo "Digit" ;;
esac

# Traditional ranges (simpler but less portable)
case "$char" in
  [a-z])  echo "Lowercase" ;;
  [0-9])  echo "Digit" ;;
esac

# Both work in Bash, but POSIX classes work in more shells
```

### Why Pattern Order Matters

In `case` statements, pattern evaluation stops at the **first match**. Therefore, pattern order can affect behavior when patterns overlap:

```bash
# This works: more specific patterns first
case "$file" in
  *.tar.gz)  echo "Compressed tar" ;;     # Matches first
  *.tar)     echo "Tar archive" ;;        # Won't reach if .tar.gz
  *.gz)      echo "Gzip file" ;;
  *)         echo "Other" ;;
esac

# This breaks: overly broad pattern stops evaluation
case "$file" in
  *.*)       echo "File with extension" ;;  # Matches EVERYTHING with dot
  *.tar.gz)  echo "Compressed tar" ;;      # Never reached!
  *.tar)     echo "Tar archive" ;;         # Never reached!
esac
```

---

## 9.2. Case with Multiple Patterns and Advanced Matching

### Multiple Patterns for Same Code Block

The pipe `|` operator allows multiple patterns to execute the same code block. This is essential for handling case-insensitive input or variations of the same choice.

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

**Why this matters**: In many languages, you'd need multiple statements or complex conditions. Here, you can elegantly group related patterns.

### Glob Patterns for File Matching

Since `case` supports glob wildcards, it's perfect for file type matching:

```bash
case "$filename" in
  *.txt)
    echo "Text file"
    ;;
  *.sh)
    echo "Shell script"
    ;;
  *.py)
    echo "Python script"
    ;;
  *.tar.gz)
    echo "Compressed archive"
    ;;
  *.*)
    echo "File with extension (type unknown)"
    ;;
  *)
    echo "File without extension"
    ;;
esac
```

**Important**: More specific patterns (like `*.tar.gz`) must come before less specific ones (like `*.*`).

### Example 11-25 (Enhanced): Contact List Menu

```bash
#!/bin/bash
# contact-list.sh: Interactive contact lookup using case

# Database of contacts (in production, use a file or database)
declare -A contacts=(
  [E]="Roland Evans|4321 Flash Dr.|Hardscrabble, CO 80753|(303) 734-9874|revans@zzy.net"
  [J]="Mildred Jones|249 E. 7th St., Apt. 19|New York, NY 10009|(212) 533-2814|milliej@loisaida.com|Birthday: Feb. 11"
  [S]="Julie Smith|123 Main St.|Boulder, CO|(555) 555-5555|julie@example.com"
  [Z]="Morris Zane|999 Mountain View|Denver, CO 80202|(720) 555-1234|mzane@example.com"
)

clear
echo "========== Contact List =========="
echo
echo "Choose a person:"
echo "[E]vans, Roland"
echo "[J]ones, Mildred"
echo "[S]mith, Julie"
echo "[Z]ane, Morris"
echo

read -p "Enter choice: " person

case "$person" in
  [Ee])
    echo
    echo "Contact: Roland Evans"
    echo "Address: 4321 Flash Dr., Hardscrabble, CO 80753"
    echo "Phone: (303) 734-9874"
    echo "Email: revans@zzy.net"
    ;;
  [Jj])
    echo
    echo "Contact: Mildred Jones"
    echo "Address: 249 E. 7th St., Apt. 19, New York, NY 10009"
    echo "Phone: (212) 533-2814"
    echo "Email: milliej@loisaida.com"
    echo "Birthday: Feb. 11"
    ;;
  [Ss])
    echo
    echo "Contact: Julie Smith"
    echo "Address: 123 Main St., Boulder, CO"
    echo "Phone: (555) 555-5555"
    echo "Email: julie@example.com"
    ;;
  [Zz])
    echo
    echo "Contact: Morris Zane"
    echo "Address: 999 Mountain View, Denver, CO 80202"
    echo "Phone: (720) 555-1234"
    echo "Email: mzane@example.com"
    ;;
  *)
    echo
    echo "✗ Not found in database"
    ;;
esac

exit 0
```

**Enhancement notes:**
- Using `[Ee]` (character class) instead of `E|e` is more concise
- More structured data output
- Could be extended to support full names (case-insensitive)

---

## 9.3. Command-Line Parameter Handling with Case

### Parameter Validation Pattern

One of the most practical uses of `case` is validating and routing command-line arguments:

```bash
#!/bin/bash
# Handle filename with leading dash (ambiguous with options)

case "$1" in
  "")
    echo "Usage: $0 <filename>"
    exit 1
    ;;
  -*)
    # If filename starts with dash, prepend ./ to clarify it's a filename
    FILENAME="./$1"
    ;;
  *)
    FILENAME="$1"
    ;;
esac

echo "Processing: $FILENAME"
```

**Why this matters**: A filename like `-config` would be interpreted as an option (flag). By detecting it, you can disambiguate.

### Complex Flag Processing with Case

```bash
#!/bin/bash
# parse-options.sh: Sophisticated argument parsing

DEBUG=0
VERBOSE=0
CONFFILE="/etc/default.conf"
OUTPUT_FILE=""

# Process arguments
while [ $# -gt 0 ]; do
  case "$1" in
    -d|--debug)
      DEBUG=1
      echo "Debug mode enabled"
      ;;
    -v|--verbose)
      VERBOSE=1
      echo "Verbose output enabled"
      ;;
    -c|--conf)
      CONFFILE="$2"
      shift  # Consume the argument
      if [ ! -f "$CONFFILE" ]; then
        echo "Error: Config file '$CONFFILE' not found"
        exit 1
      fi
      echo "Using config: $CONFFILE"
      ;;
    -o|--output)
      OUTPUT_FILE="$2"
      shift  # Consume the argument
      ;;
    -h|--help)
      echo "Usage: $0 [-d|--debug] [-v|--verbose] [-c|--conf FILE] [-o|--output FILE]"
      exit 0
      ;;
    -*)
      echo "Error: Unknown option: $1"
      exit 1
      ;;
    *)
      # Non-option argument (positional)
      echo "Processing file: $1"
      ;;
  esac
  shift  # Move to next argument
done

echo "Configuration:"
echo "  Debug: $DEBUG"
echo "  Verbose: $VERBOSE"
echo "  Config file: $CONFFILE"
echo "  Output file: ${OUTPUT_FILE:-none}"
```

**Key pattern:**
1. `while [ $# -gt 0 ]` processes all arguments
2. `shift` consumes the current argument
3. For options that take values (`-c FILE`), use `shift` twice or use `"$2"`

---

## 9.4. Case with Command Substitution

### Dynamic Pattern Matching

The case variable doesn't need to be a simple variable—it can be the result of any command substitution. This enables dynamic, context-dependent branching.

```bash
#!/bin/bash
# Detect machine architecture using $(arch) command

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
    echo "x86_64 (64-bit Intel/AMD processor)"
    ;;
  aarch64)
    echo "ARM 64-bit processor"
    ;;
  *)
    echo "Other architecture: $(uname -m)"
    ;;
esac
```

### Practical: Environment Detection

```bash
#!/bin/bash
# Detect and configure based on operating system

case $(uname -s) in
  Linux)
    echo "Running on Linux"
    INSTALL_CMD="apt-get install"
    PACKAGE_MANAGER="apt"
    ;;
  Darwin)
    echo "Running on macOS"
    INSTALL_CMD="brew install"
    PACKAGE_MANAGER="homebrew"
    ;;
  MINGW*|MSYS*)
    echo "Running on Windows/Git Bash"
    INSTALL_CMD="choco install"
    PACKAGE_MANAGER="chocolatey"
    ;;
  *)
    echo "Unknown OS: $(uname -s)"
    exit 1
    ;;
esac

echo "Using package manager: $PACKAGE_MANAGER"
```

---

## 9.5. String Validation Functions Using Case

### The Power of Case for Type Checking

Case patterns excel at string validation because they can match complex patterns without external commands:

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
```

### Character Type Validation Functions

```bash
#!/bin/bash
# String validation functions using case patterns

SUCCESS=0
FAILURE=1

# Check if string STARTS with letter
isalpha() {
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

# Check if ENTIRE string is letters (no digits, spaces, etc.)
isalpha_full() {
  [ $# -eq 1 ] || return $FAILURE
  
  # Match fails if string contains non-letters or is empty
  case "$1" in
    *[!a-zA-Z]*|"")
      return $FAILURE
      ;;
    *)
      return $SUCCESS
      ;;
  esac
}

# Check if ENTIRE string is digits
isdigit() {
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

# Check if ENTIRE string is alphanumeric
isalnum() {
  [ $# -eq 1 ] || return $FAILURE
  
  case "$1" in
    *[!a-zA-Z0-9]*|"")
      return $FAILURE
      ;;
    *)
      return $SUCCESS
      ;;
  esac
}

# Test and report
test_value() {
  local val="$1"
  
  echo "Testing: "$val""
  
  if isalpha "$val"; then
    echo "  ✓ Starts with a letter"
    
    if isalpha_full "$val"; then
      echo "  ✓ Contains ONLY letters"
    else
      echo "  ✗ Contains non-letter characters"
    fi
  else
    echo "  ✗ Does not start with a letter"
  fi
  
  if isdigit "$val"; then
    echo "  ✓ Contains ONLY digits"
  fi
  
  if isalnum "$val"; then
    echo "  ✓ Is alphanumeric (no special chars)"
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

**Key insight**: The pattern `*[!a-zA-Z]*` matches any string containing at least one non-letter character. This enables powerful type checking without external commands.

---

## 9.6. The `select` Construct: Interactive Menus

### Understanding Menu-Driven Programs

The `select` construct is Bash's built-in tool for creating interactive menus. It automatically:
- Displays numbered options
- Prompts for user input
- Stores the selected item in a variable
- Handles invalid input gracefully

### Syntax and Semantics

```bash
select variable [in list]
do
  # Code executes for each selection
  # Must explicitly break to exit
done
```

**Key points:**

- **`variable`**: Stores the user's selection (the actual item, not the number)
- **`[in list]`**: Space-separated items to choose from (optional—defaults to `$@`)
- **Automatic numbering**: Menu displays 1), 2), 3), etc.
- **`$PS3` prompt**: Customizable menu prompt (default: `#? `)
- **`break` is mandatory**: Without `break`, select loops forever
- **Invalid input handling**: Pressing Enter or invalid choice re-displays menu
- **`$REPLY`**: Contains the user's numeric input (available after selection)

### Example 11-29 (Enhanced): Vegetable Selection Menu

```bash
#!/bin/bash
# vegetable-menu.sh: Simple select menu demonstration

# Customize the prompt
PS3='Choose your favorite vegetable: '

select vegetable in "beans" "carrots" "potatoes" "onions" "rutabagas" "quit"
do
  case "$vegetable" in
    quit)
      echo "Goodbye!"
      break
      ;;
    *)
      echo
      echo "✓ Your favorite vegetable is: $vegetable"
      echo "  (You selected option #$REPLY)"
      echo
      break  # Exit after one selection
      ;;
  esac
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
6) quit
Choose your favorite vegetable: 2

✓ Your favorite vegetable is: carrots
  (You selected option #2)
```

### Example 11-30 (Enhanced): Select in a Function

```bash
#!/bin/bash
# select-function.sh: Using select within a function

choose_vegetable() {
  # If [in list] is omitted, select uses function arguments ($@)
  local PS3='Choose a vegetable: '
  
  select vegetable
  do
    echo
    echo "You chose: $vegetable"
    echo
    break
  done
}

# Pass arguments to function; they become the select list
choose_vegetable beans rice carrots radishes spinach

exit 0
```

### Advanced: Select with Validation and Multiple Selections

```bash
#!/bin/bash
# select-validation.sh: Select with error handling and multiple choices

PS3='Select an option: '

while true; do
  select option in "List Files" "Create File" "Edit Config" "Quit"
  do
    case "$option" in
      "List Files")
        echo "Files in current directory:"
        ls -lh
        break
        ;;
      "Create File")
        read -p "Enter filename: " filename
        touch "$filename"
        echo "✓ Created: $filename"
        break
        ;;
      "Edit Config")
        read -p "Enter config file path: " config
        if [ -f "$config" ]; then
          ${EDITOR:-nano} "$config"
          echo "✓ Config updated"
        else
          echo "✗ File not found: $config"
        fi
        break
        ;;
      "Quit")
        echo "Goodbye!"
        exit 0
        ;;
      *)
        echo "✗ Invalid selection (expected 1-4)"
        continue 2  # Continue outer while loop
        ;;
    esac
  done
  
  # Ask to continue after each operation
  read -p "Continue? (y/n) " -n1 response
  echo
  [ "$response" != "y" ] && break
done

exit 0
```

**Advanced control flow:**
- `break` exits inner `select`
- `continue 2` skips to next iteration of outer `while` loop
- Menu re-displays on invalid input

---

## 9.7. Combining Case and Loops

### Multi-Level Menu Systems

In real applications, you often need menus within menus. Combine `case` with loops for sophisticated navigation:

```bash
#!/bin/bash
# menu-system.sh: Multi-level menu with case and while loop

show_main_menu() {
  echo
  echo "===== Main Menu ====="
  echo "1. File Operations"
  echo "2. System Information"
  echo "3. User Management"
  echo "4. Exit"
  echo
}

show_file_menu() {
  echo "  --- File Operations ---"
  echo "  1. List files"
  echo "  2. Create file"
  echo "  3. Remove file"
  echo "  4. Back to main menu"
  echo
}

file_operations() {
  while true; do
    show_file_menu
    read -p "Choose option: " choice
    
    case "$choice" in
      1)
        echo "Listing current directory:"
        ls -lh | head -10
        ;;
      2)
        read -p "Filename: " filename
        if touch "$filename"; then
          echo "✓ File created: $filename"
        fi
        ;;
      3)
        read -p "Filename to remove: " filename
        if rm -f "$filename"; then
          echo "✓ File removed: $filename"
        fi
        ;;
      4)
        return  # Return to main menu
        ;;
      *)
        echo "✗ Invalid option"
        ;;
    esac
  done
}

system_info() {
  echo
  echo "=== System Information ==="
  echo "Hostname: $(hostname)"
  echo "Kernel: $(uname -s)"
  echo "Processor: $(uname -m)"
  echo "Uptime: $(uptime | awk -F'up' '{print $2}')"
  echo
}

# Main loop
while true; do
  show_main_menu
  read -p "Choose an option: " choice
  
  case "$choice" in
    1)
      file_operations
      ;;
    2)
      system_info
      ;;
    3)
      echo "User management: not yet implemented"
      ;;
    4)
      echo "Goodbye!"
      exit 0
      ;;
    *)
      echo "✗ Invalid option"
      ;;
  esac
done
```

### Pattern: Menu with Validation Loop

```bash
#!/bin/bash
# Repeat menu until valid choice made

while true; do
  echo "Choose action: [S]tart, [S]top, [R]estart, [Q]uit"
  read -p "> " choice
  
  case "$choice" in
    [Ss]*)
      echo "Starting service..."
      break
      ;;
    [Ss]top)
      echo "Stopping service..."
      break
      ;;
    [Rr]*)
      echo "Restarting service..."
      break
      ;;
    [Qq]*)
      echo "Quit"
      exit 0
      ;;
    *)
      echo "Invalid choice. Try again."
      ;;
  esac
done
```

---

## Best Practices and Common Patterns

### 1. Always Quote Variables in Case Statements

```bash
# Good: Proper quoting (defensive)
case "$variable" in
  pattern) ... ;;
esac

# Works but less safe: No quotes
case $variable in
  pattern) ... ;;
esac

# Why: If variable contains spaces or special chars, quoting prevents them from being interpreted
var="hello world"
case $var in           # Bad: treats as "hello" only
  "hello world") echo "match" ;;
  *) echo "no match" ;;
esac

case "$var" in         # Good: matches complete value
  "hello world") echo "match" ;;
  *) echo "no match" ;;
esac
```

### 2. Order Patterns from Specific to General

```bash
# Good: Specific patterns first
case "$file" in
  *.tar.gz)  echo "Compressed tar" ;;
  *.tar)     echo "Tar" ;;
  *.gz)      echo "Gzip" ;;
  *.*)       echo "Other" ;;
  *)         echo "No extension" ;;
esac

# Bad: General pattern blocks specific ones
case "$file" in
  *.*)       echo "Any file" ;;       # Matches ALL files with dot
  *.tar.gz)  echo "Never reaches" ;;  # Unreachable!
esac
```

### 3. Use Character Classes for Readability

```bash
# Good: POSIX character classes (portable)
case "$char" in
  [[:digit:]])  echo "Digit" ;;
  [[:alpha:]])  echo "Letter" ;;
  [[:space:]])  echo "Space" ;;
esac

# Traditional but less portable
case "$char" in
  [0-9])   echo "Digit" ;;
  [a-zA-Z]) echo "Letter" ;;
esac
```

### 4. Select Requires Explicit Break

```bash
# Correct: Break after selection
select item in apple banana cherry
do
  echo "You selected: $item"
  break  # MUST break to exit select
done

# Wrong: Infinite loop without break
select item in apple banana cherry
do
  echo "You selected: $item"
  # No break = loop forever
done
```

### 5. Use Case for Command Routing

```bash
# Good: Use case to route to functions
case "$action" in
  list)   list_items ;;
  add)    add_item "$@" ;;
  delete) delete_item "$@" ;;
  help)   show_help ;;
  *)      echo "Unknown action" ;;
esac

# Avoids deeply nested if-then-else
```

---

## 10 Core Programming Concepts from Control Flow

### 1. **Pattern Matching**
`case/esac` implements pattern matching—selecting code based on whether a value matches patterns. This is fundamental to many modern languages and functional programming.

```bash
case "$status" in
  success|ok|done)  handle_success ;;
  error|fail)       handle_error ;;
esac
```

### 2. **Dispatch Tables and Routing**
Using `case` to route to different handlers based on commands or options is a fundamental dispatch pattern used everywhere—from interpreters to servers.

```bash
case "$command" in
  list)   list_handler ;;
  add)    add_handler ;;
  delete) delete_handler ;;
esac
```

### 3. **Guard Clauses and Early Exit**
Using `case` with `return` or `exit` for input validation is a "guard clause" pattern—checking prerequisites before executing main logic.

```bash
case "$input" in
  "") echo "Error: empty input"; return 1 ;;
  *) process "$input" ;;
esac
```

### 4. **First-Match Semantics**
`case` evaluates patterns in order and executes code for the **first match**. This creates a priority system where pattern order matters (more specific patterns first).

```bash
case "$file" in
  *.tar.gz)   # Most specific
  *.gz)       # More general
  *.*         # Least specific
esac
```

### 5. **Multi-Option Input Handling**
The pipe operator (`|`) allows multiple input options for the same code path, implementing the "accept variations" pattern.

```bash
case "$response" in
  yes|y|Y|YES)   # Accept any variation
    perform_action ;;
esac
```

### 6. **Interactive Menu Systems**
The `select` construct implements a fundamental UI pattern—numbered menu with automatic prompting and validation.

```bash
select option in "File" "Edit" "Help" "Quit"
do
  case "$option" in
    "File")   file_menu ;;
    "Edit")   edit_menu ;;
    "Help")   show_help ;;
    "Quit")   exit ;;
  esac
  break
done
```

### 7. **REPL (Read-Eval-Print Loop)**
Combining `select` with loops creates a REPL pattern—display menu, read input, execute action, repeat.

```bash
while true; do
  select cmd in "Start" "Stop" "Quit"
  do
    case "$cmd" in
      "Start") start_service; break ;;
      "Stop")  stop_service; break ;;
      "Quit")  exit 0 ;;
    esac
  done
done
```

### 8. **Glob Pattern Matching for File Operations**
Using case with glob patterns (`*.ext`, `[a-z].*`, etc.) applies text pattern matching to file operations.

```bash
case "$filename" in
  *.tar.gz) decompress_targz ;;
  *.zip)    unzip ;;
  *)        echo "Unknown format" ;;
esac
```

### 9. **Default Case Handling**
The `*` pattern implements the "default behavior" pattern—what to do when no specific pattern matches.

```bash
case "$input" in
  valid)      handle_valid ;;
  expected)   handle_expected ;;
  *)          handle_unexpected ;;  # Default for anything else
esac
```

### 10. **Nested Control Structures**
Combining multiple `case` statements, loops, and conditionals creates complex decision trees and multi-level menu systems.

```bash
while true; do
  select menu in "Main" "Settings" "Quit"
  do
    case "$menu" in
      "Main")
        select action in "List" "Add" "Delete" "Back"
        do
          case "$action" in
            "List") list_items; break ;;
            "Add")  add_item; break ;;
            "Delete") delete_item; break ;;
            "Back") break 2 ;;  # Break two loops
          esac
        done
        ;;
      "Settings") configure_settings ;;
      "Quit") exit 0 ;;
    esac
  done
done
```

---

## Exercises

1. **Create a case-based menu** that displays options for file operations (view, copy, delete, rename, compress) and executes the selected operation.

2. **Write a script** that uses case to validate and process command-line options (`-d`, `-f`, `-v`, etc.) and set configuration flags accordingly.

3. **Build a character classifier** using case with character classes that identifies alphabetic, numeric, punctuation, and whitespace characters from user input.

4. **Implement a select menu** that lets users choose from a list of system commands (ls, pwd, whoami, date, etc.) and executes the chosen command.

5. **Create string validation functions** (like `isalpha`, `isdigit`, `isalnum`, `isphone`) using case patterns for negative matching (`[!...]`).

6. **Write an interactive script** that uses a combination of `select` and `case` to build a three-level menu system (Main → Submenu → Actions).

7. **Implement error code routing** using case to map exit codes to user-friendly error messages and appropriate recovery actions.

8. **Create a log parser** that uses case patterns to classify log lines by severity level (ERROR, WARN, INFO, DEBUG) and take appropriate actions.

---

## Summary

**Case/esac construct:**
- **Pattern matching**: Select code based on whether a value matches patterns
- **Cleaner than if/then/else**: More readable, efficient, and maintainable for multiple alternatives
- **Pattern support**: Glob wildcards (`*`, `?`, `[...]`), ranges, POSIX character classes (`[[:alpha:]]`, `[[:digit:]]`)
- **Multiple patterns**: Use `|` to match multiple patterns to same code block
- **Default case**: Use `*` for catch-all/default behavior
- **Command substitution**: Case variable can be dynamically generated with `$(command)`

**Select construct:**
- **Interactive menus**: Create numbered options with automatic prompting
- **Automatic validation**: Re-displays menu on invalid input
- **`$PS3` customization**: Change the menu prompt
- **Mandatory break**: Must explicitly `break` to exit, otherwise infinite loop
- **`$REPLY` variable**: Contains the user's numeric input (1, 2, 3, etc.)
- **Function integration**: Works within functions, using arguments as menu options

**Best practices:**
- Always quote variables in case statements for safety
- Order patterns from specific to general (more specific first)
- Use POSIX character classes for portability
- Combine case and loops for multi-level menus
- Use case to implement dispatch tables and routing
- Remember: `case` returns after first match (no fall-through like C's switch)

**Key insight**: Both `case` and `select` are implementations of fundamental decision-making patterns found in every language. Mastering them makes you understand control flow at a deeper level and helps you recognize these patterns in other programming contexts.
