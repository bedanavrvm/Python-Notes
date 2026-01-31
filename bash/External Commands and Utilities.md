# Chapter 11: External Commands and Utilities

## Understanding External Commands in Shell Scripts

External commands and utilities extend Bash's capabilities far beyond what builtins provide. They're specialized programs designed for data processing, file manipulation, binary handling, and system-level operations.

**Key distinction:**
- **Builtins** (Chapter 10): Execute within the shell process, no forking
- **External commands** (Chapter 11): Separate executables spawned by the shell, enable specialized functionality

Understanding external commands means understanding **data transformation**—converting between formats, extracting information, and processing binary data. These are fundamental skills in Unix philosophy: small, focused programs that do one thing well.

### Why External Commands Matter

Many tasks are **impossible** with builtins alone:
- Binary data processing (`od`, `hexdump`, `dd`)
- Unit conversions (`units`)
- Data compression/decompression (`gzip`, `tar`)
- Audio processing (`sox`)
- Macro expansion (`m4`)

External commands are where Bash's power truly lies—the ability to compose them into data pipelines.

---

## 11.1. dd: Data Duplication and Binary Manipulation

### Understanding dd Semantics

The `dd` command is one of the most misunderstood Unix utilities. Its unusual syntax (`if=`, `of=`, `bs=`) reflects its origin as a **data converter** between different systems.

**Core concept:** `dd` reads input in blocks, applies optional conversions, and writes output in blocks. This is useful for:
- Binary data manipulation
- Low-level disk operations
- Data format conversions
- Random access to data streams
- Character extraction and transformation

### Syntax and Key Options

```bash
dd if=INFILE of=OUTFILE bs=BLOCKSIZE skip=BLOCKS seek=BLOCKS count=BLOCKS conv=CONVERSION
```

**Essential options:**

| Option | Purpose | Example |
|--------|---------|---------|
| `if=FILE` | Input file (default: stdin) | `if=/dev/urandom` |
| `of=FILE` | Output file (default: stdout) | `of=output.bin` |
| `bs=SIZE` | Block size in bytes | `bs=4096` (typical) |
| `skip=N` | Skip N blocks from input | `skip=10` |
| `seek=N` | Skip N blocks in output | `seek=5` |
| `count=N` | Copy N blocks (omit for all) | `count=100` |
| `conv=TYPE` | Data conversion | `conv=ucase` |

**Block size considerations:**
- Larger block sizes (4096, 8192) = faster for large files
- Smaller block sizes (1) = byte-level precision for extractions
- Mismatch between input/output block sizes creates padding

### Common Conversions

| Conversion | Effect |
|-----------|--------|
| `ucase` | Convert lowercase to uppercase |
| `lcase` | Convert uppercase to lowercase |
| `notrunc` | Don't truncate output file (preserve existing content) |
| `unblock` | Convert fixed-length records to variable-length (add newlines) |
| `block` | Opposite of unblock |
| `swab` | Swap bytes (for byte-order conversion) |

### Example 11-1 (Enhanced): Converting File to Uppercase

```bash
#!/bin/bash
# uppercase.sh: Convert text file to uppercase using dd

if [ $# -ne 1 ]; then
  echo "Usage: $0 filename"
  exit 1
fi

input_file="$1"
output_file="${input_file%.txt}.uppercase"

if [ ! -f "$input_file" ]; then
  echo "Error: File not found: $input_file"
  exit 1
fi

# dd converts text to uppercase during copy
dd if="$input_file" of="$output_file" conv=ucase 2>/dev/null

echo "✓ Converted: $input_file → $output_file"
ls -lh "$output_file"

exit 0
```

### Example 11-2 (Enhanced): Character Extraction

