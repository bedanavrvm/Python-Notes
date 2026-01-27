# Bash Basics — Special Characters and Common Idioms

This chapter explains what we mean by "special characters" in the shell, how certain utilities treat the `-` token as stdin/stdout, and several common idioms (pipes, subshells, handling filenames that start with `-`, and control characters). Examples are practical and focused on safely using these features in scripts.

## What is a special character?

A special character is any character that has meaning beyond its literal glyph. In the shell this includes operators (`|`, `>`, `<`, `&`), metacharacters used by globbing (`*`, `?`, `[]`), quoting characters (`'`, `"`, `\`), and standalone tokens like `-` which some programs interpret as "use standard input/output".

Special characters depend on context — the same character can mean different things in different places.

## The `-` token: stdin/stdout and filters

Many Unix utilities accept `-` to mean standard input (when a program reads) or standard output (when a program writes). This lets you plug traditionally file-oriented tools into pipelines and filters.

Examples:

```bash
# echo piped to cat reading from stdin
echo "whatever" | cat -
# outputs: whatever

# Use tar with - to stream an archive between directories
(cd /source/directory && tar cf - .) | (cd /dest/directory && tar xpvf -)

# Uncompress a bzip2 archive and pipe into tar for extraction
# (useful when tar doesn't support bzip2 directly)
bunzip2 -c linux-2.6.16.tar.bz2 | tar xvf -

# Use `file -` to make `file` analyze stdin interactively
file -
# then type or paste content; Ctrl-D ends input
```

Why this is useful:
- Treat a command that normally writes to or reads from files as a filter in a pipeline.
- Avoid temporary files and improve streaming performance.

### Using `-` with `diff` and `grep`

Compare a file with output from a pipeline:

```bash
grep Linux file1 | diff file2 -
```

This compares `file2` against the lines from `file1` that mention `Linux`.

## Tar example: backup files modified in the last day

The original short script approach works, but it fails when filenames contain whitespace or when too many files are passed on the command line. Here are safer alternatives.

Simple (unsafe for filenames with spaces):

```bash
#!/bin/bash
BACKUPFILE=backup-$(date +%m-%d-%Y)
archive=${1:-$BACKUPFILE}
# WARNING: this will break on filenames with spaces
tar cvf - $(find . -mtime -1 -type f -print) > "$archive.tar"
gzip "$archive.tar"
echo "Directory $PWD backed up in archive file \"$archive.tar.gz\"."
```

Improved (handles spaces, GNU tools):

```bash
#!/bin/bash
BACKUPFILE=backup-$(date +%m-%d-%Y)
archive=${1:-$BACKUPFILE}
# Using find -print0 and xargs -0 to handle arbitrary filenames
find . -mtime -1 -type f -print0 | xargs -0 tar rvf "$archive.tar"
gzip "$archive.tar"
```

Portable alternative (without xargs -0):

```bash
#!/bin/bash
BACKUPFILE=backup-$(date +%m-%d-%Y)
archive=${1:-$BACKUPFILE}
find . -mtime -1 -type f -exec tar rvf "$archive.tar" '{}' \;
gzip "$archive.tar"
```

Notes:
- `tar rvf` appends files to an existing tar; creating a tar from a stream uses `tar cvf -` and `-` as the filename to mean stdout/stdin.
- When many files exist, prefer streaming or use `--null --files-from -` if your tar supports it.

## Filenames beginning with `-` and the `--` convention

Filenames that start with a dash can be misinterpreted as options. Use one of these strategies:

- Prefix with `./` to make it non-option: `./-myfile`.
- Use the `--` end-of-options marker to tell the command that following arguments are positional/filenames: `rm -- -file`.
- Construct full or relative paths: `$PWD/-file`.

Example:

```bash
# remove a file literally named "-f" safely
rm -- -f

# or
rm ./-f
```

## Variables that begin with `-` and builtin ambiguity

If a variable's value begins with a dash, passing it unquoted to a command can make the command treat it as an option:

```bash
var="-n"
echo $var       # expands to: echo -n  -> echo will not print a newline
echo "$var"   # expands to: echo -n  -> still treated as option by echo

# Prefer using -- where possible to separate options from positional args
printf '%s\n' -- "$var"
```

## Other useful special characters and tokens (quick reference)

- `~` : tilde expansion (home directory). `~user` expands to that user's home.
- `* ? []` : globbing patterns (wildcards).
- `$IFS` : field separator used by word splitting (defaults to whitespace).
- `%` : used in some shells and in parameter expansion pattern matching.
- `^` : beginning-of-line in regexes; `^^` upper-cases in parameter substitution (bash 4+).
- `=` : assignment (`a=28`) or used in `[[` comparisons.

## Control characters and terminal behavior

Control characters are produced with Ctrl+key combinations and affect the terminal (not usually used inside scripts):

- `Ctrl-C` : interrupt (SIGINT) — stops a foreground job.
- `Ctrl-D` : End-of-file on a terminal; when typed on an empty line it logs out the shell.
- `Ctrl-Z` : suspend the current foreground job (SIGTSTP).
- `Ctrl-R` : reverse search through command history (interactive).

Be careful when embedding control characters into scripts; prefer explicit escapes like `$'\x0a'` or `\n` in printf.

## Whitespace and `IFS`

Whitespace (spaces, tabs, newlines) separates words in the shell. To preserve whitespace inside variables or arguments always quote expansions:

```bash
name="John Doe"
echo "$name"   # prints 'John Doe'
echo $name       # prints 'John' 'Doe' separately if used in a context that splits words
```

For advanced parsing, change `IFS` carefully and locally (e.g., `IFS=',' read -r a b <<< "$line"`).

## Exercises

1. Update the improved backup script to use `tar --null -T - -cvf` if your `tar` supports it; otherwise keep the `find -print0 | xargs -0` pattern and document compatibility notes.
2. Write a small script `safe-rm` that safely removes files whose names may start with `-`. The script should accept filenames as arguments and remove them without being confused by leading `-` characters.
3. Demonstrate the difference between `echo $var` and `echo "$var"` with a variable containing multiple whitespace characters and special characters.

---

When you're ready I can:

- Add a chapter on quoting, expansions, and parameter substitution.
- Produce a tested `sysreport.sh` with rotation and CLI flags.
- Add link-checking automation to ensure `SUMMARY.md` and file paths remain in sync.
