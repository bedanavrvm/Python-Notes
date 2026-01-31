# Chapter 8: Parameter Expansion and Substitution

## Understanding Parameter Expansion

Parameter expansion is one of Bash's most powerful features, allowing you to transform, validate, and manipulate variables in sophisticated ways. Unlike simple variable substitution (`$var`), parameter expansion uses the `${...}` syntax to enable advanced operations that are fundamental to writing robust, production-quality shell scripts.

The key insight is that parameter expansion allows you to:
- **Provide intelligent defaults** when variables are missing or empty
- **Extract and transform text** without spawning external processes
- **Test variable existence** with clear error messages
- **Perform pattern matching** to parse files, paths, and strings
- **Manage indirect references** to dynamically access other variables

Understanding parameter expansion deeply separates novice shell scripts from professional ones. It's the difference between writing scripts that fail silently with cryptic bugs and scripts that handle edge cases gracefully.

### Why Parameter Expansion Matters

Consider a common scenario: Your script needs a database password, but you want it to come from an environment variable with a hardcoded fallback. Without parameter expansion, you'd need an if-statement:

```bash
# Without parameter expansion (verbose)
if [ -z "$DB_PASSWORD" ]; then
  DB_PASSWORD="default_password"
fi
```

With parameter expansion, this becomes a single, clear line:

```bash
# With parameter expansion (clean)
DB_PASSWORD="${DB_PASSWORD:-default_password}"
```

This is more than cosmetic—parameter expansion operators are often **more efficient** because they don't spawn subshells and work entirely within Bash's built-in mechanisms.

### Core Concepts: Unset vs. Null

Before diving into operators, understand the distinction Bash makes:

- **Unset**: The variable doesn't exist at all. `$var` expands to nothing; `[[ -z $var ]]` is true.
- **Null**: The variable exists but contains an empty string. Example: `var=""`; `$var` expands to an empty string; `[[ -z $var ]]` is still true, but `[[ -v var ]]` is true (variable exists).

Some parameter expansion operators treat unset and null the same way (noted with `:` prefix), while others distinguish between them (without `:`). This subtle distinction allows for precise control over variable handling.

---

## 8.1. Default Values and Parameter Testing

### Understanding Default Value Operators

The four operators in this section provide increasingly powerful ways to handle missing or empty variables. Each serves a specific purpose in defensive programming:

- **`${var:-default}`**: Use default if absent—the most common operator
- **`${var:=default}`**: Use default AND permanently assign it
- **`${var:+alternate}`**: Use alternate value only if variable exists
- **`${var?error}`**: Fail hard with an error message if missing

These operators follow a pattern: the `:` character includes null/empty checks, while operators without `:` only check for unset variables.

### The `${parameter:-default}` Operator: Coalesce with Default

This is the "provide a fallback" operator. If the parameter is unset or null, the `default` is used. Otherwise, the parameter's value is used.

**Use cases:**
- Function parameter defaults
- Environment variable fallbacks
- Configuration initialization
- Optional argument handling

```bash
# Variable not set - fallback is used
echo "${undefined:-fallback}"    # fallback

# Variable set to empty string - fallback is used (because of :)
var=""
echo "${var:-fallback}"          # fallback

# Variable set to a value - variable is used
var="value"
echo "${var:-fallback}"          # value

# Practical: function parameter defaults
greet() {
  local name="${1:-Guest}"
  local greeting="${2:-Hello}"
  echo "$greeting, $name!"
}

greet                        # Hello, Guest!
greet "Alice"                # Hello, Alice!
greet "Bob" "Hi"             # Hi, Bob!

# Practical: environment configuration
TIMEOUT="${TIMEOUT:-30}"
LOG_LEVEL="${LOG_LEVEL:-INFO}"
MAX_RETRIES="${MAX_RETRIES:-3}"
echo "Timeout: $TIMEOUT, Log level: $LOG_LEVEL, Retries: $MAX_RETRIES"
```

**Key difference without the colon:**

```bash
# ${var-default} (no colon) - only checks for unset, not null
var=""
echo "${var:-fallback}"   # fallback (treats empty as null)
echo "${var-fallback}"    # (empty string - var IS set, just empty)

# This distinction matters in edge cases:
# Use :- when you want to treat empty strings as "missing"
# Use - only when you want to allow empty strings as valid values
```

### The `${parameter:=default}` Operator: Initialize with Default

This operator combines checking (like `:-`) with assignment. If the parameter is unset or null, it's **set** to the default AND that value is used. This permanently modifies the variable.

**Use cases:**
- One-time initialization
- Lazy initialization patterns
- Persistent counter/state setup
- Default configuration assignment

```bash
echo "${myvar:=newvalue}"   # newvalue
echo "$myvar"               # newvalue (variable WAS set)

# Verify assignment:
unset myvar
echo "${myvar:=initialvalue}"
echo "$myvar"               # initialvalue (var now exists)

# Practical: Initialize a counter
: ${count:=0}
((count++))
echo $count                 # 1

((count++))
echo $count                 # 2 (variable persists because it was assigned)

# Practical: Configuration registry
DEFAULT_PORT=8080
: ${SERVER_PORT:=$DEFAULT_PORT}
echo "Server port: $SERVER_PORT"    # Server port: 8080

# Run again in same session
echo "Server port: $SERVER_PORT"    # Server port: 8080 (unchanged)
```

**Important note:** The `:=` operator **modifies the variable**. This is useful for one-time initialization but be aware of its side effects.

### The `${parameter:+alt_value}` Operator: Use Alternate if Exists