```bash
#!/bin/bash
# extract-chars.sh: Extract substring by byte position using dd
# Usage: extract-chars.sh filename start_position end_position

if [ $# -ne 3 ]; then
  echo "Usage: $0 filename start_pos end_pos"
  echo "  start_pos: First byte position (1-indexed)"
  echo "  end_pos: Last byte position (inclusive)"
  exit 1
fi

infile="$1"
start_pos="$2"
end_pos="$3"

if [ ! -f "$infile" ]; then
  echo "Error: File not found"
  exit 1
fi

# dd operations:
# skip=$((start_pos - 1)): Skip to starting position (convert to 0-indexed)
# count=$((end_pos - start_pos + 1)): Number of bytes to copy
length=$((end_pos - start_pos + 1))

echo "Extracting bytes $start_pos to $end_pos from $infile:"
echo "---"

dd if="$infile" of=/dev/stdout bs=1 skip=$((start_pos - 1)) count=$length 2>/dev/null

echo
echo "---"
exit 0
```

**Example usage:**
```bash
$ echo "Hello World" | ./extract-chars.sh /dev/stdin 1 5
Hello
```

### Example 11-3: Vertical Text Output

```bash
#!/bin/bash
# vertical.sh: Display text vertically using dd

if [ $# -ne 1 ]; then
  echo "Usage: $0 'text_string'"
  exit 1
fi

text="$1"

echo "Text vertically:"
echo "$text" | dd cbs=1 conv=unblock 2>/dev/null

exit 0
```

**How it works:**
- `cbs=1`: Set conversion block size to 1 character
- `conv=unblock`: Convert fixed-length records to variable-length (adds newline after each "record")
- Result: Each character on separate line

### Example 11-4: Random Access Write

```bash
#!/bin/bash
# random-access.sh: Write at specific position without truncating

outfile="test.dat"

# Create initial file
echo "AAAAAAAAAA" > "$outfile"
echo "Original: $(cat $outfile)"

# Write 'X' at position 5 (0-indexed) without truncating
echo -n "X" | dd of="$outfile" bs=1 seek=4 conv=notrunc 2>/dev/null

echo "Modified: $(cat $outfile)"

# Key insight: notrunc preserves the rest of file
# Result: XXXXAAAAAA (position 4 overwrites with X)

exit 0
```

### Example 11-5 (Enhanced): Capturing Keystrokes Without ENTER

```bash
#!/bin/bash
# keypress.sh: Read keypresses without waiting for ENTER
# Useful for menu systems without requiring Enter key

keypresses=3
echo "Press exactly $keypresses keys (no need to press Enter)"

# Save current terminal settings
old_tty_setting=$(stty -g)

# Disable canonical mode (-icanon) and echo (-echo)
stty -icanon -echo

# Read keystrokes directly
keys=$(dd bs=1 count=$keypresses 2>/dev/null)

# Restore terminal settings
stty "$old_tty_setting"

echo
echo "You pressed: "$keys""
echo "Keypress analysis:"
for ((i = 0; i < ${#keys}; i++)); do
  char="${keys:i:1}"
  ascii=$(printf '%d' "'$char")
  echo "  Position $((i + 1)): '$char' (ASCII $ascii)"
done

exit 0
```

### Example 11-6 (Enhanced): Secure File Deletion

```bash
#!/bin/bash
# secure-delete.sh: Securely overwrite file before deletion
# Overwrites file multiple times with random data to prevent recovery

PASSES=7          # Number of overwrite passes
BLOCKSIZE=1       # Write byte-by-byte (slower but thorough)
E_BADARGS=70

if [ $# -ne 1 ]; then
  echo "Usage: $(basename $0) filename"
  echo "This script SECURELY DELETES a file by overwriting it"
  exit $E_BADARGS
fi

file="$1"

if [ ! -e "$file" ]; then
  echo "✗ Error: File not found: $file"
  exit 71
fi

# Warn user
echo "WARNING: This will securely delete: $file"
read -p "Are you absolutely sure (y/N)? " -n 1 answer
echo

case "$answer" in
  [yY])
    echo "Proceeding with secure deletion..."
    ;;
  *)
    echo "Cancelled"
    exit 0
    ;;
esac

# Get file size
file_size=$(stat -c%s "$file")

echo "File size: $file_size bytes"
echo

# Overwrite passes
chmod u+w "$file" 2>/dev/null

for ((pass = 1; pass <= PASSES; pass++)); do
  echo "Pass $pass/$PASSES: Overwriting with $([ $((pass % 2)) -eq 0 ] && echo "zeros" || echo "random data")"
  
  if [ $((pass % 2)) -eq 0 ]; then
    # Even passes: zeros
    dd if=/dev/zero of="$file" bs=$BLOCKSIZE count=$file_size conv=notrunc 2>/dev/null
  else
    # Odd passes: random data
    dd if=/dev/urandom of="$file" bs=$BLOCKSIZE count=$file_size conv=notrunc 2>/dev/null
  fi
  
  sync  # Flush to disk
done

# Final pass: zeros
echo "Pass $((PASSES + 1))/$((PASSES + 1)): Final overwrite with zeros"
dd if=/dev/zero of="$file" bs=$BLOCKSIZE count=$file_size conv=notrunc 2>/dev/null
sync

# Delete the overwritten file
rm -f "$file"
sync

echo "✓ File securely deleted: $file"
exit 0
```

