# Chapter 8: Parameter Expansion and Substitution

Parameter expansion allows you to manipulate variables in powerful ways: providing defaults, testing for existence, extracting substrings, removing patterns, and replacing text. These operators are essential for robust script writing.

## 8.1. Default Values and Parameter Testing

{% tabs %}
{% tab title="${parameter:-default}" %}

If `parameter` is unset or null, use `default`. Otherwise, use `parameter`.

```bash
# Variable not set
echo "${undefined:-fallback}"    # fallback

# Variable set to empty string
var=""
echo "${var:-fallback}"          # fallback (treats null as unset)

# Variable set to a value
var="value"
echo "${var:-fallback}"          # value

# Practical use: function defaults
greet() {
  local name="${1:-Guest}"
  echo "Hello, $name!"
}

greet                  # Hello, Guest!
greet "Alice"          # Hello, Alice!
```

{% endtab %}
{% tab title="${parameter:=default}" %}

If `parameter` is unset or null, set it to `default` and use that value.

```bash
echo "${myvar:=newvalue}"   # newvalue
echo "$myvar"               # newvalue (variable was set)

# Useful for initialization
: ${count:=0}
((count++))
echo $count                 # 1
```

{% endtab %}
{% tab title="${parameter:+alt_value}" %}

If `parameter` is set and non-null, use `alt_value`. Otherwise, use nothing.

```bash
var="something"
echo "${var:+replacement}"  # replacement

var=""
echo "${var:+replacement}"  # (empty)

var_unset=""
echo "${undefined:+value}"  # (empty)

# Useful for conditional appending
file="${1:+/path/to/$1}"    # Only append path if $1 is provided
```

{% endtab %}
{% tab title="${parameter?err_msg}" %}

If `parameter` is set, use it. Otherwise, print `err_msg` and exit with status 1.

```bash
: ${HOME?}                  # Exit if HOME not set
: ${USER?}                  # Exit if USER not set

username=${1?"Missing required argument: username"}
# Script exits if $1 is not provided

# In a real scenario
: ${HOSTNAME?} ${USER?} ${HOME?} ${MAIL?}
echo "All required variables are set."
```

{% endtab %}
{% endtabs %}

### Example 8-7: Parameter Testing with Error Messages

```bash
#!/bin/bash
# Check essential environmental variables

: ${HOSTNAME?} ${USER?} ${HOME?} ${MAIL?}

echo "System Check: PASSED"
echo "Hostname: $HOSTNAME"
echo "User: $USER"
echo "Home: $HOME"
echo "Mail: $MAIL"
echo

# Check script-specific variables
script_var="important_value"
: ${script_var?}
echo "Script variable is set: $script_var"

echo
# This will cause the script to exit if ZZXy23AB is not defined
: ${ZZXy23AB?"Variable ZZXy23AB has not been set."}

echo "This line will not print because ZZXy23AB is undefined."
exit 0
```

### Example 8-8: Usage Messages

```bash
#!/bin/bash
# usage-message.sh: Require command-line arguments

: ${1?"Usage: $0 ARGUMENT"}

echo "You provided an argument: $1"
exit 0

# If called without arguments, exits with error:
# bash: 1: Usage: usage-message.sh ARGUMENT
```

---

## 8.2. String Length and Substring Removal

{% tabs %}
{% tab title="String Length ${#var}" %}

Get the length (number of characters) of a variable.

```bash
var="hello"
echo "${#var}"          # 5

var="hello world"
echo "${#var}"          # 11 (includes space)

# Array elements
array=("one" "two" "three")
echo "${#array}"        # 3 (length of first element)
echo "${#array[@]}"     # 3 (number of elements)

# Positional parameters
echo "${#@}"            # Number of arguments passed to script
```

{% endtab %}
{% tab title="Remove from Front: #, ##" %}

| Operator | Behavior |
|----------|----------|
| `${var#Pattern}` | Remove shortest match of Pattern from **front** |
| `${var##Pattern}` | Remove longest match of Pattern from **front** |

```bash
var="abcd12345abc6789"
pattern="a*c"

echo "${var#$pattern}"   # 12345abc6789 (shortest: removes "abc")
echo "${var##$pattern}"  # 6789 (longest: removes "abcd12345abc")

# Practical examples
filename="archive.tar.gz"
echo "${filename#*.}"    # tar.gz (remove up to first dot)
echo "${filename##*.}"   # gz (remove up to last dot)

# Get basename
path="/home/user/docs/file.txt"
echo "${path##*/}"       # file.txt (remove longest match of */ from front)
```