This is the opposite of `:-`. Use the alternate value **only if** the parameter is set and non-null. If unset or null, expand to nothing.

**Use cases:**
- Conditional appending or modification
- Building paths based on conditions
- Activating features only when configured
- Flag evaluation

```bash
var="something"
echo "${var:+replacement}"  # replacement (var is set)

var=""
echo "${var:+replacement}"  # (empty - var exists but is null)

unset var
echo "${var:+replacement}"  # (empty - var doesn't exist)

# Practical: Conditional path building
file="${1:+/path/to/$1}"    # Only append path if $1 is provided
[ -n "$file" ] && echo "File: $file"

# Practical: Feature flags
ENABLE_DEBUG="${DEBUG:+--debug}"
command_with_flag $ENABLE_DEBUG

# Practical: Building command options
OPTIONS=""
[ "$VERBOSE" ] && OPTIONS="${OPTIONS:+$OPTIONS }--verbose"
[ "$QUIET" ] && OPTIONS="${OPTIONS:+$OPTIONS }--quiet"
echo "Options: $OPTIONS"
```

**Use case: Conditional environment setup**

```bash
# Only set up proxy if it's configured
[ -n "$HTTP_PROXY" ] && CURL_ARGS="${CURL_ARGS:+$CURL_ARGS }-x $HTTP_PROXY"
```

### The `${parameter?error}` Operator: Validate and Exit

This operator enforces that a variable **must** be set and non-null. If it isn't, the script prints the error message and exits with status 1. This is a "fail-fast" validation mechanism.

**Use cases:**
- Validate required script arguments
- Ensure critical environment variables are set
- Make implicit dependencies explicit
- Guard against accidental variable typos

```bash
# Exit if critical variable not set
: ${HOME?}                  # Exit if HOME not set
: ${USER?}                  # Exit if USER not set

# With custom error message
username=${1?"Missing required argument: username"}
# Script stops here if $1 is not provided

# Validate multiple required variables
: ${HOSTNAME?} ${USER?} ${HOME?} ${MAIL?}
echo "All required variables are set."

# Practical: Ensure database connection
: ${DB_HOST?"Database host not configured"}
: ${DB_USER?"Database user not configured"}
: ${DB_PASS?"Database password not configured"}

db_connect() {
  # These variables MUST exist for the function to work
  : ${DB_HOST?} ${DB_USER?} ${DB_PASS?}
  echo "Connecting to $DB_HOST as $DB_USER..."
}
```

**Error message format:**
When a variable fails the `?` test, Bash prints: `bash: parameter: error message` and exits.

```bash
#!/bin/bash
: ${REQUIRED_VAR?"This variable must be set"}
# Output: bash: REQUIRED_VAR: This variable must be set
# Exit code: 1
```

### Combining Default Operators: Multi-Level Fallback

You can nest these operators to create sophisticated fallback chains:

```bash
# Try environment first, then function argument, then hardcoded default
APP_NAME="${APP_NAME:-${1:-MyApp}}"

# Configuration loading pattern:
: ${CONFIG_FILE:=$HOME/.myapp.conf}    # Set if not already set
[ -f "$CONFIG_FILE" ] || CONFIG_FILE="/etc/myapp/default.conf"
: ${CONFIG_FILE?}   # Fail if still not found

echo "Using config: $CONFIG_FILE"
```

---

## Example 8-7 (Enhanced): Parameter Testing with Error Messages

```bash
#!/bin/bash
# check-vars.sh: Validate essential environmental variables

echo "=== System Variable Check ==="
echo

# Require critical environment variables - fail if missing
: ${HOSTNAME?} ${USER?} ${HOME?} ${MAIL?}

echo "✓ All critical variables are set"
echo "  Hostname: $HOSTNAME"
echo "  User: $USER"
echo "  Home: $HOME"
echo "  Mail: $MAIL"
echo

# Script-specific variables with defaults
script_var="${1:-important_value}"
echo "✓ Script variable is set: $script_var"
echo

# Demonstrate error condition (uncomment to test)
# This will cause the script to exit with error if ZZXy23AB is not defined
: ${ZZXy23AB?"Variable ZZXy23AB has not been set."}

echo "This line will not print because ZZXy23AB is undefined."
exit 0
```

**Expected behavior:**
- When all variables are set: Prints success messages
- When any critical variable is missing: Exits immediately with error message
- The `?` operator makes implicit dependencies explicit

---

## Example 8-8 (Enhanced): Usage Messages and Argument Validation

```bash
#!/bin/bash
# usage-message.sh: Require command-line arguments with helpful error

# Make the requirement explicit - fail if no argument provided
: ${1?"Usage: $0 ARGUMENT"}

echo "You provided an argument: $1"
echo "Argument length: ${#1}"
echo "First 3 characters: ${1:0:3}"
exit 0
```

**Usage:**
```bash
$ ./usage-message.sh
bash: 1: Usage: usage-message.sh ARGUMENT

$ ./usage-message.sh "hello"
You provided an argument: hello
Argument length: 5
First 3 characters: hel
```

This pattern makes your requirements explicit in the code, eliminating silent failures.

---

## 8.2. String Length and Substring Removal

### Understanding String Length

The `${#var}` syntax returns the number of characters in a variable's value. This is useful for:
- Validating input length
- Parsing fixed-format strings
- Building formatted output
- Array size checking