**Security note:** Modern SSDs with wear-leveling may not fully support this technique. For critical data, encrypt before deletion or use dedicated secure deletion utilities.

---

## 11.2. od: Octal and Binary Dump Filter

### Understanding od for Binary Analysis

The `od` (octal dump) command displays file contents in various numeric bases. It's essential for examining binary data, device files, and non-printable characters.

### Common Output Formats

| Option | Format | Use Case |
|--------|--------|----------|
| `-x` | Hexadecimal | Most readable binary format |
| `-o` | Octal | Historical (less common now) |
| `-d` | Decimal | Numeric value analysis |
| `-c` | ASCII characters | See printable chars |
| `-tu4` | Unsigned 4-byte integers | Large number analysis |
| `-N bytes` | Read only N bytes | Sample first N bytes |

### Example 11-7: Analyzing Binary Files

```bash
#!/bin/bash
# binary-analyzer.sh: Examine binary file structure

if [ $# -ne 1 ]; then
  echo "Usage: $0 filename"
  exit 1
fi

file="$1"

if [ ! -f "$file" ]; then
  echo "Error: File not found"
  exit 1
fi

echo "=== Binary Analysis: $file ==="
echo

echo "First 16 bytes in hexadecimal:"
od -N16 -x "$file"

echo
echo "First 16 bytes as ASCII:"
od -N16 -c "$file"

echo
echo "File magic bytes (first 4 bytes as hex):"
od -N4 -x "$file" | head -1

exit 0
```

### Example 11-8: Generate Random Integer

```bash
#!/bin/bash
# random-int.sh: Generate random unsigned integer using /dev/urandom

# Read 4 random bytes and interpret as unsigned 32-bit integer
random_int=$(head -c4 /dev/urandom | od -N4 -tu4 | sed -ne '1s/.* //p')

echo "Random unsigned 32-bit integer: $random_int"
echo "Range: 0 to 4,294,967,295"

exit 0
```

---

## 11.3. hexdump: Hexadecimal Display

### Displaying Files in Hex Format

The `hexdump` command displays file contents in hexadecimal format with side-by-side ASCII representation.

```bash
# Display binary file in hex
hexdump -C /bin/ls | head -20

# Show first 256 bytes
hexdump -C -n 256 /bin/ls

# Pretty formatting
hexdump -C /some/binary/file | less
```

**Output format:**
```
00000000  7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00  |.ELF............|
00000010  02 00 03 00 01 00 00 00 80 83 04 08 34 00 00 00  |............4...|
```

---

## 11.4. objdump: Object File Analysis

### Analyzing Executable Files

The `objdump` command displays information about compiled binaries and object files.

```bash
# Disassemble executable (machine code → assembly)
objdump -d /bin/ls | head -50

# Show all sections
objdump -h /bin/ls

# List symbols
objdump -t /bin/ls

# Show relocation information
objdump -r /bin/ls
```

---

## 11.5. mcookie: Pseudorandom Cookie Generator

### Generating Random Tokens

The `mcookie` command generates 128-bit hexadecimal pseudorandom numbers (32 characters).