{% endtab %}
{% tab title="Remove from Back: %, %%" %}

| Operator | Behavior |
|----------|----------|
| `${var%Pattern}` | Remove shortest match of Pattern from **back** |
| `${var%%Pattern}` | Remove longest match of Pattern from **back** |

```bash
var="abcd12345abc6789"
pattern="b*9"

echo "${var%$pattern}"   # abcd12345a (shortest: removes "bc6789")
echo "${var%%$pattern}"  # a (longest: removes "bcd12345abc6789")

# Practical examples
filename="archive.tar.gz"
echo "${filename%.*}"    # archive.tar (remove shortest .ext)
echo "${filename%%.*}"   # archive (remove longest .ext)

# Get directory path
path="/home/user/docs/file.txt"
echo "${path%/*}"        # /home/user/docs (remove shortest /* from back)
```

{% endtab %}
{% endtabs %}

### Example 8-9: Length of a Variable

```bash
#!/bin/bash
# length.sh: Demonstrating string length

var01="abcdEFGH28ij"
echo "var01 = $var01"
echo "Length = ${#var01}"

var02="abcd EFGH28ij"
echo "var02 = $var02"
echo "Length = ${#var02}"

echo "Number of arguments: ${#@}"
echo "Number of arguments: ${#*}"

# Strip leading zeros (practical example)
strip_leading_zero() {
  echo ${1#0}
}

echo "011 with leading zero stripped: $(strip_leading_zero 011)"
echo "007 with leading zero stripped: $(strip_leading_zero 007)"

exit 0
```

### Example 8-10: Pattern Matching in Substitution

```bash
#!/bin/bash
# patt-matching.sh: Using #, ##, %, %% operators

var1="abcd12345abc6789"
echo "var1 = $var1"
echo "Length = ${#var1}"
echo

# --- Remove from FRONT ---
pattern1="a*c"
echo "Pattern: $pattern1"
echo "  # (shortest): ${var1#$pattern1}"    # d12345abc6789
echo "  ## (longest): ${var1##$pattern1}"   # 6789
echo

# --- Remove from BACK ---
pattern2="b*9"
echo "Pattern: $pattern2"
echo "  % (shortest): ${var1%$pattern2}"    # abcd12345a
echo "  %% (longest): ${var1%%$pattern2}"   # a
echo

# --- Practical: File extensions ---
filename="document.tar.gz"
echo "Filename: $filename"
echo "  Remove shortest ext: ${filename%.*}"   # document.tar
echo "  Remove longest ext: ${filename%%.*}"   # document
echo

# --- Practical: Path parsing ---
path="/home/user/documents/file.txt"
echo "Path: $path"
echo "  Basename: ${path##*/}"              # file.txt
echo "  Directory: ${path%/*}"              # /home/user/documents

exit 0
```

---

## 8.3. Substring Extraction and Replacement

{% tabs %}
{% tab title="Substring Extraction: :pos:len" %}

Extract a substring starting at position `pos` with length `len`.

```bash
var="hello world"

echo "${var:0}"         # hello world (from position 0 to end)
echo "${var:6}"         # world (from position 6 to end)
echo "${var:0:5}"       # hello (5 chars starting at position 0)
echo "${var:6:5}"       # world (5 chars starting at position 6)

# Negative offsets (from the end)
echo "${var: -5}"       # world (last 5 characters)
echo "${var:0:${#var}-1}"  # hello worl (all but last char)
```

{% endtab %}
{% tab title="Substring Replacement: /" %}

Replace patterns within a string.

| Operator | Behavior |
|----------|----------|
| `${var/Pattern/Replacement}` | Replace **first** match |
| `${var//Pattern/Replacement}` | Replace **all** matches |

```bash
var="hello world hello"

# Replace first match
echo "${var/hello/goodbye}"     # goodbye world hello

# Replace all matches
echo "${var//hello/goodbye}"    # goodbye world goodbye

# Delete (replace with nothing)
echo "${var/hello/}"            # world hello
echo "${var//hello/}"           # world

# Practical: sanitize filenames
filename="my file (old).txt"
safe_name="${filename// /_}"    # my_file_(old).txt
safe_name="${safe_name//(/}"    # my_file_old).txt
```

