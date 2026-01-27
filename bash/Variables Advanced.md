# Chapter 9: Variables (Advanced)

Beyond basic variable assignment, Bash offers arrays, special variables, and techniques for generating random numbers. This chapter explores these advanced features.

## 9.1. Arrays

Arrays allow you to store multiple values in a single variable, indexed by position or key.

{% tabs %}
{% tab title="Indexed Arrays" %}

Indexed (numeric) arrays use integer indices starting at 0.

**Creating and populating:**
```bash
# Method 1: Individual assignment
array[0]="first"
array[1]="second"
array[2]="third"

# Method 2: Initializer list
array=("apple" "banana" "cherry")

# Method 3: Using declare
declare -a myarray
myarray[0]="element0"
myarray[1]="element1"
```

**Accessing elements:**
```bash
echo "${array[0]}"        # first
echo "${array[1]}"        # second
echo "${array[@]}"        # all elements
echo "${array[*]}"        # all elements (different quoting)
echo "${#array[@]}"       # number of elements
echo "${!array[@]}"       # indices
```

**Looping over arrays:**
```bash
for element in "${array[@]}"
do
  echo "$element"
done

for i in "${!array[@]}"
do
  echo "Index $i: ${array[$i]}"
done
```

{% endtab %}
{% tab title="Associative Arrays" %}

Associative arrays (hash maps) use string keys instead of integer indices. Requires `declare -A`.

**Creating and populating:**
```bash
declare -A person
person["name"]="Alice"
person["age"]="30"
person["city"]="New York"

# Or initialize all at once
declare -A colors=(["red"]="#FF0000" ["green"]="#00FF00" ["blue"]="#0000FF")
```

**Accessing elements:**
```bash
echo "${person["name"]}"     # Alice
echo "${person["age"]}"      # 30
echo "${person[@]}"          # all values
echo "${!person[@]}"         # all keys
```

**Looping over associative arrays:**
```bash
for key in "${!person[@]}"
do
  echo "$key => ${person[$key]}"
done
```

{% endtab %}
{% endtabs %}

### Example 9-1: Working with Indexed Arrays

```bash
#!/bin/bash
# array-demo.sh: Demonstrating indexed arrays

fruits=("apple" "banana" "cherry" "date" "elderberry")

echo "Fruits array:"
echo "Total fruits: ${#fruits[@]}"
echo

for i in "${!fruits[@]}"
do
  printf "  %d: %s\n" "$i" "${fruits[$i]}"
done

echo
echo "All fruits: ${fruits[@]}"
echo

# Add an element
fruits[5]="fig"
echo "After adding fig: ${fruits[@]}"

# Modify an element
fruits[1]="blueberry"
echo "After modifying index 1: ${fruits[@]}"
```

### Example 9-2: Slot Machine Simulation

```bash
#!/bin/bash
# slots.sh: Simulating a slot machine with array distribution

ROWS=23
COLS=21
RANGE=3        # Values 0, 1, 2 for left, stay, right
PASSES=500     # Number of marbles to drop

declare -a Slots
POS=0

Initialize_Slots() {
  for ((i=0; i<COLS; i++))
  do
    Slots[$i]=0
  done
}

Show_Slots() {
  # Display histogram
  echo
  echo "Slot Machine Results:"
  echo
  printf "    "
  for ((i=0; i<COLS; i++))
  do
    printf "%3d" $i
  done
  echo
  
  for ((i=0; i<ROWS; i++))
  do
    printf "%3d |" $((ROWS-i))
    for ((j=0; j<COLS; j++))
    do
      if [ ${Slots[$j]} -ge $((ROWS-i)) ]
      then
        printf " o "
      else
        printf "   "
      fi
    done
    echo "|"
  done
  
  echo "    |"
  for ((i=0; i<COLS; i++))
  do
    printf "___"
  done
  echo "__|"
  
  echo
  printf "    "
  for ((i=0; i<COLS; i++))
  do
    printf "%3d" ${Slots[$i]}
  done
  echo
}

Move() {
  # Random movement: left (-1), stay (0), right (+1)
  Move=$RANDOM
  let "Move %= RANGE"
  case "$Move" in
    0) ;;            # Stay put
    1) ((POS--));;   # Move left
    2) ((POS++));;   # Move right
  esac
}

Play() {
  # One pass (drop one marble)
  for ((i=0; i<ROWS; i++))
  do
    Move
  done
  
  SHIFT=11
  let "POS += $SHIFT"
  (( Slots[$POS]++ ))
}

Run() {
  # Main loop: drop multiple marbles
  for ((p=0; p<PASSES; p++))
  do
    Play
    POS=0
  done
}

# Main execution
Initialize_Slots
Run
Show_Slots

exit 0
```

---

## 9.2. Special Variables and $RANDOM

Bash provides several special variables for system information and randomness.

{% tabs %}
{% tab title="Special Variables" %}