```bash
# Single string length
var="hello"
echo "${#var}"          # 5

var="hello world"
echo "${#var}"          # 11 (includes the space)

# Array - first element length
array=("one" "two" "three")
echo "${#array}"        # 3 (length of first element, "one")

# Array - number of elements
echo "${#array[@]}"     # 3 (number of elements in array)
echo "${#array[*]}"     # 3 (alternative syntax, same result)

# Function arguments
echo "${#@}"            # Number of arguments passed to script
echo "${#*}"            # Alternative syntax, same result

# Positional parameters
function my_func() {
  echo "Number of arguments: ${#@}"
}

my_func arg1 arg2 arg3  # Number of arguments: 3
```

**Practical example: Validate password strength**

```bash
validate_password() {
  local pass="$1"
  
  if (( ${#pass} < 8 )); then
    echo "Error: Password must be at least 8 characters"
    return 1
  fi
  
  echo "✓ Password meets length requirement"
  return 0
}

validate_password "short"          # Error: ...
validate_password "longenough123"  # ✓ Password meets...
```

### The `#` and `##` Operators: Remove from Front

These operators remove matching patterns from the **beginning** (front) of a string. The distinction is between greedy (longest match) and non-greedy (shortest match).

| Operator | Behavior | Mnemonic |
|----------|----------|----------|
| `${var#pattern}` | Remove **shortest** match from front | `#` (one symbol = minimal) |
| `${var##pattern}` | Remove **longest** match from front | `##` (two symbols = maximal) |

**Key principle:** These operators use glob patterns (same as wildcards):
- `*` matches any sequence of characters
- `?` matches a single character
- `[...]` matches character class

```bash
var="abcd12345abc6789"
pattern="a*c"

# Remove SHORTEST match of pattern from front
echo "${var#$pattern}"   # 12345abc6789
# Removed: "abc" (shortest match of a*c at the start)

# Remove LONGEST match of pattern from front
echo "${var##$pattern}"  # 6789
# Removed: "abcd12345abc" (longest match of a*c at the start)

# Practical: File extension handling
filename="archive.tar.gz"
echo "${filename#*.}"    # tar.gz (remove up to first dot)
echo "${filename##*.}"   # gz (remove up to last dot)

# The difference illustrated:
# #*.  matches ".tar" (shortest .*)
# ##*. matches ".tar.gz" (longest .*)

# Practical: Extract command from path
path="/usr/local/bin/python3"
basename="${path##*/}"   # python3 (remove everything up to last slash)
dirname="${path%/*}"     # /usr/local/bin (remove last slash onwards - see % operator)

# Practical: Remove URL protocol
url="https://example.com/path"
domain="${url##*://}"    # example.com/path (remove protocol)
```

**Understanding the greedy difference:**

```bash
# The pattern is evaluated from the START, matching as much as possible
var="prefix-middle-suffix"

echo "${var#*-}"         # middle-suffix (removes "prefix-" - shortest -*-) 
echo "${var##*-}"        # suffix (removes "prefix-middle-" - longest -*-)

# Why? Because * in the pattern "matches any characters"
# # uses the shortest match: a-
# ## uses the longest match: prefix-middle-
```

### The `%` and `%%` Operators: Remove from Back

These operators remove matching patterns from the **end** (back) of a string. Like `#` and `##`, the distinction is between shortest and longest matches.

| Operator | Behavior | Mnemonic |
|----------|----------|----------|
| `${var%pattern}` | Remove **shortest** match from back | `%` (trailing symbol) |
| `${var%%pattern}` | Remove **longest** match from back | `%%` (two trailing symbols) |

```bash
var="abcd12345abc6789"
pattern="b*9"

# Remove SHORTEST match of pattern from back
echo "${var%$pattern}"   # abcd12345a
# Removed: "bc6789" (shortest match of b*9 at the end)

# Remove LONGEST match of pattern from back
echo "${var%%$pattern}"  # a
# Removed: "bcd12345abc6789" (longest match of b*9 at the end)

# Practical: File extension handling
filename="archive.tar.gz"
echo "${filename%.*}"    # archive.tar (remove shortest .* from end)
echo "${filename%%.*}"   # archive (remove longest .* from end)

# Practical: Directory path handling
path="/home/user/docs/file.txt"
dirname="${path%/*}"     # /home/user/docs (remove shortest /* from end)
dirname="${path%%/*}"    # (empty - would remove everything)

# More useful:
path="/home/user/docs/file.txt"
dirname="${path%/*}"     # /home/user/docs (removes /file.txt)
```

**Visualizing the difference:**

```bash
var="one-two-three-four"

# From FRONT (#):
echo "${var#*-}"         # two-three-four (#: "one-")
echo "${var##*-}"        # four (##: "one-two-three-")

# From BACK (%):
echo "${var%-*}"         # one-two-three (%-*: "-four")
echo "${var%%-*}"        # one (%%: "-two-three-four")
```

---

## Example 8-9 (Enhanced): String Length and Practical Applications