```bash
#!/bin/bash
# temp-filename.sh: Generate unique temporary filename

base_str=$(mcookie)
suffix="${base_str:0:8}"  # Use first 8 characters
temp_file="/tmp/work_$suffix"

echo "Temporary file: $temp_file"

# Create unique file
touch "$temp_file"
ls -l "$temp_file"

# Cleanup
rm "$temp_file"

exit 0
```

**Practical use:**
- Session tokens
- Unique filenames
- Database record identifiers
- Cryptographic seeding (not for serious crypto)

---

## 11.6. units: Unit Conversion

### Converting Between Measurement Units

The `units` command converts between different units of measurement (length, weight, temperature, etc.).

```bash
#!/bin/bash
# convert-units.sh: Safe unit conversion function

convert() {
  local value="$1"
  local from_unit="$2"
  local to_unit="$3"
  
  # Extract conversion factor from units output
  local factor=$(units "$from_unit" "$to_unit" 2>/dev/null | sed -n '1s/.*multiplied by //p')
  
  if [ -z "$factor" ]; then
    echo "Conversion not supported: $from_unit → $to_unit"
    return 1
  fi
  
  # Calculate result
  echo "scale=6; $value * $factor" | bc
}

echo "5 miles to kilometers:"
result=$(convert 5 miles km)
echo "$result km"

echo
echo "32 Fahrenheit to Celsius:"
result=$(convert 32 degF degC)
echo "$result °C"

exit 0
```

---

## 11.7. m4: Macro Processing

### Text Macro Expansion

The `m4` command is a powerful macro processor for text expansion and manipulation.

```bash
#!/bin/bash
# macro-example.sh: Using m4 for text processing

echo "String length:"
echo "len(hello_world)" | m4

echo
echo "Substring extraction:"
echo "substr(hello_world, 7)" | m4  # From position 7 onwards

echo
echo "Arithmetic:"
echo "eval(10 + 20 * 3)" | m4

echo
echo "Incrementing numbers:"
for i in 1 2 3; do
  echo "incr($i)" | m4
done

exit 0
```

---

## 11.8. GUI Dialog Utilities

### Creating Dialog Boxes from Shell Scripts

Several utilities provide GUI dialogs for user interaction:

**zenity** (GTK+ based):
```bash
# Simple message box
zenity --info --text "Hello, World!"

# File selection
file=$(zenity --file-selection)
echo "Selected: $file"

# Text input
name=$(zenity --entry --text "Enter your name:")
echo "Hello, $name"

# Question dialog
zenity --question --text "Do you want to continue?"
[ $? -eq 0 ] && echo "User said yes"
```

**dialog** (Terminal based):
```bash
# Message box
dialog --msgbox "Welcome!" 10 30

# Menu selection
dialog --menu "Choose:" 10 30 5 \
  "Option 1" "Description 1" \
  "Option 2" "Description 2"

# Password input
dialog --passwordbox "Enter password:" 10 30
```

---

## 11.9. sox: Sound Exchange

### Audio File Processing

The `sox` command is a powerful audio processor for format conversion and effects.

```bash
#!/bin/bash
# audio-convert.sh: Convert audio formats

if [ $# -ne 2 ]; then
  echo "Usage: $0 input_file output_file"
  exit 1
fi

input="$1"
output="$2"

# Convert WAV to MP3
sox "$input" "$output"

# Convert with effects
sox "$input" "$output" lowpass 3000  # Apply low-pass filter

# Normalize audio
sox "$input" "$output" norm

# Concatenate audio files
sox input1.wav input2.wav combined.wav

exit 0
```

---

## Best Practices and Common Patterns

### 1. Always Redirect dd Diagnostic Output

```bash
# Good: Suppress dd's verbose output
dd if=source of=dest 2>/dev/null

# Bad: Shows confusing byte counts
dd if=source of=dest

# See output if needed
dd if=source of=dest 2>&1 | grep "records"
```

### 2. Use Block Sizes Appropriately

```bash
# Fast for large files: 4KB blocks
dd if=/dev/urandom of=largefile bs=4096 count=1000

# Precise for small extractions: 1 byte
dd if=file of=extract bs=1 skip=10 count=5
```

