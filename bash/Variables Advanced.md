# Chapter 7: Variables (Advanced)

Beyond basic variable assignment, Bash offers arrays, special variables, and techniques for generating random numbers. This chapter explores advanced variable concepts that enable sophisticated scripting patterns.

**Arrays** are the primary data structure for managing collections of values. **Special variables** give access to system information and randomness. Together, these features enable real-world script functionality.

---

## 7.1. Understanding Arrays

An **array** is a data structure that stores multiple values under a single name, each accessed by an index. Bash supports two types: **indexed arrays** (integer-based) and **associative arrays** (string-based).

### Why Arrays Matter

```bash
# Without arrays (poor approach)
fruit1="apple"
fruit2="banana"
fruit3="cherry"
echo "$fruit1"
echo "$fruit2"
echo "$fruit3"

# With arrays (scalable approach)
fruits=("apple" "banana" "cherry")
for fruit in "${fruits[@]}"; do
  echo "$fruit"
done
```

The array version scales easily to 100 or 1000 items without code duplication.

---

## 7.2. Indexed Arrays

Indexed arrays use integer indices (0, 1, 2, ...) to access elements. This is the most common array type in Bash.

### Creating Indexed Arrays

**Method 1: Individual element assignment**

```bash
array[0]="first"
array[1]="second"
array[2]="third"
```

**Method 2: Array literal (initializer list)**

```bash
array=("apple" "banana" "cherry")
```

**Method 3: Using `declare`**

```bash
declare -a myarray
myarray[0]="element0"
myarray[1]="element1"
```

### Accessing Array Elements

| Syntax | Meaning |
|--------|---------|
| `${array[0]}` | First element |
| `${array[1]}` | Second element |
| `${array[@]}` | All elements (preferred) |
| `${array[*]}` | All elements (same as above with proper quoting) |
| `${#array[@]}` | Number of elements |
| `${!array[@]}` | Array indices/keys |

**Examples:**

```bash
#!/bin/bash

fruits=("apple" "banana" "cherry" "date" "elderberry")

# Access single element
echo "First fruit: ${fruits[0]}"     # apple
echo "Third fruit: ${fruits[2]}"     # cherry

# Get array length
echo "Total fruits: ${#fruits[@]}"   # 5

# Get all elements
echo "All fruits: ${fruits[@]}"

# Get indices
echo "Indices: ${!fruits[@]}"        # 0 1 2 3 4
```

### Looping Over Indexed Arrays

**Method 1: Loop over values**

```bash
for fruit in "${fruits[@]}"; do
  echo "$fruit"
done
```

**Method 2: Loop over indices**

```bash
for i in "${!fruits[@]}"; do
  echo "Index $i: ${fruits[$i]}"
done
```

**Method 3: C-style for loop**

```bash
for ((i = 0; i < ${#fruits[@]}; i++)); do
  echo "[$i] = ${fruits[$i]}"
done
```

### Modifying Arrays

```bash
#!/bin/bash

fruits=("apple" "banana")

# Add element (appends at next index)
fruits[2]="cherry"
echo "${fruits[@]}"                 # apple banana cherry

# Modify existing element
fruits[1]="blueberry"
echo "${fruits[@]}"                 # apple blueberry cherry

# Append using += (Bash 3.1+)
fruits+=("date")
echo "${fruits[@]}"                 # apple blueberry cherry date

# Get number of elements
echo "Count: ${#fruits[@]}"          # 4
```

### Example 7-1: Managing a List with Indexed Arrays

```bash
#!/bin/bash
# task-list.sh - Simple task management with arrays

tasks=()

add_task() {
  local task="$1"
  tasks+=("$task")
  echo "Added: $task"
}

list_tasks() {
  echo "Tasks:"
  for i in "${!tasks[@]}"; do
    echo "  $((i + 1)). ${tasks[$i]}"
  done
}

remove_task() {
  local index=$1
  if [ $index -ge 1 ] && [ $index -le ${#tasks[@]} ]; then
    echo "Removed: ${tasks[$((index - 1))]}"
    unset 'tasks[$((index - 1))]'
    # Reindex array
    tasks=("${tasks[@]}")
  fi
}

# Main
add_task "Buy groceries"
add_task "Fix the leak"
add_task "Call dentist"

list_tasks

remove_task 2
echo "After removing task 2:"
list_tasks
```

---

## 7.3. Associative Arrays

Associative arrays (also called **hash maps** or **dictionaries**) use string keys instead of integer indices. Requires Bash 4.0+ and explicit declaration.

### Declaring Associative Arrays

```bash
# Declaration with declare -A
declare -A person
person["name"]="Alice"
person["age"]="30"
person["city"]="New York"

# Or initialize all at once
declare -A colors=(
  ["red"]="#FF0000"
  ["green"]="#00FF00"
  ["blue"]="#0000FF"
)
```