```bash
#!/bin/bash
# string-length.sh: Demonstrate string length operations

echo "=== String Length Operations ==="
echo

var01="abcdEFGH28ij"
echo "String: $var01"
echo "Length: ${#var01} characters"
echo

# Array length
arr=("apple" "banana" "cherry" "date")
echo "Array: ${arr[@]}"
echo "Array elements: ${#arr[@]}"
echo "First element length: ${#arr[0]}"
echo

# Function: Strip leading zeros (common data cleaning task)
strip_leading_zeros() {
  local num="$1"
  # Remove all leading zeros: shortest match of 0*
  echo "${num#0}"
}

# Note: This removes ONE leading zero. For multiple:
strip_all_leading_zeros() {
  local num="$1"
  # Keep removing the leading character while it's 0
  while [[ "$num" =~ ^0[0-9] ]]; do
    num="${num#0}"
  done
  echo "$num"
}

echo "=== Leading Zero Stripping ==="
echo "011 stripped: $(strip_leading_zeros 011)"  # 11
echo "007 stripped: $(strip_leading_zeros 007)"  # 7
echo "000 stripped: $(strip_leading_zeros 000)"  # 00 (one zero removed)
echo "00012 fully stripped: $(strip_all_leading_zeros 00012)"  # 12
echo

# Practical: Validate input format
validate_phone() {
  local phone="$1"
  if (( ${#phone} != 10 )); then
    echo "Error: Phone must be 10 digits"
    return 1
  fi
  echo "✓ Valid phone number"
  return 0
}

validate_phone "5551234567"    # ✓ Valid phone number
validate_phone "555123"        # Error: Phone must be 10 digits

exit 0
```

---

## Example 8-10 (Enhanced): Pattern Matching in Substring Removal

```bash
#!/bin/bash
# pattern-removal.sh: Comprehensive pattern matching demonstration

echo "=== Pattern Matching: Removal from Front (#, ##) ==="
echo

var1="abcd12345abc6789"
echo "Original: var1 = "$var1""
echo "Length: ${#var1}"
echo

pattern1="a*c"
echo "Pattern: "$pattern1""
echo "  ${var1#$pattern1} (# - shortest removal)"
echo "    └─ Removes: "abc" (shortest a*c at start)"
echo
echo "  ${var1##$pattern1} (## - longest removal)"
echo "    └─ Removes: "abcd12345abc" (longest a*c at start)"
echo

# Show with more practical patterns
echo "=== Practical Examples: File Extensions ==="
filename="document.tar.gz"
echo "Filename: "$filename""
echo "  First extension:  ${filename%.*}     (% - shortest .*)"
echo "  Base name:        ${filename%%.*}    (%% - longest .*)"
echo

# Show with path examples
echo "=== Practical Examples: Path Parsing ==="
path="/home/user/documents/file.txt"
echo "Path: "$path""
echo "  Basename:         ${path##*/}        (## - longest */)"
echo "  Directory:        ${path%/*}         (% - shortest /*)"
echo

# Advanced: Extract without extension
echo "=== Advanced: File Processing ==="
file_with_ext="mydata.backup.tar.gz"
echo "File: "$file_with_ext""
basename_only="${file_with_ext##*/}"       # Remove path
name_no_ext="${basename_only%.*}"          # Remove extension
echo "  Full path removed: $basename_only"
echo "  Extension removed: $name_no_ext"
echo

# Practical: Environment variable parsing
echo "=== Practical: Parse Full Name ==="
fullname="John Michael Smith"
firstname="${fullname%% *}"    # Remove from first space onwards
lastname="${fullname##* }"     # Remove from start to last space
echo "Full name: "$fullname""
echo "  First name: $firstname"
echo "  Last name: $lastname"

exit 0
```

---

## 8.3. Substring Extraction and Replacement

### Understanding Substring Operations

Parameter expansion provides two key substring operations:
1. **Extraction** (`${var:offset:length}`): Get a portion of the string
2. **Replacement** (`${var/pattern/replacement}`): Transform the string

These are fundamental for text processing without spawning external commands like `cut` or `sed`.

### Substring Extraction: `:offset:length`

The syntax `${var:offset:length}` extracts a substring starting at `offset` with `length` characters.

**Parameters:**
- `offset`: Starting position (0-indexed). Negative values mean "from the end"
- `length`: Number of characters to extract (optional - if omitted, goes to end)

```bash
var="hello world"

# From position 0 to end
echo "${var:0}"         # hello world

# From position 6 to end
echo "${var:6}"         # world

# 5 characters starting at position 0
echo "${var:0:5}"       # hello

# 5 characters starting at position 6
echo "${var:6:5}"       # world

# Last 5 characters (note the space after colon with negative)
echo "${var: -5}"       # world (must have space before -)

# All but last character
echo "${var:0:${#var}-1}"  # hello worl

# Middle of string
echo "${var:3:4}"       # lo w (4 chars starting at position 3)
```

**Important:** When using negative offsets, you MUST include a space: `${var: -5}` not `${var:-5}` (which is the default operator).

**Practical example: Time formatting**

```bash
time_str="14:35:42"
hour="${time_str:0:2}"      # 14
minute="${time_str:3:2}"    # 35
second="${time_str:6:2}"    # 42

echo "Time - $hour hours, $minute minutes, $second seconds"
```

**Practical example: Process data from fixed-format file**

```bash
# A record with fixed-width fields
record="JOHN       SMITH      1985-06-15"

firstname="${record:0:10}"       # JOHN       (positions 0-9)
lastname="${record:10:10}"       # SMITH      (positions 10-19)
birthdate="${record:20:10}"      # 1985-06-15 (positions 20-29)

echo "Name: ${firstname% } ${lastname% }"  # Use % to trim trailing spaces
echo "Born: $birthdate"
```

### Substring Replacement: Pattern Matching and Substitution

Parameter expansion provides multiple replacement operators for different matching strategies:

| Operator | Match | Behavior |
|----------|-------|----------|
| `${var/pattern/repl}` | Anywhere | Replace **first** occurrence |
| `${var//pattern/repl}` | Anywhere | Replace **all** occurrences |
| `${var/#pattern/repl}` | Start only | Replace if matches at **beginning** |
| `${var/%pattern/repl}` | End only | Replace if matches at **end** |

**Basic replacement: First match**

