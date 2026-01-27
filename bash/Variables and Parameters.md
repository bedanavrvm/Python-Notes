# Chapter 4: Introduction to Variables and Parameters

Variables are how programming and scripting languages represent data. A variable is nothing more than a label—a name assigned to a location or set of locations in computer memory holding an item of data. Variables appear in arithmetic operations, manipulation of quantities, and in string parsing.

## 4.1. Variable Substitution

The name of a variable is a placeholder for its value, the data it holds. Referencing (retrieving) its value is called **variable substitution**.

{% tabs %}
{% tab title="Key Concept" %}
Let us carefully distinguish between the **name** of a variable and its **value**:
- If `variable1` is the name of a variable, then `$variable1` is a reference to its value, the data it contains.

```bash
bash$ variable1=23
bash$ echo variable1
variable1
bash$ echo $variable1
23
```

The only times a variable appears "naked" (without the `$` prefix) is when:
- **Declared** or **assigned** (`variable1=value`)
- **Unset** (`unset variable1`)
- **Exported** (`export variable1`)
- In an **arithmetic expression** within double parentheses (`(( ... ))`)
- As a **signal variable** (see signal handling in advanced chapters)

{% endtab %}
{% tab title="Quoting Behavior" %}

**Partial Quoting (Double Quotes):** Enclosing a referenced value in double quotes (`"..."`) does not interfere with variable substitution.

```bash
hello="A B C D"
echo $hello      # A B C D (word-splits on whitespace)
echo "$hello"    # A B C D (preserves spaces)
```

**Full Quoting (Single Quotes):** Using single quotes (`'...'`) causes the variable name to be used literally; no substitution occurs.

```bash
echo '$hello'    # $hello (literal string, not substituted)
```

**Brace Notation:** `$variable` is a simplified form of `${variable}`. In contexts where `$variable` causes an error, the longer form `${variable}` may work.

{% endtab %}
{% endtabs %}

### Example 4-1: Variable Assignment and Substitution

```bash
#!/bin/bash
# Variables: assignment and substitution

a=375
hello=$a

# No space permitted on either side of = sign when initializing variables.
# "VARIABLE =value"  tries to run VARIABLE command with argument "=value"
# "VARIABLE= value"  tries to run value command with VARIABLE=""

echo hello          # hello (not a variable reference)
echo $hello         # 375 (variable reference)
echo ${hello}       # 375 (same, explicit brace notation)

# Quoting examples
echo "$hello"       # 375
echo "${hello}"     # 375
echo '$hello'       # $hello (literal, no substitution)

# Setting to null value
hello=
echo "\$hello (null) = $hello"  # $hello (null) =

# Multiple variables on one line
var1=21 var2=22 var3=$var3
echo "var1=$var1 var2=$var2 var3=$var3"

# Whitespace in variables requires quoting
numbers="one two three"
other_numbers="1 2 3"
echo "numbers = $numbers"
echo "other_numbers = $other_numbers"

# Escaping whitespace
mixed_bag=2\ ---\ Whatever
echo "$mixed_bag"   # 2 --- Whatever

# Uninitialized variables have null value
echo "uninitialized_variable = $uninitialized_variable"  # (empty)
uninitialized_variable=23
unset uninitialized_variable
echo "uninitialized_variable = $uninitialized_variable"  # (empty again)

exit 0
```

### Uninitialized Variables

An **uninitialized variable** has a "null" value — no assigned value at all (not zero!).

```bash
if [ -z "$unassigned" ]
then
  echo "\$unassigned is NULL."
fi
# Output: $unassigned is NULL.
```

**Arithmetic on Uninitialized Variables:** It is possible to perform arithmetic operations on an uninitialized variable, which evaluates as 0:

```bash
echo "$uninitialized"      # (blank line)
let "uninitialized += 5"   # Add 5 to it
echo "$uninitialized"      # 5
```

---

## 4.2. Variable Assignment

The **`=`** operator assigns a value to a variable. No space is permitted before or after the `=` sign.

{% tabs %}
{% tab title="Assignment Rules" %}

**Correct:**
```bash
a=879
let a=16+5
```

**Incorrect:**
```bash
a = 879      # Error: tries to run command 'a' with '=' as argument
a= 879       # Error: tries to run command '879' with a=""
a =879       # Error: tries to run command 'a' with '=879' as argument
```

Do not confuse `=` (assignment) with `-eq` (equality test):
- **Assignment:** `a=5` (sets a to 5)
- **Test:** `[ $a -eq 5 ]` (checks if a equals 5)

{% endtab %}
{% endtabs %}

### Example 4-2: Plain Variable Assignment

```bash
#!/bin/bash
# Naked variables: when is a variable used without the '$'?
# Answer: When it is being assigned, rather than referenced.

# Simple assignment
a=879
echo "The value of \"a\" is $a."

# Assignment using 'let'
let a=16+5
echo "The value of \"a\" is now $a."

# In a 'for' loop (a type of disguised assignment)
echo -n "Values of \"a\" in the loop are: "
for a in 7 8 9 11
do
  echo -n "$a "
done
echo

# In a 'read' statement (also a type of assignment)
echo -n "Enter \"a\": "
read a
echo "The value of \"a\" is now $a."

exit 0
```

### Example 4-3: Variable Assignment with Command Substitution

```bash
#!/bin/bash

a=23
echo $a         # 23

b=$a
echo $b         # 23

# Command substitution using backticks (older method)
a=`echo Hello!`
echo $a         # Hello!

a=`ls -l`
echo $a         # (removes tabs and newlines in unquoted form)
echo "$a"       # (preserves whitespace when quoted)

# Command substitution using $(...) (newer, preferred method)
R=$(cat /etc/redhat-release)
arch=$(uname -m)

exit 0
```