**Important:** You must declare associative arrays before use with `declare -A`. Indexed arrays don't require this.

### Accessing Associative Array Elements

| Syntax | Meaning |
|--------|---------|
| `${array["key"]}` | Value for specific key |
| `${array[@]}` | All values |
| `${!array[@]}` | All keys |
| `${#array[@]}` | Number of key-value pairs |

**Examples:**

```bash
#!/bin/bash

declare -A person=(
  ["name"]="Alice"
  ["age"]="30"
  ["city"]="New York"
)

# Access specific value
echo "Name: ${person["name"]}"       # Alice
echo "Age: ${person["age"]}"         # 30

# Get all values
echo "Values: ${person[@]}"

# Get all keys
echo "Keys: ${!person[@]}"

# Get count
echo "Count: ${#person[@]}"          # 3
```

### Looping Over Associative Arrays

```bash
#!/bin/bash

declare -A settings=(
  ["theme"]="dark"
  ["font_size"]="12"
  ["language"]="en"
)

# Loop over key-value pairs
echo "Settings:"
for key in "${!settings[@]}"; do
  echo "  $key = ${settings[$key]}"
done
```

### When to Use Associative Arrays

```bash
# Good use case: representing an object/record
declare -A user=(
  ["id"]="12345"
  ["username"]="alice"
  ["email"]="alice@example.com"
  ["created"]="2026-01-28"
)

# Good use case: configuration/settings
declare -A config=(
  ["db_host"]="localhost"
  ["db_port"]="5432"
  ["db_user"]="admin"
)

# Avoid: when order matters
# (Associative arrays don't maintain insertion order)
```

### Example 7-2: Storing Configuration Data

```bash
#!/bin/bash
# config.sh - Configuration management with associative arrays

# Define configuration
declare -A db=(
  ["host"]="localhost"
  ["port"]="5432"
  ["user"]="dbadmin"
  ["password"]="secret123"
  ["name"]="myapp"
)

get_connection_string() {
  echo "postgresql://${db["user"]}@${db["host"]}:${db["port"]}/${db["name"]}"
}

test_connection() {
  local conn=$(get_connection_string)
  echo "Testing connection: $conn"
  # In real script, would actually test connection
}

print_settings() {
  echo "Database Configuration:"
  for key in "${!db[@]}"; do
    if [ "$key" != "password" ]; then
      echo "  $key: ${db[$key]}"
    else
      echo "  $key: ****"
    fi
  done
}

# Usage
print_settings
echo
test_connection
```

---

## 7.4. Special Variables

### Essential Special Variables

| Variable | Purpose | Example |
|----------|---------|---------|
| `$0` | Script name | `./myscript.sh` |
| `$1`, `$2`, ... | Positional parameters | Function/script arguments |
| `$#` | Number of parameters | `for (( i = 1; i <= $#; i++ ))` |
| `$$` | Process ID (PID) | `temp_${$}.txt` for unique files |
| `$?` | Exit code of last command | `if [ $? -eq 0 ]` |
| `$!` | PID of last background process | `kill $!` to terminate |
| `$RANDOM` | Random integer (0-32767) | `$((RANDOM % 6 + 1))` for die roll |
| `$HOSTNAME` | System hostname | For network operations |
| `$USER` | Current username | For logging/permissions |
| `$HOME` | Home directory path | For config file locations |
| `$PWD` | Current working directory | `cd "$HOME"; echo "$PWD"` |
| `$IFS` | Internal Field Separator | Controls word splitting |

### Using Process ID for Uniqueness

```bash
#!/bin/bash

# Create unique temporary files
temp_file="/tmp/data_$$.txt"
echo "Creating temp file: $temp_file"

# Combine with timestamp for extra uniqueness
timestamp=$(date +%s)
unique_name="backup_$${timestamp}_$$.tar.gz"
echo "Backup filename: $unique_name"
```

### Using $? for Error Checking

```bash
#!/bin/bash

# Check if command succeeded
grep "pattern" file.txt
if [ $? -eq 0 ]; then
  echo "Pattern found"
else
  echo "Pattern not found"
fi

# More concise
grep "pattern" file.txt && echo "Found" || echo "Not found"
```

---

## 7.5. The $RANDOM Variable

`$RANDOM` is a built-in Bash variable that generates a **pseudorandom** integer between 0 and 32767 (15-bit range, inclusive).

### Understanding $RANDOM

```bash
#!/bin/bash

# Basic usage
echo "Random number: $RANDOM"

# Each invocation is different
echo "$RANDOM"
echo "$RANDOM"
echo "$RANDOM"
# Output: three different values
```

### Generating Random Numbers in Ranges

**Formula: Random between min and max (inclusive)**