{% endtab %}
{% tab title="Prefix/Suffix Matching" %}

Replace only if pattern matches at prefix or suffix.

| Operator | Behavior |
|----------|----------|
| `${var/#Pattern/Replacement}` | Replace if **prefix** matches |
| `${var/%Pattern/Replacement}` | Replace if **suffix** matches |

```bash
var="abc1234xyz"

# Prefix matching
echo "${var/#abc/ABC}"   # ABC1234xyz (replaces abc at start)
echo "${var/#xyz/XYZ}"   # abc1234xyz (xyz not at start, no match)

# Suffix matching
echo "${var/%xyz/XYZ}"   # abc1234XYZ (replaces xyz at end)
echo "${var/%abc/ABC}"   # abc1234xyz (abc not at end, no match)

# Practical: URL protocol handling
url="http://example.com"
echo "${url/#http/https}"    # https://example.com
```

{% endtab %}
{% endtabs %}

### Example 8-12: Pattern Matching for String Parsing

```bash
#!/bin/bash
# var-match.sh: Parse strings using pattern substitution

var1="abcd-1234-defg"
echo "var1 = $var1"

# Remove everything up to and including first '-'
t=${var1#*-}
echo "After first dash: $t"          # 1234-defg

# Remove everything up to and including last '-'
t=${var1##*-}
echo "After last dash: $t"           # defg

# Remove everything from last '-' onwards
t=${var1%*-*}
echo "Before last dash: $t"          # abcd-1234

echo

# --- File path manipulation ---
path_name="/home/user/docs/file.txt"
echo "path_name = $path_name"

# Get filename only (basename)
basename="${path_name##*/}"
echo "Basename: $basename"           # file.txt

# Get directory (dirname)
dirname="${path_name%/*}"
echo "Directory: $dirname"           # /home/user/docs

# Get filename without extension
noext="${path_name##*/}"
noext="${noext%.*}"
echo "No extension: $noext"          # file

echo

# --- Substring extraction ---
t=${path_name:11}
echo "From position 11: $t"          # docs/file.txt

t=${path_name:11:5}
echo "From position 11, length 5: $t"  # docs/

echo

# --- Replacement examples ---
t=${path_name/user/admin}
echo "Replace 'user' with 'admin': $t"  # /home/admin/docs/file.txt

t=${path_name/file/backup}
echo "Replace 'file' with 'backup': $t" # /home/user/docs/backup.txt

t=${path_name//o/O}
echo "Replace all 'o' with 'O': $t"     # /hOme/user/dOcs/file.txt

exit 0
```

### Example 8-13: Prefix and Suffix Matching

```bash
#!/bin/bash
# var-match.sh: Prefix and suffix pattern matching

v0="abc1234zip1234abc"
echo "Original: v0 = $v0"
echo

# Prefix matching (replace at beginning)
v1=${v0/#abc/ABCDEF}
echo "Prefix replacement: ${v0/#abc/ABCDEF} = $v1"
# abc1234zip1234abc → ABCDEF1234zip1234abc

# Suffix matching (replace at end)
v2=${v0/%abc/ABCDEF}
echo "Suffix replacement: ${v0/%abc/ABCDEF} = $v2"
# abc1234zip1234abc → abc1234zip1234ABCDEF

echo

# No match cases (pattern exists but not at boundary)
v3=${v0/#123/000}
echo "No prefix match: v3 = $v3"          # abc1234zip1234abc (unchanged)

v4=${v0/%123/000}
echo "No suffix match: v4 = $v4"          # abc1234zip1234abc (unchanged)

exit 0
```

---

## 8.4. Renaming Files with Pattern Substitution

### Example 8-11: Batch File Extension Renaming

```bash
#!/bin/bash
# rfe.sh: Rename file extensions in bulk
# Usage: rfe.sh old_extension new_extension
# Example: rfe.sh gif jpg

E_BADARGS=65

if [ $# -ne 2 ]
then
  echo "Usage: $(basename $0) old_extension new_extension"
  exit $E_BADARGS
fi

count=0
for filename in *.$1
do
  # Skip if no files match
  [ -f "$filename" ] || break
  
  # Strip old extension, append new one
  newname="${filename%$1}$2"
  mv "$filename" "$newname"
  echo "Renamed: $filename → $newname"
  ((count++))
done

echo "Total files renamed: $count"
exit 0
```