```bash
var="hello world hello"

# Replace first match of "hello"
echo "${var/hello/goodbye}"     # goodbye world hello

# Delete first match (replace with nothing)
echo "${var/hello/}"            # world hello
```

**Global replacement: All matches**

```bash
var="hello world hello"

# Replace all matches of "hello"
echo "${var//hello/goodbye}"    # goodbye world goodbye

# Delete all matches
echo "${var//hello/}"           # world

# Pattern matching (not literal strings)
var="aaa bbb aaa"
echo "${var//a/X}"              # XXX bbb XXX
```

**Practical: Clean filename for safe storage**

```bash
filename="my report (final).txt"

# Remove spaces
safe="${filename// /_}"          # my_report_(final).txt

# Remove parentheses
safe="${safe//(/}"               # my_report_final).txt
safe="${safe//)/}"               # my_report_final.txt

echo "Safe filename: $safe"
```

### Prefix and Suffix Matching: `/# and `/%`

These operators match patterns only at specific positions (start or end):

| Operator | Position | Behavior |
|----------|----------|----------|
| `${var/#pattern/repl}` | **Start** | Replace only if pattern matches at beginning |
| `${var/%pattern/repl}` | **End** | Replace only if pattern matches at end |

```bash
var="abc1234xyz"

# Prefix matching - replaces abc at start
echo "${var/#abc/ABC}"   # ABC1234xyz

# Prefix doesn't match - no change
echo "${var/#xyz/XYZ}"   # abc1234xyz (xyz not at start)

# Suffix matching - replaces xyz at end
echo "${var/%xyz/XYZ}"   # abc1234XYZ

# Suffix doesn't match - no change
echo "${var/%abc/ABC}"   # abc1234xyz (abc not at end)
```

**Practical: Protocol upgrading**

```bash
url="http://example.com/api"

# Upgrade http to https (only if starts with http)
secure_url="${url/#http:/https:}"
echo "$secure_url"       # https://example.com/api

# This preserves https URLs unchanged
url2="https://example.com"
echo "${url2/#http:/https:}"    # https://example.com (unchanged)
```

**Practical: Configuration file parsing**

```bash
# Remove comments from config lines
config_line="server=192.168.1.1  # Internal server"

# Remove everything from # onwards (if present)
cleaned="${config_line/%' #'*/}"  # Might work with pattern
# Better approach for this use case:
cleaned="${config_line%% #*}"     # Uses % operator from section 8.2
echo "Cleaned: $cleaned"
```

---

## Example 8-12 (Enhanced): Advanced Pattern Matching for String Parsing

```bash
#!/bin/bash
# string-parsing.sh: Comprehensive string parsing examples

echo "=== Parsing Structured Data ==="
echo

# Parse a delimited string
var1="abcd-1234-defg-5678"
echo "Original: $var1"
echo

# Extract components
echo "Components using # (remove from front):"
after_first_dash="${var1#*-}"
echo "  After 1st dash: $after_first_dash"    # 1234-defg-5678

after_last_dash="${var1##*-}"
echo "  After last dash: $after_last_dash"    # 5678
echo

# Using % (remove from back)
before_last_dash="${var1%*-*}"
echo "  Before last dash: $before_last_dash"  # abcd-1234-defg
echo

# Parse file paths
echo "=== File Path Manipulation ==="
path_name="/home/user/docs/project/file.txt"
echo "Path: $path_name"
echo

# Get just the filename
basename="${path_name##*/}"
echo "  Basename (filename):     $basename"         # file.txt

# Get the directory
dirname="${path_name%/*}"
echo "  Directory:               $dirname"          # /home/user/docs/project

# Get filename without extension
noext="${basename%.*}"
echo "  Filename (no ext):       $noext"            # file

# Get the file extension
ext="${basename##*.}"
echo "  Extension:               $ext"              # txt
echo

# Extract using substring
echo "=== Substring Extraction ==="
pos=${path_name#/home/user/}
echo "  From position 11:        $pos"              # docs/project/file.txt

partial="${path_name:11:16}"
echo "  Position 11, length 16:  $partial"         # docs/project/fil
echo

# Replace patterns
echo "=== Replacement Operations ==="

# Replace specific component
new_path="${path_name/user/admin}"
echo "  User→Admin:              $new_path"        # /home/admin/docs/project/file.txt

new_path="${path_name/file/backup}"
echo "  File→Backup:             $new_path"        # /home/user/docs/project/backup.txt

new_path="${path_name//o/O}"
echo "  All 'o'→'O':             $new_path"        # /hOme/user/dOcs/prOject/file.txt
echo

# Practical: URL path extraction
echo "=== URL Parsing ==="
url="https://example.com:8080/api/users?id=123"
echo "URL: $url"

# Remove protocol
domain="${url##*://}"                  # example.com:8080/api/users?id=123
echo "  Without protocol: $domain"

# Remove port and path (just domain)
domain_only="${domain%%:*}"            # example.com
echo "  Domain only: $domain_only"

exit 0
```

---

## Example 8-13 (Enhanced): Prefix and Suffix Pattern Matching