```bash
# Random between 0 and 9
(( random_digit = RANDOM % 10 ))

# Random between 1 and 6 (die roll)
(( die_roll = RANDOM % 6 + 1 ))

# Random between min and max
min=10
max=20
(( random_value = RANDOM % (max - min + 1) + min ))
```

**Breaking down the formula:**
- `RANDOM % (max - min + 1)` — Get random value in range [0, max - min]
- `+ min` — Shift range to [min, max]

### Practical Examples

```bash
#!/bin/bash

# Coin flip (0 or 1)
(( coin = RANDOM % 2 ))
[ "$coin" -eq 0 ] && echo "Heads" || echo "Tails"

# Card from standard deck (1-52)
(( card = RANDOM % 52 + 1 ))
echo "Card number: $card"

# Password character (ASCII 33-126, printable)
(( ascii = RANDOM % 94 + 33 ))
printf "Character: \\$(printf '%03o' $ascii)\n"
```

### Example 7-3: Rolling a Die with Distribution Testing

```bash
#!/bin/bash
# dice-stats.sh - Roll a die N times and show distribution

ROLLS=6000
declare -a counts=(0 0 0 0 0 0)

for ((i = 0; i < ROLLS; i++)); do
  (( roll = RANDOM % 6 ))
  (( counts[$roll]++ ))
done

echo "Die roll distribution ($ROLLS rolls):"
for face in 0 1 2 3 4 5; do
  percentage=$(( (counts[$face] * 100) / ROLLS ))
  echo "  Face $((face + 1)): ${counts[$face]} times (~${percentage}%)"
done

echo
echo "Expected: ~1000 of each for fair distribution"
```

### Important Facts About $RANDOM

**Pseudorandom, not random:**
- Uses a predictable algorithm (Linear Congruential Generator)
- Suitable for games and simulations
- **Not suitable** for cryptography or security

**Seeding for reproducibility:**

```bash
# Seed with a value for reproducible sequences
RANDOM=1
echo "$RANDOM"     # Always same first value with seed=1
echo "$RANDOM"     # Always same second value

# Reset seed
RANDOM=1
echo "$RANDOM"     # Same as before
```

**For better randomness:**

```bash
# Use /dev/urandom (hardware randomness)
random_byte=$(od -An -td1 -N1 < /dev/urandom | tr -d ' ')

# Use /dev/random (blocking, higher quality)
random_byte=$(od -An -td1 -N1 < /dev/random | tr -d ' ')

# Use nanoseconds from date
(( random_seed = $(date +%s%N) % 100 ))
```

---

## 7.6. Generating Random Numbers With Constraints

### Divisibility Constraint

Sometimes you need a random number divisible by a specific value:

```bash
#!/bin/bash

# Random multiple of 5, between 0 and 100
divisor=5
max=100
(( random_num = (RANDOM % ((max / divisor) + 1)) * divisor ))
echo "Random (multiple of 5): $random_num"
```

### Example 7-4: Random Number With Multiple Constraints

```bash
#!/bin/bash
# random-constrained.sh

random_in_range() {
  local min=$1
  local max=$2
  local divisor=${3:-1}
  
  # Validate inputs
  [ $min -gt $max ] && { echo "Error: min > max"; return 1; }
  [ $divisor -eq 0 ] && { echo "Error: divisor is 0"; return 1; }
  
  # Calculate range accounting for divisor
  local count=$(( (max - min) / divisor + 1 ))
  local random=$(( RANDOM % count ))
  local result=$(( random * divisor + min ))
  
  echo $result
}

# Generate random numbers
echo "Random 0-50: $(random_in_range 0 50)"
echo "Random 1-100, multiple of 10: $(random_in_range 10 100 10)"
echo "Random 5-25, multiple of 5: $(random_in_range 5 25 5)"
```

---

## 7.7. Best Practices for Arrays

### For Indexed Arrays

```bash
# ✓ Correct: Always quote array subscripts
for i in "${!array[@]}"; do
  echo "${array[$i]}"
done

# ✓ Correct: Use [@] for all elements
for item in "${array[@]}"; do
  echo "$item"
done

# ✗ Avoid: Unquoted arrays (word-splitting)
for item in $array; do   # WRONG!
  echo "$item"
done

# ✓ Correct: Check length before accessing
if [ ${#array[@]} -gt 0 ]; then
  echo "First: ${array[0]}"
fi
```

### For Associative Arrays

```bash
# ✓ Correct: Declare before use
declare -A config
config["key"]="value"

# ✓ Correct: Quote keys with spaces
declare -A person=(
  ["first name"]="Alice"
  ["last name"]="Bob"
)
echo "${person["first name"]}"

# ✗ Wrong: Missing declare -A
config["key"]="value"  # Creates indexed array, not associative!

# ✓ Correct: Iterate over keys
for key in "${!config[@]}"; do
  echo "$key: ${config[$key]}"
done
```

