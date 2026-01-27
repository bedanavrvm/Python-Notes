# Chapter 11: External Commands and Utilities

External commands and utilities extend Bash's capabilities for data processing, file manipulation, and system-level operations. This chapter covers powerful tools for binary data handling, conversions, and specialized processing.

## dd: Data Duplication and Conversion

The `dd` command copies files or streams with optional data conversions. Originally designed for exchanging data between different systems, it remains useful for specialized file operations, binary data handling, and low-level disk operations.

### Basic Syntax

```bash
dd if=INFILE of=OUTFILE bs=BLOCKSIZE skip=BLOCKS seek=BLOCKS count=BLOCKS conv=CONVERSION
```

### Key Options

| Option | Purpose |
|--------|---------|
| `if=INFILE` | Source file (default: stdin) |
| `of=OUTFILE` | Target/destination file (default: stdout) |
| `bs=BLOCKSIZE` | Block size in bytes (typically power of 2) |
| `skip=BLOCKS` | Number of input blocks to skip before copying |
| `seek=BLOCKS` | Number of output blocks to skip before writing |
| `count=BLOCKS` | Number of blocks to copy (omit for entire file) |
| `conv=CONVERSION` | Conversion type (ucase, lcase, notrunc, unblock, etc.) |

### Common Conversions

- `ucase` - Convert to uppercase
- `lcase` - Convert to lowercase
- `notrunc` - Don't truncate output file
- `unblock` - Convert fixed-length records to variable-length

### Examples

**Converting file to uppercase:**
```bash
dd if=$filename conv=ucase > $filename.uppercase
```

**Extracting characters from file:**
```bash
infile=$0
outfile=log.txt
n=8
p=11
dd if=$infile of=$outfile bs=1 skip=$((n-1)) count=$((p-n+1)) 2>/dev/null
# Extracts characters 8 to 11 from input file
```

**Vertical text output:**
```bash
echo -n "hello world" | dd cbs=1 conv=unblock 2>/dev/null
# Outputs text vertically (one character per line)
```

**Random access on data stream:**
```bash
echo -n . | dd bs=1 seek=4 of=file conv=notrunc
# Places dot at position 4 without truncating file
```

**Capturing keystrokes without ENTER:**
```bash
#!/bin/bash
# dd-keypress.sh

keypresses=4
old_tty_setting=$(stty -g)

echo "Press $keypresses keys."
stty -icanon -echo

keys=$(dd bs=1 count=$keypresses 2>/dev/null)

stty "$old_tty_setting"

echo "You pressed the \"$keys\" keys."
exit 0
```

**Securely deleting a file:**
```bash
#!/bin/bash
# blot-out.sh: Secure file deletion

PASSES=7
BLOCKSIZE=1
E_BADARGS=70

if [ -z "$1" ]; then
    echo "Usage: $(basename $0) filename"
    exit $E_BADARGS
fi

file=$1

if [ ! -e "$file" ]; then
    echo "File \"$file\" not found."
    exit 71
fi

read -p "Are you absolutely sure you want to delete \"$file\" (y/n)? " answer

case "$answer" in
    [nN]) echo "Changed your mind, huh?"
          exit 72
          ;;
    *)    echo "Blotting out file \"$file\"."
          ;;
esac

flength=$(ls -l "$file" | awk '{print $5}')
pass_count=1

chmod u+w "$file"
echo

while [ "$pass_count" -le "$PASSES" ]; do
    echo "Pass #$pass_count"
    sync
    dd if=/dev/urandom of=$file bs=$BLOCKSIZE count=$flength
    sync
    dd if=/dev/zero of=$file bs=$BLOCKSIZE count=$flength
    sync
    ((pass_count++))
    echo
done

rm -f "$file"
sync

echo "File \"$file\" blotted out and deleted."
exit 0
```

### Use Cases for dd

- **Creating boot floppies:** `dd if=kernel-image of=/dev/fd0H1440`
- **Copying floppy disk to image:** `dd if=/dev/fd0 of=~/floppy.img`
- **Creating bootable USB drives:** `dd if=image.iso of=/dev/sdb`
- **Initializing swap files or ramdisks**
- **Low-level disk partition copying**

## od: Octal Dump Filter

The `od` command converts input to octal (base-8) or other numeric bases. Useful for viewing binary data, device files, or unreadable system data.

### Syntax

```bash
od [options] [file]
```

### Common Options

| Option | Purpose |
|--------|---------|
| `-N4` | Read only 4 bytes |
| `-tu4` | Output as unsigned 4-byte integers |
| `-x` | Hexadecimal output |
| `-c` | ASCII character output |

### Example

**Generate random unsigned 4-byte integer:**
```bash
head -c4 /dev/urandom | od -N4 -tu4 | sed -ne '1s/.* //p'
```

## hexdump: Hexadecimal Dump

The `hexdump` command displays file contents in hexadecimal, octal, decimal, or ASCII format.

### Example

```bash
dd if=/bin/ls | hexdump -C | less
```

## objdump: Object File Disassembler

The `objdump` command displays information about object files and binary executables.

```bash
objdump -d /bin/ls
```

## mcookie: Magic Cookie Generator

The `mcookie` command generates a 128-bit pseudorandom hexadecimal number (32 characters).

### Example

```bash
BASE_STR=$(mcookie)
POS=11
LEN=5
suffix=${BASE_STR:POS:LEN}
temp_filename=temp.$suffix
echo "Temp filename = $temp_filename"
```

## units: Unit Conversion

The `units` command converts between different units of measure.

```bash
convert_units() {
    cf=$(units "$1" "$2" | sed --silent -e '1p' | awk '{print $2}')
    echo "$cf"
}
```

## m4: Macro Processing Filter

The `m4` command is a powerful macro processor.

```bash
string=abcdA01
echo "len($string)" | m4
echo "substr($string,4)" | m4
echo "incr(99)" | m4
```

## GUI Dialog Utilities

- **xmessage** - X-based message boxes
- **zenity** - GTK+ dialog widgets
- **dialog** - Terminal dialog boxes

## sox: Sound Exchange

The `sox` command plays and transforms sound files.

```bash
sox soundfile.wav soundfile.au
```

## Summary

External commands provide specialized capabilities for data processing and system operations.