```bash
#!/bin/bash
# prefix-suffix.sh: Demonstrate prefix and suffix pattern matching

echo "=== Prefix and Suffix Pattern Matching ==="
echo

v0="abc1234zip1234abc"
echo "Original: v0 = "$v0""
echo

echo "Pattern at START (prefix):"
echo "  Replace 'abc' at start: ${v0/#abc/ABCDEF}"
# abc1234zip1234abc → ABCDEF1234zip1234abc

echo "  Try to replace 'zip' at start: ${v0/#zip/ZIP}"
# abc1234zip1234abc (no change - zip not at start)
echo

echo "Pattern at END (suffix):"
echo "  Replace 'abc' at end: ${v0/%abc/ABCDEF}"
# abc1234zip1234abc → abc1234zip1234ABCDEF

echo "  Try to replace '1234' at end: ${v0/%1234/XXXX}"
# abc1234zip1234abc (no change - 1234 not at end)
echo

echo "=== Practical Use Case: URL Protocol Enforcement ==="
echo

process_url() {
  local url="$1"
  local secure="${url/#http:/https:}"
  
  if [[ "$secure" != "$url" ]]; then
    echo "Upgraded: $url"
    echo "       To: $secure"
  else
    echo "Already secure or not HTTP: $url"
  fi
}

process_url "http://example.com"       # Upgraded
process_url "https://example.com"      # Already secure
process_url "ftp://example.com"        # Already secure (not HTTP)

echo

echo "=== Practical Use Case: Configuration Value Processing ==="
echo

config_var="value"
if [[ "$config_var" == "true" || "$config_var" == "yes" || "$config_var" == "1" ]]; then
  echo "Configuration is enabled"
fi

# Alternative using parameter expansion with default check
enabled="${config_var:+yes}"
echo "Enabled check: ${enabled:-no}"

exit 0
```

---

## 8.4. Renaming and Transforming Files with Parameter Expansion

### The Power of Parameter Expansion for File Operations

Parameter expansion eliminates the need for external commands in many file operations, making scripts faster and more portable. The `${var%pattern}` and `${var#pattern}` operators are perfect for batch file renaming.

### Example 8-11 (Enhanced): Batch File Extension Renaming

```bash
#!/bin/bash
# rfe.sh: Batch file extension renaming using parameter expansion
# Usage: rfe.sh old_extension new_extension
# Example: rfe.sh gif jpg

E_BADARGS=65
E_NOFILES=66

# Validate arguments
if [ $# -ne 2 ]; then
  echo "Usage: $(basename $0) old_extension new_extension"
  echo "Examples:"
  echo "  $(basename $0) txt bak    # Rename all .txt to .bak"
  echo "  $(basename $0) jpg png    # Rename all .jpg to .png"
  exit $E_BADARGS
fi

old_ext="$1"
new_ext="$2"

# Check if files with old extension exist
if ! compgen -G "*.$old_ext" > /dev/null; then
  echo "Error: No files found with extension .$old_ext"
  exit $E_NOFILES
fi

count=0
for filename in *.$old_ext; do
  # Skip if no files match (shouldn't happen with compgen check, but safe)
  [ -f "$filename" ] || break
  
  # Remove old extension and append new one
  # Using ${filename%.*} removes everything from last dot onwards
  # Then append the new extension
  newname="${filename%.$old_ext}.$new_ext"
  
  # Verify files are different before attempting rename
  if [ "$filename" = "$newname" ]; then
    echo "Skip: $filename (source and target are the same)"
    continue
  fi
  
  # Perform the rename
  if mv "$filename" "$newname"; then
    echo "✓ Renamed: $filename → $newname"
    ((count++))
  else
    echo "✗ Failed: $filename (rename error)"
  fi
done

echo
echo "Total files renamed: $count"
exit 0
```

**Usage examples:**

```bash
# Rename all .txt files to .bak
$ ./rfe.sh txt bak
✓ Renamed: file1.txt → file1.bak
✓ Renamed: file2.txt → file2.bak
Total files renamed: 2

# Rename all .jpg files to .png
$ ./rfe.sh jpg png
✓ Renamed: photo.jpg → photo.png
Total files renamed: 1
```

**How it works:**
1. Takes old and new extensions as arguments
2. Iterates through all files matching `*.old_extension`
3. For each file, removes the old extension with `${filename%.$old_ext}`
4. Appends the new extension
5. Renames the file and tracks success

---

## 8.5. Indirect References and Variable Name Expansion

### Understanding Indirect References

Indirect references allow you to use a variable's **value** as another variable's **name**. This is powerful for dynamic variable access but can make code harder to follow—use carefully.

**Key concept:** `${!varname}` dereferences the variable. If `varname` contains "other_var", then `${!varname}` is equivalent to `${other_var}`.

### Basic Indirect References: `${!varname}`

```bash
# Direct reference: var contains the value
var="hello"
echo $var              # hello

# Indirect reference: var_name contains another variable's name
var_name="var"
echo ${!var_name}      # hello (dereferences to $var)

# More complex example
password_var="db_password"
db_password="secretpassword"
echo ${!password_var}  # secretpassword
```

**Practical: Dynamic variable selection**

```bash
# Set up configuration variables
production_host="prod.example.com"
development_host="dev.example.com"
staging_host="stage.example.com"

# Select based on environment
environment="${1:-development}"
host_var="${environment}_host"

# Access the appropriate host without conditional
target_host="${!host_var}"
echo "Connecting to: $target_host"

# Run: ./script.sh production  → prod.example.com
# Run: ./script.sh development → dev.example.com
# Run: ./script.sh staging     → stage.example.com
```

**Practical: Feature flags and configuration**

```bash
enable_debug="debug_mode"
debug_mode=1

enable_verbose="verbose_mode"
verbose_mode=0

# Read flags dynamically
if (( ${!enable_debug} )); then
  echo "Debug mode is ON"
fi

if (( ${!enable_verbose} )); then
  echo "Verbose mode is ON"
fi
```

### Variable Name Matching: `${!prefix*}` and `${!prefix@}`