### Common Pitfalls

```bash
# ✗ Pitfall 1: Forgetting declare -A for associative arrays
assoc["key"]="value"    # Actually creates indexed array!

# ✗ Pitfall 2: Unquoted array expansion
for item in ${array[@]}; do    # Word-splitting occurs!
  echo "$item"
done

# ✓ Fix: Quote array expansion
for item in "${array[@]}"; do
  echo "$item"
done

# ✗ Pitfall 3: Assuming order in associative arrays
# Associative arrays don't maintain insertion order!
declare -A items=(["a"]=1 ["b"]=2 ["c"]=3)
for key in "${!items[@]}"; do
  echo "$key"
done
# Order is undefined!
```

---

## Programming Keywords and Concepts

### Essential Array Keywords

| Keyword | Type | Purpose |
|---|---|---|
| `declare -a` | Declaration | Create indexed array |
| `declare -A` | Declaration | Create associative array |
| `${array[@]}` | Expansion | All elements |
| `${#array[@]}` | Expansion | Array length |
| `${!array[@]}` | Expansion | Keys/indices |
| `+=` | Operator | Append to array |

### Core Programming Concepts

**1. Data Structures**
- Arrays are collections of values under one name
- Indexed arrays for ordered collections
- Associative arrays for key-value pairs (objects/records)
- Enables managing related data together

**2. Array Indexing**
- Indexed arrays: 0-based integer indices
- Associative arrays: string-based keys
- Access via bracket notation: `${array[key]}`
- Important: Always quote indices to prevent word-splitting

**3. Array Iteration Patterns**
- Over values: `for item in "${array[@]}"`
- Over indices: `for i in "${!array[@]}"`
- C-style: `for ((i = 0; i < ${#array[@]}; i++))`
- Choose based on what you need (values or positions)

**4. Array Modification**
- Individual element: `array[5]="value"`
- Append: `array+=("new element")`
- Unset: `unset 'array[index]'`
- Reindex: `array=("${array[@]}")`

**5. Special Variables Access**
- Provide access to system state without external commands
- `$$` for process-specific filenames
- `$?` for error checking
- `$RANDOM` for non-cryptographic randomness
- Essential for defensive programming

**6. Pseudorandom Number Generation**
- `$RANDOM` generates 15-bit values (0-32767)
- Predictable and reproducible with seeding
- Suitable for games, simulations, sampling
- Not suitable for security/cryptography

**7. Randomness Formulas**
- Range [0, N): `RANDOM % N`
- Range [min, max]: `RANDOM % (max - min + 1) + min`
- Multiples: `(RANDOM / step) * step`
- Complex constraints require custom logic

**8. Associative vs. Indexed Arrays**
- Choose based on data structure needs
- Associative when semantic meaning in keys
- Indexed when order matters
- Bash 4.0+ required for associative arrays

**9. Quote Protection in Arrays**
- Quoting prevents word-splitting: `"${array[@]}"`
- Unquoted expands with IFS: `${array[@]}`
- Critical when filenames have spaces
- Habit: always quote array variables

**10. Process ID Usage**
- `$$` for unique filenames: `file_$$.tmp`
- Combine with timestamp for extra uniqueness
- Prevents conflicts in concurrent scripts
- Essential for temporary file management

---

## Common Array Patterns

### Pattern 1: Accumulating Values

```bash
results=()
for file in *.txt; do
  # Process file
  status=$(process "$file")
  results+=("$status")
done

# Display all results
for result in "${results[@]}"; do
  echo "$result"
done
```

### Pattern 2: Lookup Table (Associative Array)

```bash
declare -A status_codes=(
  ["success"]="0"
  ["error"]="1"
  ["not_found"]="2"
  ["timeout"]="3"
)

handle_result() {
  local code=$1
  local msg="${status_codes[$code]}"
  echo "Result: $msg"
}
```

### Pattern 3: Counting Occurrences

```bash
declare -a frequencies
for item in "${items[@]}"; do
  (( frequencies[$item]++ ))
done

for value in "${!frequencies[@]}"; do
  echo "$value: ${frequencies[$value]} times"
done
```

---

## Summary

Advanced variables provide powerful data management:

- **Indexed arrays** store ordered collections; access with `${array[i]}`
- **Associative arrays** store key-value pairs; require `declare -A`
- **Always quote** array expansions: `"${array[@]}"`
- **Special variables** provide system information: `$$`, `$?`, `$HOME`, etc.
- **`$RANDOM`** generates pseudorandom integers (0-32767)
- **Randomness formulas** enable generating constrained random numbers
- **Array best practices** prevent common mistakes with quoting and initialization
- **Choose the right structure** — indexed for sequences, associative for objects

Master arrays and special variables, and you'll write more sophisticated and maintainable Bash scripts.
