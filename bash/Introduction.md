# Introduction to Bash Scripting

Bash scripting is the practical glue of Unix-like systems. As a Linux machine boots it typically runs initialization scripts (for example `/etc/rc.d` on some distributions) to restore system configuration and start services — many of those tasks are performed with small shell scripts. Shell scripts follow the classic Unix philosophy: break a complex task into small steps and chain existing utilities together.

## What is Bash?

Bash (the "Bourne-Again SHell") is a commonly used command interpreter on Linux. A Bash script is simply a text file containing shell commands; saving commands in a script has advantages beyond avoiding retyping: scripts are reproducible, automatable, and can be shared or scheduled.

## The shebang (`#!`) and interpreter selection

At the top of most scripts you'll find a "shebang" (#!) line. This tells the OS which interpreter to use to run the file:

```sh
#!/bin/bash
# or, for portability to other Unix-like systems:
#!/bin/sh
```

- The `#!` bytes act as a marker so the kernel treats the file as an executable script. 
- Use `#!/bin/sh` when you want maximum portability to systems where `/bin/sh` is not Bash (note: on many Linux distributions `/bin/sh` is a link to `/bin/bash`).
- Use `#!/bin/bash` when you rely on Bash-specific features (arrays, `[[ ]]`, process substitution, etc.).

## Making a script executable

Write your script in a text file (for example `myscript`) and make it executable using `chmod`:

```bash
# Give owner read+execute only
chmod u+rx myscript
# Give everyone read+execute
chmod +rx myscript
# A numeric mode equivalent
chmod 555 myscript
```

After that you can run it directly from the current directory:

```bash
./myscript
```

You can also run a script explicitly with an interpreter:

```bash
bash myscript
sh myscript
```

Avoid `sh < myscript` because that redirects stdin and can disable reading from standard input within the script.

To make a script available system-wide, move it to a directory on `PATH`, for example `/usr/local/bin` (requires root):

```bash
sudo mv myscript /usr/local/bin/
# Now you can run it as:
myscript
```

## Replace constants with variables; use functions

A first step toward reusable scripts is to avoid hard-coded constants. Use variables and functions to centralize configuration and avoid repetition.

Example: a small script that collects system info and appends it to a logfile.

```bash
#!/bin/bash

LOGFILE="/var/log/sysinfo.log"

record_sysinfo() {
  echo "========================" >> "$LOGFILE"
  echo "Timestamp: $(date --iso-8601=seconds)" >> "$LOGFILE"
  echo "Users: $(who | wc -l)" >> "$LOGFILE"
  echo "Logged in users:" >> "$LOGFILE"
  who >> "$LOGFILE"
  echo "Uptime: $(uptime -p)" >> "$LOGFILE"
}

# Make sure the logfile exists and is writable (adjust path/permissions as necessary)
mkdir -p "$(dirname "$LOGFILE")"
: > "$LOGFILE"  # (optional) truncate for testing; remove in production

record_sysinfo

# Also show the same output to stdout for convenience
cat "$LOGFILE"
```

Notes:
- Use `$(...)` rather than backticks for command substitution.
- Quote variable expansions `"$VAR"` to avoid word-splitting.
- Use functions to group logically related steps.

## Portability tips

- For scripts intended to run on many UNIX variants, prefer POSIX-compatible constructs and `#!/bin/sh`.
- If you need Bash features (arrays, `[[`, `==` in `[[ ]]`, `[[ -v var ]]`, `shopt`), use `#!/bin/bash` and document this at the top of the file.

## Style and best practices (practical, repo-specific)

- Keep one concept per file when authoring notes (this repo's style).
- Use a top-level heading and subsections (`#`, `##`, `###`).
- Preserve example code fences and mark the language (````bash`` or ````sh``) so renderers highlight properly.
- When referring to files in this repository, use backticks (for example: `bash/Introduction.md`).

## Preliminary Exercises

System administrators often write scripts to automate tasks. Try these exercises to practice basic scripting:

1. Write a script `sysreport.sh` that, when invoked, prints the current date/time, lists all logged-in users, shows system uptime, and appends the same information to a logfile (e.g. `/tmp/sysreport.log`). Include a usage message and an optional argument to specify a different logfile.

Hint for `sysreport.sh`:

```bash
#!/bin/bash
LOGFILE=${1:-/tmp/sysreport.log}
{
  date
  echo "Logged-in users:"
  who
  echo "Uptime:"
  uptime -p
} >> "$LOGFILE"
```

2. Extend `sysreport.sh` to rotate the logfile when it exceeds 1MB (move the old file to `sysreport.log.1` and create a new one).

3. Create a `bin/` directory in your home and move a tested script there. Add the directory to your `PATH` by editing `~/.profile` or `~/.bashrc` and verify you can run the script from any location.

---

If you'd like, I can:

- Add more chapters (variables, control flow, I/O, signals, cron integration).
- Provide a polished example `sysreport.sh` with rotation and CLI flags.
- Add the new file to `SUMMARY.md` with the correct URL-encoded link.

Tell me which next step you'd like.