### 3. Verify Conversions with hexdump

```bash
# After conversion, verify results
dd if=input of=output conv=ucase
hexdump -C output | less  # Inspect result
```

### 4. Save Terminal Settings Before Modification

```bash
#!/bin/bash
old_settings=$(stty -g)
trap "stty "$old_settings"" EXIT  # Restore on exit

stty -echo  # Disable echo
read password
stty "$old_settings"
```

---

## 10 Core Programming Concepts from External Commands

### 1. **Specialized Tools and Modularity**
Unix philosophy: Small tools that do one thing well. External commands represent specialization—each tool handles specific tasks (binary processing, audio, conversion) that would be bloated as builtins.

```bash
dd           # Binary manipulation
od           # Binary analysis
sox          # Audio processing
```

### 2. **Block-Based Processing**
`dd` introduces block-based I/O—reading/writing data in fixed-size chunks. This is fundamental to efficient file processing and appears in many programming contexts.

### 3. **Format Conversions and Adapters**
The adapter pattern: Converting between formats (`ucase`, `unblock`, `swab`) without changing semantic content.

### 4. **Streams and Pipelines**
External commands are designed for piping—each program reads from stdin, processes, writes to stdout. This enables composition of complex operations.

```bash
cat file | od -x | grep pattern | sed transform
```

### 5. **Random Access and Seeking**
`dd` with `skip` and `seek` enables random access to sequential data—crucial for file parsing and binary manipulation.

### 6. **Data Inspection and Analysis**
`od`, `hexdump`, `objdump` reveal internal structure—fundamental to debugging and reverse engineering.

### 7. **Signal Safety**
Understanding when to save/restore system state (terminal settings, signals) before spawning external commands.

### 8. **Text vs. Binary Processing**
Different tools for different data types (`od` for binary, `m4` for text), highlighting the importance of understanding data representation.

### 9. **Macro Expansion and Templating**
`m4` demonstrates metaprogramming—programs that generate programs. Text-based template expansion is fundamental to many build systems and code generators.

### 10. **Composition and Integration**
The power of external commands lies in their **composability**—combining simple tools into powerful workflows:

```bash
find . -type f | xargs file | grep "text" | sed 's/:.*//g' | sort | uniq
# Find → categorize → filter → extract → sort → deduplicate
```

---

## Exercises

1. **Binary Analyzer**: Write a script that uses `dd` and `hexdump` to analyze executable files and display their magic numbers.

2. **Text Transformer**: Create a script using `dd conv=` options to build a text processing pipeline (uppercase, remove duplicates, sort).

3. **Secure Eraser**: Implement a safe file deletion tool that overwrites files multiple times before deletion (reference: secure-delete.sh example).

4. **Audio Batch Processor**: Write a script using `sox` to batch convert audio files and apply effects.

5. **Random Token Generator**: Create a function using `mcookie` to generate unique identifiers for sessions or temporary files.

6. **Unit Converter**: Build a conversion calculator that handles temperature, distance, and weight using the `units` command.

7. **Dialog Menu System**: Create an interactive menu using `zenity` or `dialog` for user interaction.

8. **Binary File Matcher**: Use `od` to find binary patterns in files and report their positions.

---

## Summary

External commands extend Bash with specialized capabilities:

- **`dd`**: Block-based data copying with conversions (binary manipulation, format conversion)
- **`od`, `hexdump`**: Binary data inspection (viewing non-printable data, analysis)
- **`objdump`**: Executable analysis (disassembly, symbol extraction)
- **`mcookie`**: Random token generation (unique identifiers, session tokens)
- **`units`**: Unit conversion (measurement conversion)
- **`m4`**: Macro processing (text templating, code generation)
- **`sox`**: Audio processing (format conversion, effects)
- **`zenity`, `dialog`**: GUI dialogs (user interaction in scripts)

**Key insight**: External commands represent Unix philosophy's modularity. Bash's power comes not from what it does, but from its ability to **compose** these specialized tools into powerful workflows. Understanding external commands means understanding pipeline composition and data transformation.