These operators expand to all variable names that start with a given prefix. `@` vs `*` matters only in arrays (not relevant here).

**Use cases:**
- Iterate over related variables
- Build dynamic configuration
- Registry patterns

```bash
# Set up related variables
item1="first"
item2="second"
item3="third"

# Get all variable names starting with 'item'
# Expands to the list: "item1" "item2" "item3"
echo "${!item@}"       # item1 item2 item3
echo "${!item*}"       # item1 item2 item3 (same for non-array context)

# Iterate and access values
for var_name in ${!item@}; do
  echo "$var_name = ${!var_name}"
done

# Output:
# item1 = first
# item2 = second
# item3 = third
```

**Practical: Configuration registry iteration**

```bash
# Define configuration with consistent naming
db_host="localhost"
db_port="5432"
db_user="admin"
db_pass="secret"

# Validate all db_* variables are set
echo "Configuration check:"
for var_name in ${!db_@}; do
  if [[ -z "${!var_name}" ]]; then
    echo "  ✗ $var_name is empty"
  else
    echo "  ✓ $var_name is set"
  fi
done

# Output:
#   ✓ db_host is set
#   ✓ db_port is set
#   ✓ db_user is set
#   ✓ db_pass is set
```

**Practical: Build command-line arguments dynamically**

```bash
# Flags are off by default
flag_verbose=0
flag_debug=0
flag_dry_run=0

# Enable based on environment or input
if [[ "$DEBUG" == "1" ]]; then
  flag_debug=1
fi

# Build argument string
args=""
for var_name in ${!flag_@}; do
  if (( ${!var_name} )); then
    # Convert flag_verbose to --verbose
    arg_name="${var_name#flag_}"
    arg_name="${arg_name//_/-}"
    args="${args:+$args }--${arg_name}"
  fi
done

echo "Running with: $args"
```

---

## 8.6. Summary of Parameter Expansion Operators

| Operator | Purpose | Example | Notes |
|----------|---------|---------|-------|
| `${var:-default}` | Use default if unset/null | `${name:-Guest}` | Treats empty as missing |
| `${var-default}` | Use default if unset | `${name-Guest}` | Allows empty values |
| `${var:=default}` | Set and use default if unset/null | `${count:=0}` | Modifies variable |
| `${var:+alt}` | Use alt if set/non-null | `${file:+/path/$file}` | Opposite of `:- ` |
| `${var?error}` | Exit with error if unset | `${USER?}` | Fail-fast validation |
| `${var:?error}` | Exit with error if unset/null | `${DB_HOST:?}` | More strict validation |
| `${#var}` | String length | `${#name}` | Character count |
| `${var#pattern}` | Remove shortest from front | `${file#*.}` | Glob pattern match |
| `${var##pattern}` | Remove longest from front | `${path##*/}` | Glob pattern match |
| `${var%pattern}` | Remove shortest from back | `${file%.*}` | Glob pattern match |
| `${var%%pattern}` | Remove longest from back | `${file%%.*}` | Glob pattern match |
| `${var:offset}` | Substring from offset | `${str:5}` | 0-indexed |
| `${var:offset:len}` | Substring with length | `${str:0:5}` | 0-indexed |
| `${var/pattern/repl}` | Replace first match | `${path/old/new}` | Glob pattern match |
| `${var//pattern/repl}` | Replace all matches | `${str//a/b}` | Glob pattern match |
| `${var/#pattern/repl}` | Replace prefix match | `${str/#pre/PRE}` | Glob pattern match |
| `${var/%pattern/repl}` | Replace suffix match | `${str/%end/END}` | Glob pattern match |
| `${!varname}` | Indirect reference | `${!ref}` | Dereference |
| `${!prefix*}` | Names of vars with prefix | `${!item*}` | Expansion to names |

---

## Best Practices and Common Patterns

### 1. Always Use `:-` for Function Defaults (Not Arguments)

```bash
# Good: Provides clean default for optional parameter
function log_message() {
  local level="${1:-INFO}"
  local message="${2:-No message}"
  echo "[$level] $message"
}

log_message                  # [INFO] No message
log_message "ERROR"          # [ERROR] No message
log_message "ERROR" "Failed" # [ERROR] Failed
```

### 2. Use `:=` for Lazy Initialization

```bash
# Good: Initialize counter only when first used
bump_counter() {
  : ${counter:=0}
  ((counter++))
  echo "Count: $counter"
}

bump_counter  # Count: 1
bump_counter  # Count: 2
```

### 3. Use `?` to Make Dependencies Explicit

```bash
# Good: Explicit error when required variable is missing
#!/bin/bash
: ${API_KEY?"API_KEY environment variable must be set"}
: ${API_URL?"API_URL environment variable must be set"}

# Script continues only if both variables are set
echo "Using API at $API_URL"
```

### 4. Prefer Parameter Expansion Over External Commands

```bash
# Slower: Spawns sed process
dirname=$(echo "$path" | sed 's|/[^/]*$||')

# Faster: Pure Bash parameter expansion
dirname="${path%/*}"

# Slower: Spawns basename process
filename=$(basename "$path")

# Faster: Pure Bash parameter expansion
filename="${path##*/}"
```

### 5. Use `##` and `%%` for Last Occurrence

```bash
# Extract from last delimiter
filename="archive.tar.gz"
echo "${filename##*.}"  # gz (not tar.gz)
echo "${filename%.*}"   # archive.tar (not just archive)
```

### 6. Combine Operators for Complex Transformations