| Variable | Meaning |
|----------|---------|
| `$0` | Script name |
| `$1, $2, ...` | Positional parameters |
| `$#` | Number of positional parameters |
| `$*` | All positional parameters (as one word) |
| `$@` | All positional parameters (as separate words) |
| `$?` | Exit status of last command |
| `$$` | Process ID (PID) of script |
| `$!` | Process ID of last background job |
| `$RANDOM` | Random integer between 0 and 32767 |
| `$HOSTNAME` | System hostname |
| `$USER` | Current user |
| `$HOME` | Home directory |
| `$PWD` | Current working directory |
| `$IFS` | Internal Field Separator |

{% endtab %}
{% tab title="The $RANDOM Variable" %}

`$RANDOM` is a built-in Bash variable that generates a random integer between 0 and 32767 (inclusive).

**Basic usage:**
```bash
# Simple random number
echo $RANDOM           # e.g., 27486

# Random number in range 0-9
echo $((RANDOM % 10))  # e.g., 7

# Random number in range 1-6 (like a die)
echo $((RANDOM % 6 + 1))  # e.g., 4

# Random number in range 10-20
echo $((RANDOM % 11 + 10))
```

**Key facts:**
- `$RANDOM` generates a **pseudorandom** number (not truly random)
- Returns values between 0 and 32767 (15-bit range)
- Each invocation produces a different value
- Can be seeded for reproducibility: `RANDOM=seed`
- For better randomness, use `/dev/urandom`

{% endtab %}
{% endtabs %}

---

## 9.3. Generating Random Numbers in a Range

Several techniques exist for generating random numbers within specific ranges and constraints.

### Basic Range Generation

```bash
# Random number between 0 and N-1
rnumber=$((RANDOM % N))

# Random number between 1 and N
rnumber=$((RANDOM % N + 1))

# Random number between min and max
rnumber=$((RANDOM % (max - min + 1) + min))
```

### Example 9-14: Random Between Values

```bash
#!/bin/bash
# random-between.sh: Generate random numbers in a range
# with divisibility constraint

randomBetween() {
  # Generates a random number between $min and $max
  # divisible by $divisibleBy
  
  local min=${1:-0}
  local max=${2:-32767}
  local divisibleBy=${3:-1}
  
  # Ensure divisibleBy is positive
  [ ${divisibleBy} -lt 0 ] && divisibleBy=$((0 - divisibleBy))
  
  # Sanity checks
  if [ $# -gt 3 ] || [ ${divisibleBy} -eq 0 ] || [ ${min} -eq ${max} ]
  then
    echo "Usage: randomBetween [min] [max] [divisibleBy]"
    return 1
  fi
  
  # Swap if min > max
  if [ ${min} -gt ${max} ]
  then
    local x=${min}
    min=${max}
    max=${x}
  fi
  
  # Adjust min if not divisible by $divisibleBy
  if [ $((min / divisibleBy * divisibleBy)) -ne ${min} ]
  then
    if [ ${min} -lt 0 ]
    then
      min=$((min / divisibleBy * divisibleBy))
    else
      min=$(((min / divisibleBy + 1) * divisibleBy))
    fi
  fi
  
  # Adjust max similarly
  if [ $((max / divisibleBy * divisibleBy)) -ne ${max} ]
  then
    if [ ${max} -lt 0 ]
    then
      max=$((((max / divisibleBy) - 1) * divisibleBy))
    else
      max=$((max / divisibleBy * divisibleBy))
    fi
  fi
  
  # Generate random number
  local spread=$((max - min + divisibleBy))
  randomBetweenAnswer=$(((RANDOM % spread) / divisibleBy * divisibleBy + min))
  
  return 0
}

# Test the function
randomBetween -14 20 3
echo "Random number between -14 and 20, divisible by 3: $randomBetweenAnswer"

randomBetween 1 100
echo "Random number between 1 and 100: $randomBetweenAnswer"

randomBetween 0 9
echo "Random digit: $randomBetweenAnswer"
```

### Techniques for Better Randomness

**Using `/dev/urandom`:**
```bash
# Generate a random byte and convert to decimal
rnumber=$(od -An -td1 -N1 < /dev/urandom | tr -d ' ')

# Or using hexdump
rnumber=$(hexdump -n 1 -e '%d' /dev/urandom)
```

**Using `awk` for floating-point randomness:**
```bash
# Random number between 0 and 1 (with decimal places)
awk 'BEGIN { srand(); print rand() }'

# Random integer in range
awk "BEGIN { srand(); print int(rand() * 100) }"
```

**Using `date` for sequence-based randomness:**
```bash
# Use current time as seed
rnumber=$(($(date +%s%N) % 100))
```

---

## 9.4. Testing Random Distribution

To verify that `$RANDOM` produces a reasonable distribution, write a script that tracks the frequency of outcomes.

### Example 9-15: Rolling a Die with $RANDOM