**Usage:**
```bash
# Rename all .txt files to .bak
./rfe.sh txt bak

# Rename all .jpg files to .png
./rfe.sh jpg png
```

---

## 8.5. Indirect References and Variable Name Expansion

{% tabs %}
{% tab title="Indirect Reference: ${!varname}" %}

Access a variable whose name is stored in another variable.

```bash
# Direct reference
var="hello"
echo $var              # hello

# Indirect reference
var_name="var"
echo ${!var_name}      # hello (dereference $var_name)

# Practical: option flags
enable_debug="debug_mode"
debug_mode=1
echo ${!enable_debug}  # 1
```

{% endtab %}
{% tab title="Matching Variable Names: ${!varprefix*}" %}

Expand to all variable names beginning with a prefix.

```bash
item1="first"
item2="second"
item3="third"

# Get all variable names starting with 'item'
all_items="${!item@}"
echo "$all_items"      # item1 item2 item3

# Iterate over matching variables
for var_name in ${!item@}
do
  echo "$var_name = ${!var_name}"
done

# Output:
# item1 = first
# item2 = second
# item3 = third
```

{% endtab %}
{% endtabs %}

---

## 8.6. Summary of Parameter Expansion Operators

| Operator | Purpose | Example |
|----------|---------|---------|
| `${var:-default}` | Use default if unset/null | `${name:-Guest}` |
| `${var:=default}` | Set and use default if unset/null | `${count:=0}` |
| `${var:+alt}` | Use alt if set/non-null | `${file:+/path/$file}` |
| `${var?error}` | Exit with error if unset | `${USER?}` |
| `${#var}` | String length | `${#name}` |
| `${var#pattern}` | Remove shortest from front | `${file#*.}` |
| `${var##pattern}` | Remove longest from front | `${path##*/}` |
| `${var%pattern}` | Remove shortest from back | `${file%.*}` |
| `${var%%pattern}` | Remove longest from back | `${file%%.*}` |
| `${var:offset}` | Substring from offset | `${str:5}` |
| `${var:offset:len}` | Substring with length | `${str:0:5}` |
| `${var/pattern/repl}` | Replace first match | `${path/old/new}` |
| `${var//pattern/repl}` | Replace all matches | `${str//a/b}` |
| `${var/#pattern/repl}` | Replace prefix match | `${str/#pre/PRE}` |
| `${var/%pattern/repl}` | Replace suffix match | `${str/%end/END}` |
| `${!varname}` | Indirect reference | `${!ref}` |
| `${!prefix*}` | Names of vars with prefix | `${!item*}` |

---

## Exercises

1. **Write a script** that uses default values with `:-` to set configuration variables, falling back to sensible defaults if environment variables aren't set.

2. **Create a function** that uses `${parameter?}` to validate required arguments and exit with a helpful error message if they're missing.

3. **Implement a file renaming tool** using `${var%pattern}` and `${var#pattern}` to batch-rename files in a directory.

4. **Write a script** that parses a URL using substring extraction and removal operators to extract protocol, domain, and path components.

5. **Create a variable name matcher** that uses `${!prefix*}` to find all variables starting with a given prefix and print their values.

6. **Build a configuration initializer** that uses `:=` to set uninitialized variables while ensuring they remain persistent within the script.

---

## Summary

- **Default values:** `${var:-default}`, `${var:=default}`, `${var:+alt}`, `${var?error}`
- **String length:** `${#var}` for character count, `${#array[@]}` for array size
- **Substring removal:**
  - From front: `${var#pattern}` (shortest), `${var##pattern}` (longest)
  - From back: `${var%pattern}` (shortest), `${var%%pattern}` (longest)
- **Substring operations:**
  - Extract: `${var:offset:length}`
  - Replace: `${var/pattern/repl}` (first), `${var//pattern/repl}` (all)
  - Prefix/suffix: `${var/#pattern/repl}`, `${var/%pattern/repl}`
- **Indirect references:** `${!varname}` for variable name indirection, `${!prefix*}` for name matching
- **Use `${var:-default}`** in functions to provide sensible defaults
- **Use `${var?error}`** to enforce required variables and provide clear error messages
- **Patterns use globbing:** `*` matches any characters, `?` matches single char