```bash
# Get filename without extension from full path
path="/home/user/docs/file.tar.gz"

# Step 1: Remove directory
filename="${path##*/}"           # file.tar.gz

# Step 2: Remove extension
no_extension="${filename%.*}"    # file.tar

# Combined in one variable assignment
no_extension="${path##*/}"
no_extension="${no_extension%.*}"

# Or using nested parameter expansion
no_extension="${${path##*/}%.*}"  # Not supported - must chain
```

---

## 10 Core Programming Concepts from Parameter Expansion

### 1. **Default Values and Null Coalescing**
Parameter expansion's `:-` operator implements the null coalescing pattern: use a value if available, otherwise use a sensible default. This is fundamental to defensive programming and appears in virtually every programming language.

```bash
name="${name:-Unknown}"  # Null coalescing
```

### 2. **Lazy Initialization**
The `:=` operator performs lazy initialization—setting a variable only when first accessed. This pattern delays computation until needed and is crucial for performance-sensitive code.

```bash
cache="${cache:=expensive_computation}"
```

### 3. **Fail-Fast Validation**
The `?` operator implements fail-fast: validate preconditions immediately and exit with clear error messages if violated. This prevents silent failures and unclear errors later.

```bash
: ${REQUIRED_VAR?}  # Exit immediately if not set
```

### 4. **Pattern Matching and Text Processing**
The `#`, `##`, `%`, `%%` operators implement glob pattern matching directly in Bash without external commands. This demonstrates how modern languages integrate pattern matching for string manipulation.

```bash
basename="${path##*/}"  # Match up to last /
```

### 5. **Substring Operations**
The `:offset:length` syntax provides substring extraction (slicing), a fundamental operation for string manipulation in all languages.

```bash
substring="${string:0:5}"  # Get first 5 characters
```

### 6. **String Replacement and Transformation**
The `/pattern/replacement` operators implement find-and-replace, with variants for first-match, all-matches, prefix-match, and suffix-match. This is a common task across all text processing.

```bash
cleaned="${text//whitespace/}"  # Remove all whitespace
```

### 7. **Conditional Execution Based on Variable State**
The `:+` operator demonstrates conditional logic based on variable existence. It's a form of guard clause: "use this value only if the condition is true."

```bash
option="${DEBUG:+--debug}"  # Only add if DEBUG is set
```

### 8. **Indirection and Dynamic Access**
The `${!name}` operator implements variable indirection—using a variable's value as another variable's name. This demonstrates the power of treating variable names as first-class values.

```bash
value="${!variable_ref}"  # Dereference through name
```

### 9. **Reflection and Introspection**
The `${!prefix*}` operator allows programs to discover what variables exist at runtime—a form of reflection that's essential for configuration management and dynamic behavior.

```bash
all_flags="${!flag_*}"  # Find all flag_* variables
```

### 10. **Compound Predicates and Multi-Level Fallbacks**
By combining multiple operators, you create compound logic like "try this, then try that, then use default." This is the basis for robust error handling and fallback chains.

```bash
value="${OVERRIDE:-${ENVIRONMENT_VAR:-${DEFAULT_VALUE}}}"
```

---

## Exercises

1. **Write a function** that uses `:-` to set default values for username, hostname, and port, falling back to sensible defaults if environment variables aren't set.

2. **Create an argument validator** that uses `${parameter?}` to enforce required arguments and exit with a helpful error message if they're missing.

3. **Implement a file renaming tool** using `${var%pattern}` and `${var#pattern}` to batch-rename all files in a directory (e.g., remove extension, change prefix).

4. **Write a URL parser** that uses substring extraction and removal operators to extract protocol, domain, path, and query components from a URL.

5. **Create a variable registry system** that uses `${!prefix*}` to find all variables starting with a given prefix and print their names and values.

6. **Build a configuration loader** that uses `:=` to set uninitialized variables to defaults while ensuring they remain persistent within a script execution.

7. **Implement a text sanitizer** that uses `${var//pattern/repl}` to clean user input by removing or replacing special characters.

8. **Write a log formatter** that uses prefix and suffix matching (`/#` and `/%`) to add timestamp prefixes and log level suffixes to messages.

9. **Create a command-line flag builder** that uses `${var:+}` to conditionally add flags to a command based on whether variables are set.

10. **Develop a path manipulator** that combines multiple operators to extract and transform file paths (dirname, basename, extension, no-extension).

---

## Summary

**Default value operators** provide intelligent fallback mechanisms:
- **`${var:-default}`** is the most common—use default if unset/null
- **`${var:=default}`** also assigns the default permanently
- **`${var:+alt}`** uses alternate only if variable exists (opposite pattern)
- **`${var?error}`** enforces required variables with fail-fast validation

**String operations** allow sophisticated text processing without external commands:
- **`${#var}`** provides character count
- **`${var#pattern}` and `${var##pattern}`** remove from the front (shortest vs. longest)
- **`${var%pattern}` and `${var%%pattern}`** remove from the back (shortest vs. longest)
- **`${var:offset:length}`** extracts substrings by position
- **`${var/pattern/repl}`** replaces patterns (first or all matches)
- **`${var/#pattern/repl}` and `${var/%pattern/repl}`** match at boundaries

**Advanced features** enable dynamic behavior:
- **`${!varname}`** provides variable indirection (dereference)
- **`${!prefix*}`** discovers variables by name prefix (introspection)

**Key takeaway:** Parameter expansion is Bash's most powerful built-in feature for text manipulation and conditional logic. Master it to write faster, more portable, more robust scripts that don't depend on external utilities like `sed`, `awk`, or `cut`.
