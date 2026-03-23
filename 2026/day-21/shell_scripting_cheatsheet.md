# Shell Scripting Cheat Sheet

> Personal quick-reference guide built during #90DaysOfDevOps — Days 16–21

---

## Quick Reference Table

| Topic | Key Syntax | Example |
|-------|-----------|---------|
| Variable | `VAR="value"` | `NAME="DevOps"` |
| Argument | `$1`, `$2` | `./script.sh arg1` |
| If | `if [ condition ]; then` | `if [ -f file ]; then` |
| For loop | `for i in list; do` | `for i in 1 2 3; do` |
| While loop | `while [ condition ]; do` | `while [ $i -lt 5 ]; do` |
| Function | `name() { ... }` | `greet() { echo "Hi"; }` |
| Local var | `local VAR="val"` | `local COUNT=0` |
| Strict mode | `set -euo pipefail` | First line after shebang |
| Grep | `grep pattern file` | `grep -i "error" log.txt` |
| Awk | `awk '{print $1}' file` | `awk -F: '{print $1}' /etc/passwd` |
| Sed | `sed 's/old/new/g' file` | `sed -i 's/foo/bar/g' config.txt` |

---

## 1. Basics

### Shebang
```bash
#!/bin/bash
```
Tells the OS which interpreter to use. Without it, the script may run with `sh` instead of `bash` — and some bash features won't work.

### Running a Script
```bash
chmod +x script.sh     # make executable
./script.sh            # run directly
bash script.sh         # run with bash explicitly (no chmod needed)
```

### Comments
```bash
# This is a full-line comment
echo "Hello"  # This is an inline comment
```

### Variables
```bash
NAME="Wajih"
echo $NAME          # basic usage
echo "$NAME"        # always prefer quoted — prevents word splitting
echo '$NAME'        # single quotes: literal, no expansion → prints $NAME
```

### Reading User Input
```bash
read -p "Enter your name: " USERNAME
echo "Hello, $USERNAME!"
```

### Command-Line Arguments
```bash
$0    # script name itself
$1    # first argument
$2    # second argument
$#    # total number of arguments
$@    # all arguments as separate words
$?    # exit code of last command (0 = success)
```

---

## 2. Operators and Conditionals

### String Comparisons
```bash
[ "$A" = "$B" ]    # equal
[ "$A" != "$B" ]   # not equal
[ -z "$A" ]        # true if string is empty
[ -n "$A" ]        # true if string is NOT empty
```

### Integer Comparisons
```bash
[ $A -eq $B ]   # equal
[ $A -ne $B ]   # not equal
[ $A -lt $B ]   # less than
[ $A -gt $B ]   # greater than
[ $A -le $B ]   # less than or equal
[ $A -ge $B ]   # greater than or equal
```

### File Test Operators
```bash
[ -f file ]   # is a regular file
[ -d dir ]    # is a directory
[ -e path ]   # exists (file or dir)
[ -r file ]   # readable
[ -w file ]   # writable
[ -x file ]   # executable
[ -s file ]   # exists and size > 0
```

### If / Elif / Else
```bash
if [ "$USER" = "root" ]; then
    echo "You are root"
elif [ -f "/etc/config" ]; then
    echo "Config file exists"
else
    echo "Neither condition met"
fi
```

### Logical Operators
```bash
[ -f file ] && echo "exists"       # AND
[ -f file ] || echo "not found"    # OR
[ ! -f file ] && echo "missing"    # NOT
```

### Case Statement
```bash
case "$1" in
    start)   echo "Starting..." ;;
    stop)    echo "Stopping..." ;;
    restart) echo "Restarting..." ;;
    *)       echo "Unknown command" ;;
esac
```

---

## 3. Loops

### For Loop — List Based
```bash
for name in Alice Bob Charlie; do
    echo "Hello, $name"
done
```

### For Loop — C-Style
```bash
for (( i=1; i<=5; i++ )); do
    echo "Count: $i"
done
```

### While Loop
```bash
COUNT=0
while [ $COUNT -lt 5 ]; do
    echo "Count: $COUNT"
    (( COUNT++ ))
done
```

### Until Loop
```bash
until [ $COUNT -ge 5 ]; do
    echo "Count: $COUNT"
    (( COUNT++ ))
done
```

### Loop Control
```bash
break       # exit the loop entirely
continue    # skip current iteration, go to next
```

### Loop Over Files
```bash
for file in *.log; do
    echo "Processing: $file"
done
```

### Loop Over Command Output
```bash
while read line; do
    echo "Line: $line"
done < /etc/passwd
```

---

## 4. Functions

### Defining and Calling
```bash
greet() {
    echo "Hello, $1!"
}

greet "Wajih"   # → Hello, Wajih!
```

### Passing Arguments
```bash
add() {
    echo $(( $1 + $2 ))
}

add 10 20   # → 30
```

### Return Values

`return` only returns exit codes (0–255). Use `echo` to return actual values:
```bash
get_date() {
    echo "$(date +%Y-%m-%d)"
}

TODAY=$(get_date)
echo "Today is: $TODAY"
```

