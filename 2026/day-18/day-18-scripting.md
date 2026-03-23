# Day 18 – Shell Scripting: Functions & Intermediate Concepts

## Task 1: Basic Functions — `functions.sh`

```bash
#!/bin/bash

greet() {
    echo "Hello, $1!"
}

add() {
    local result=$(( $1 + $2 ))
    echo "Sum of $1 and $2 = $result"
}

greet "Wajih"
greet "Shubham"
add 10 25
add 7 93
```

**Output:**
```
Hello, Wajih!
Hello, Shubham!
Sum of 10 and 25 = 35
Sum of 7 and 93 = 100
```

---

## Task 2: Functions with Return Values — `disk_check.sh`

```bash
#!/bin/bash

check_disk() {
    echo "=== Disk Usage ==="
    df -h /
}

check_memory() {
    echo "=== Memory Usage ==="
    free -h
}

check_disk
echo ""
check_memory
```

**Output:**
```
=== Disk Usage ===
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda1       20G  4.2G   16G  21% /

=== Memory Usage ===
               total        used        free      shared  buff/cache   available
Mem:           985Mi       312Mi       145Mi       1.0Mi       527Mi       524Mi
Swap:             0B          0B          0B
```

---

## Task 3: Strict Mode — `strict_demo.sh`

```bash
#!/bin/bash
set -euo pipefail

# Uncomment one at a time to test each behavior

# Test set -u: using undefined variable
# echo $UNDEFINED_VAR

# Test set -e: command that fails
# ls /nonexistent_directory

# Test set -o pipefail: pipe where one part fails
# cat /nonexistent_file | grep "something"

echo "Script ran successfully."
```

### What Each Flag Does

| Flag | Behavior |
|------|----------|
| `set -e` | Script **exits immediately** if any command returns a non-zero exit code |
| `set -u` | Script **exits with error** if an undefined/unset variable is referenced |
| `set -o pipefail` | Script **fails** if any command in a pipe fails, not just the last one |

**Without `set -o pipefail`:**
```bash
cat /nonexistent | grep "foo"   # exit code = 0 (grep succeeds on empty input)
```
**With `set -o pipefail`:**
```bash
cat /nonexistent | grep "foo"   # exit code = 1 (cat failed, pipe fails)
```

---

## Task 4: Local Variables — `local_demo.sh`

```bash
#!/bin/bash

# Function WITH local variable
with_local() {
    local MY_VAR="I am LOCAL"
    echo "Inside with_local: $MY_VAR"
}

# Function WITHOUT local variable
without_local() {
    GLOBAL_VAR="I am GLOBAL"
    echo "Inside without_local: $GLOBAL_VAR"
}

with_local
echo "Outside with_local: ${MY_VAR:-'[variable not accessible - as expected]'}"

without_local
echo "Outside without_local: $GLOBAL_VAR"
```

**Output:**
```
Inside with_local: I am LOCAL
Outside with_local: [variable not accessible - as expected]
Inside without_local: I am GLOBAL
Outside without_local: I am GLOBAL
```

> `local` variables are **scoped to the function**. Without `local`, variables bleed into the global scope — a common source of bugs in longer scripts.

---

## Task 5: System Info Reporter — `system_info.sh`

```bash
#!/bin/bash
set -euo pipefail

print_header() {
    echo ""
    echo "=========================================="
    echo "  $1"
    echo "=========================================="
}

host_info() {
    print_header "HOSTNAME & OS INFO"
    echo "Hostname   : $(hostname)"
    echo "OS         : $(grep PRETTY_NAME /etc/os-release | cut -d= -f2 | tr -d '"')"
    echo "Kernel     : $(uname -r)"
    echo "Architecture: $(uname -m)"
}

uptime_info() {
    print_header "UPTIME"
    uptime -p
}

disk_info() {
    print_header "TOP 5 DISK USAGE"
    du -h / --max-depth=1 2>/dev/null | sort -rh | head -5
}

memory_info() {
    print_header "MEMORY USAGE"
    free -h
}

cpu_info() {
    print_header "TOP 5 CPU-CONSUMING PROCESSES"
    ps aux --sort=-%cpu | awk 'NR==1 || NR<=6' | awk '{printf "%-10s %-6s %-6s %s\n", $1, $2, $3, $11}'
}

main() {
    echo ""
    echo "  SYSTEM INFO REPORT — $(date '+%Y-%m-%d %H:%M:%S')"
    host_info
    uptime_info
    disk_info
    memory_info
    cpu_info
    echo ""
    echo "=========================================="
    echo "  END OF REPORT"
    echo "=========================================="
}

main
```

**Sample Output:**
```
  SYSTEM INFO REPORT — 2026-03-23 02:14:37

==========================================
  HOSTNAME & OS INFO
==========================================
Hostname   : wajihworkstation
OS         : Ubuntu 24.04.1 LTS
Kernel     : 6.8.0-1021-aws
Architecture: x86_64

==========================================
  UPTIME
==========================================
up 2 hours, 14 minutes

==========================================
  TOP 5 DISK USAGE
==========================================
7.4G    /
3.1G    /usr
1.8G    /snap
856M    /var
312M    /lib

==========================================
  MEMORY USAGE
==========================================
               total        used        free
Mem:           985Mi       312Mi       145Mi
Swap:             0B          0B          0B

==========================================
  TOP 5 CPU-CONSUMING PROCESSES
==========================================
USER       PID    %CPU   COMMAND
root       1023   2.1    /usr/bin/python3
wajih      2841   1.4    bash
root       512    0.8    /usr/sbin/sshd
www-data   3301   0.5    nginx
root       1      0.1    /sbin/init

==========================================
  END OF REPORT
==========================================
```

---

## Explanation of `set -euo pipefail`

```bash
set -euo pipefail
```

This single line makes your scripts **dramatically safer**:

- **`-e`** → Exit immediately on any error. No more scripts silently continuing after a failed command.
- **`-u`** → Treat unset variables as errors. Catches typos like `$USERNMAE` instead of `$USERNAME`.
- **`-o pipefail`** → The entire pipe fails if any command in it fails. Without this, `false | true` would exit 0.

Always put this at the top of every script you write for production use.

---

## What I Learned

1. **Functions make scripts reusable and readable** — instead of repeating 5 lines of disk-check code, you write it once and call it anywhere.
2. **`local` is not optional, it's a habit** — variables without `local` pollute the global scope and cause hard-to-debug side effects in longer scripts.
3. **`set -euo pipefail` is non-negotiable for real scripts** — without it, a failing command mid-script can leave your system in a broken half-state silently.