```bash
#!/bin/bash
# dice-roll.sh: Testing randomness with die rolls

RANDOM=$$           # Seed using script's process ID
PIPS=6              # A die has 6 faces
MAXTHROWS=600       # Number of throws

throw=0
ones=0 twos=0 threes=0 fours=0 fives=0 sixes=0

while [ "$throw" -lt "$MAXTHROWS" ]
do
  let "die = RANDOM % PIPS"
  case "$die" in
    0) ((ones++));;
    1) ((twos++));;
    2) ((threes++));;
    3) ((fours++));;
    4) ((fives++));;
    5) ((sixes++));;
  esac
  let "throw += 1"
done

echo "Results of $MAXTHROWS throws:"
echo "ones   = $ones"
echo "twos   = $twos"
echo "threes = $threes"
echo "fours  = $fours"
echo "fives  = $fives"
echo "sixes  = $sixes"
echo
echo "(Expected ~100 of each for even distribution)"

exit 0
```

**Expected output:** With 600 throws, each face should appear approximately 100 times (±20 or so).

---

## 9.5. Reseeding $RANDOM

Setting `$RANDOM` to a specific value reseeds the pseudorandom generator, allowing reproducible sequences.

### Example 9-16: Seeding $RANDOM

```bash
#!/bin/bash
# seeding-random.sh: Demonstrating RANDOM reseeding

MAXCOUNT=10

echo "Seed = 1:"
RANDOM=1
for ((i=0; i<MAXCOUNT; i++))
do
  echo -n "$RANDOM "
done
echo

echo
echo "Seed = 1 again (should be identical):"
RANDOM=1
for ((i=0; i<MAXCOUNT; i++))
do
  echo -n "$RANDOM "
done
echo

echo
echo "Seed = 2 (different sequence):"
RANDOM=2
for ((i=0; i<MAXCOUNT; i++))
do
  echo -n "$RANDOM "
done
echo

echo
echo "Seed from process ID:"
RANDOM=$$
for ((i=0; i<MAXCOUNT; i++))
do
  echo -n "$RANDOM "
done
echo

echo
echo "Seed from /dev/urandom:"
SEED=$(od -N 1 -tu1 /dev/urandom | awk '{print $2}')
RANDOM=$SEED
for ((i=0; i<MAXCOUNT; i++))
do
  echo -n "$RANDOM "
done
echo

exit 0
```

**Key insight:** Same seed always produces the same sequence, useful for reproducible testing.

---

## 9.6. Alternative Methods for Random Numbers

### Using `awk`

```bash
#!/bin/bash
# awk-random.sh: Generate random numbers with awk

echo "Random number between 0 and 1 (6 decimals):"
echo | awk '{ srand(); print rand() }'

echo
echo "10 random integers between 1 and 100:"
awk 'BEGIN {
  srand()
  for (i=1; i<=10; i++)
    print int(rand() * 100) + 1
}'

echo
echo "Random float between 10 and 20:"
awk 'BEGIN {
  srand()
  print 10 + rand() * 10
}'
```

### Using Date for Seeding

```bash
# Use nanoseconds from date
seed=$(date +%s%N)
rnumber=$((seed % 100))

# Or use just seconds
seed=$(date +%s)
rnumber=$((seed % 100))
```

---

## 9.7. Array Best Practices

{% tabs %}
{% tab title="Indexed Arrays" %}

- Always quote array subscripts: `"${array[$i]}"` not `${array[$i]}`
- Use `[@]` to expand all elements: `"${array[@]}"`
- Use `[*]` only when you want word-splitting
- Initialize arrays explicitly: `array=()`
- Check array length with `${#array[@]}`

{% endtab %}
{% tab title="Associative Arrays" %}

- Declare with `declare -A` before use
- Keys can contain any string (spaces, special characters)
- Access with quoted keys: `"${array["key name"]}"`
- Iterate over keys with `"${!array[@]}"`
- No guaranteed order of iteration

{% endtab %}
{% endtabs %}

---

## Exercises

1. **Create an indexed array** of your favorite programming languages and iterate over it, printing each language with its index.

2. **Build an associative array** representing a student's grades (subjects as keys, grades as values) and calculate the average.

3. **Write a script** that rolls two dice 1000 times and displays the frequency distribution of sums (2-12).

4. **Generate random numbers** in the range 50-100, divisible by 5, using the formula approach.

5. **Implement a simple lottery picker** that generates N random numbers from 1 to 50 without repetition (using an array to track picks).

6. **Create a function** that accepts min, max, and divisor, then returns a random number meeting those constraints (like `randomBetween`).

---

## Summary

- **Indexed arrays:** Use integer indices; access with `${array[i]}`; iterate with `for i in "${!array[@]}"`
- **Associative arrays:** Require `declare -A`; use string keys; access with `${array["key"]}`
- **Special variables:** `$RANDOM`, `$$`, `$?`, `$!`, `$HOME`, `$PWD`, etc.
- **`$RANDOM` range:** 0 to 32767 (15-bit pseudorandom)
- **Generate random in range:** `$((RANDOM % (max - min + 1) + min))`
- **Reseeding:** `RANDOM=seed` produces reproducible sequences
- **Better randomness:** Use `/dev/urandom`, `awk`, or `date` for higher-quality randomness
- **Always quote arrays:** `"${array[@]}"` and `"${array[$i]}"` to prevent word-splitting