---

## 4.3. Bash Variables Are Untyped

Unlike many other programming languages, **Bash does not segregate its variables by "type."** Essentially, Bash variables are character strings, but depending on context, Bash permits arithmetic operations and comparisons on variables. The determining factor is whether the value contains only digits.

### Example 4-4: Integer or String?

```bash
#!/bin/bash
# int-or-string.sh

a=2334          # Integer
let "a += 1"
echo "a = $a"   # a = 2335 (still integer)

# String substitution transforms it
b=${a/23/BB}    # Substitute "BB" for "23"
echo "b = $b"   # b = BB35 (now a string!)

declare -i b    # Declaring as integer doesn't help
echo "b = $b"   # b = BB35 (still a string value)

let "b += 1"    # BB35 + 1
echo "b = $b"   # b = 1 (Bash sets integer value of non-numeric string to 0)

# Making a string into an integer
c=BB34
echo "c = $c"   # c = BB34

d=${c/BB/23}    # Substitute "23" for "BB"
echo "d = $d"   # d = 2334 (now numeric)

let "d += 1"
echo "d = $d"   # d = 2335

# Null variables in arithmetic
e=''            # Empty/null value
let "e += 1"
echo "e = $e"   # e = 1 (null becomes 0 in arithmetic)

# Undeclared variables in arithmetic
echo "f = $f"   # f = (empty)
let "f += 1"
echo "f = $f"   # f = 1 (undeclared also becomes 0)

# Division by zero still errors
# let "f /= 0"
# let: f /= 0: division by 0 (error token is "0")

exit $?
```

**Key Insight:** Variables in Bash are untyped, with all attendant consequences. This is both a blessing (flexibility) and a curse (easier to make subtle errors). To mitigate this, Bash does permit declaring variables with `declare -i` for integers, though this does not prevent string assignment.

---

## 4.4. Special Variable Types

### Local Variables

Variables visible only within a code block or function. These are crucial for avoiding naming conflicts in larger scripts and functions.

```bash
func() {
  local x=10        # Local to func()
  echo "Inside: x=$x"
}

x=5
func
echo "Outside: x=$x"  # x=5 (unaffected by local in function)
```

### Environmental Variables

Variables that affect the behavior of the shell and user interface.

**Definition:** Each process has an "environment"—a group of variables that the process may reference. Every time a shell starts, it creates shell variables that correspond to its own environmental variables.

**Export:** If a script sets environmental variables, they need to be "exported" (reported to the environment local to the script) using the `export` command.

```bash
export MY_VAR=value   # Available to child processes
bash -c 'echo $MY_VAR'  # Child process sees $MY_VAR
```

**Important:** A script can export variables only to child processes (commands the script initiates), not back to the parent shell or sibling processes.

### Positional Parameters

Arguments passed to the script from the command line: `$0`, `$1`, `$2`, `$3`, etc.

- **`$0`:** The name of the script itself
- **`$1`, `$2`, ..., `$9`:** The first through ninth arguments
- **`${10}`, `${11}`:** After `$9`, arguments must be enclosed in brackets
- **`$*` and `$@`:** Denote all positional parameters (with subtle differences in quoting)
- **`$#`:** The number of positional parameters

### Example 4-5: Positional Parameters

```bash
#!/bin/bash
# Call this script with at least 10 parameters, e.g.:
# ./scriptname 1 2 3 4 5 6 7 8 9 10

MINPARAMS=10

echo "The name of this script is \"$0\"."
echo "The name of this script is \"$(basename $0)\"."

if [ -n "$1" ]
then
  echo "Parameter #1 is $1"
fi

if [ -n "$2" ]
then
  echo "Parameter #2 is $2"
fi

# ... (check additional parameters)

if [ -n "${10}" ]
then
  echo "Parameter #10 is ${10}"
fi

echo "-----------------------------------"
echo "All the command-line parameters are: $*"

if [ $# -lt "$MINPARAMS" ]
then
  echo "This script needs at least $MINPARAMS command-line arguments!"
fi

exit 0
```

### Referencing the Last Argument

Bracket notation for positional parameters allows a simple way to reference the last argument:

```bash
args=$#
lastarg=${!args}
# Or, more concisely:
lastarg=${!#}
```

This uses **indirect referencing** to access the value of `$args` (or `$#`) as a variable name.

### Script Behavior Based on Invocation Name

Some scripts perform different operations depending on which name they are invoked with. To enable this, the script checks `$0` and symbolic links to alternate names may exist.

### Preventing Null Variable Issues

If a script expects a command-line parameter but is invoked without one, this may cause a null variable assignment (generally undesirable). One prevention strategy:

```bash
# Append a marker to both sides to ensure a safe comparison
if [ "${1:?Missing required argument!}" ]
then
  # Parameter was provided
  echo "Processing: $1"
fi
```

---

## Exercises

1. **Write a script** that takes three command-line arguments and displays them in reverse order.

2. **Declare a variable**, assign it a value, then demonstrate the difference between quoting with double quotes and single quotes.

3. **Create a function** with a local variable and a global variable; show how changes inside the function do not affect the global variable.

4. **Use command substitution** to capture the output of `date` into a variable and display it in a formatted message.

5. **Write a script** that checks if a required parameter is provided and displays an error message if it is missing.

---

## Summary

- Variables are labels for data stored in memory; use `$variable` to reference the value.
- Bash variables are untyped character strings; context determines arithmetic vs. string behavior.
- Use double quotes for partial quoting (preserves whitespace, allows substitution); use single quotes for full quoting (no substitution).
- Assignment uses `=` with no spaces around it.
- Special variables include local, environmental, and positional parameters.
- Positional parameters (`$0`, `$1`, ..., `$#`, `$*`, `$@`) provide access to script name and arguments.