### Local Variables
```bash
counter() {
    local COUNT=0
    COUNT=$(( COUNT + 1 ))
    echo "Inside: $COUNT"
}

counter
echo "Outside: ${COUNT:-not set}"   # → not set (local didn't leak)
```

---

## 5. Text Processing Commands

### `grep` — Search Patterns
```bash
grep "error" file.log          # basic search
grep -i "error" file.log       # case-insensitive
grep -r "TODO" ./src/          # recursive search
grep -c "error" file.log       # count matching lines
grep -n "error" file.log       # show line numbers
grep -v "info" file.log        # invert — lines NOT matching
grep -E "ERROR|WARN" file.log  # extended regex (multiple patterns)
```

### `awk` — Column Processing
```bash
awk '{print $1}' file              # print first column
awk -F: '{print $1}' /etc/passwd   # custom field separator
awk '$3 > 100 {print $1}' file     # conditional filter
awk 'BEGIN{sum=0} {sum+=$2} END{print sum}' file  # sum a column
```

### `sed` — Stream Editor
```bash
sed 's/foo/bar/g' file           # replace all occurrences
sed -i 's/foo/bar/g' file        # in-place edit (modifies file)
sed '/^#/d' file                 # delete comment lines
sed -n '10,20p' file             # print lines 10–20 only
```

### `cut` — Extract Columns
```bash
cut -d: -f1 /etc/passwd          # first field, colon delimiter
cut -d, -f2,4 data.csv           # fields 2 and 4, comma delimiter
cut -c1-10 file                  # first 10 characters of each line
```

### `sort`
```bash
sort file                  # alphabetical
sort -n file               # numerical
sort -r file               # reverse
sort -u file               # unique (deduplicates)
sort -t: -k3 -n /etc/passwd  # sort by 3rd field, colon-delimited
```

### `uniq` — Deduplicate
```bash
uniq file              # remove adjacent duplicates (sort first!)
uniq -c file           # count occurrences
uniq -d file           # only show duplicated lines
sort file | uniq -c | sort -rn   # frequency count pattern
```

### `tr` — Translate Characters
```bash
echo "hello" | tr 'a-z' 'A-Z'    # lowercase to uppercase
echo "hello world" | tr -d ' '   # delete spaces
echo "a:b:c" | tr ':' ','        # replace colons with commas
```

### `wc` — Count
```bash
wc -l file    # count lines
wc -w file    # count words
wc -c file    # count bytes/characters
```

### `head` / `tail`
```bash
head -5 file          # first 5 lines
tail -5 file          # last 5 lines
tail -f /var/log/syslog   # follow (live stream)
tail -n +10 file      # all lines from line 10 onwards
```

---

## 6. Useful One-Liners

```bash
# Find and delete files older than 30 days
find /var/log -name "*.gz" -mtime +30 -delete

# Count lines across all .log files
wc -l *.log | tail -1

# Replace a string across multiple files
sed -i 's/old_string/new_string/g' *.conf

# Check if a service is running
systemctl is-active --quiet nginx && echo "Running" || echo "Stopped"

# Monitor disk usage and alert if over 80%
USAGE=$(df / | awk 'NR==2 {print $5}' | tr -d '%')
[ "$USAGE" -gt 80 ] && echo "ALERT: Disk at ${USAGE}%"

# Tail a log and filter for errors in real time
tail -f /var/log/syslog | grep --line-buffered "ERROR"

# Count unique IPs in an access log
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10

# Extract HTTP status codes from nginx access log
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c | sort -rn
```

---

## 7. Error Handling and Debugging

### Exit Codes
```bash
echo $?        # exit code of last command (0 = success, non-0 = failure)
exit 0         # exit script with success
exit 1         # exit script with error
```

### Strict Mode Flags
```bash
set -e          # exit immediately on any error
set -u          # treat unset variables as errors
set -o pipefail # fail if any command in a pipe fails
set -x          # debug mode — prints every command as it executes
```

Combine them:
```bash
set -euo pipefail
```

### Trap — Run Cleanup on Exit
```bash
cleanup() {
    echo "Cleaning up temp files..."
    rm -f /tmp/myapp_*
}

trap cleanup EXIT          # runs on any exit
trap cleanup EXIT INT TERM # runs on exit, Ctrl+C, or kill signal
```

### Debug Mode
```bash
bash -x script.sh          # run entire script in debug mode
set -x                     # enable debug from this point
set +x                     # disable debug
```

---

## 8. Quick Patterns

### Check if running as root
```bash
if [[ $EUID -ne 0 ]]; then
    echo "Run this script as root."
    exit 1
fi
```

### Default value for variable
```bash
NAME="${1:-default_value}"
```

### Confirm before destructive action
```bash
read -p "Delete all logs? (y/N): " CONFIRM
[[ "$CONFIRM" =~ ^[Yy]$ ]] || exit 0
```

### Timestamp for filenames
```bash
TIMESTAMP=$(date +%Y-%m-%d_%H-%M-%S)
ARCHIVE="backup_${TIMESTAMP}.tar.gz"
```
