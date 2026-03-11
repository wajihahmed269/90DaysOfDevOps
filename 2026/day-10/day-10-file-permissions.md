# Day 10 – File Permissions & File Operations Challenge


## Files Created

| File | Size | Created With |
|------|------|-------------|
| `devops.txt` | 0 bytes | `touch` |
| `notes.txt` | 34 bytes | `echo >` |
| `script.sh` | 28 bytes | `vim` |
| `project/` | directory | `mkdir` |

---

## Task 1: Create Files

```bash
touch devops.txt
echo "Learning DevOps one day at a time" > notes.txt
vim script.sh
ls -l devops.txt notes.txt script.sh
```

**Output:**
```
-rw-rw-r-- 1 ubuntu ubuntu  0 Mar 11 23:12 devops.txt
-rw-rw-r-- 1 ubuntu ubuntu 34 Mar 11 23:12 notes.txt
-rw-rw-r-- 1 ubuntu ubuntu 28 Mar 11 23:13 script.sh
```

**Observed:** All 3 files created successfully. Default permissions are `rw-rw-r--` (664) — no execute permission yet.

---

## Task 2: Read Files

```bash
cat notes.txt
vim -R script.sh
head -n 5 /etc/passwd
tail -n 5 /etc/passwd
```

**Output — cat notes.txt:**
```
Learning DevOps one day at a time
```

**Output — head -n 5 /etc/passwd:**
```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
```

**Output — tail -n 5 /etc/passwd:**
```
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
tokyo:x:1001:1001::/home/tokyo:/bin/sh
berlin:x:1002:1002::/home/berlin:/bin/sh
professor:x:1003:1003::/home/professor:/bin/sh
nairobi:x:1004:1006::/home/nairobi:/bin/sh
```

**Observed:** `vim -R` opens file in read-only mode — safe for viewing without accidental edits. tail of /etc/passwd shows Day 09 users still present on server.

---

## Task 3: Understand Permissions

```bash
ls -l devops.txt notes.txt script.sh
```

**Output:**
```
-rw-rw-r-- 1 ubuntu ubuntu  0 Mar 11 23:12 devops.txt
-rw-rw-r-- 1 ubuntu ubuntu 34 Mar 11 23:12 notes.txt
-rw-rw-r-- 1 ubuntu ubuntu 28 Mar 11 23:13 script.sh
```

**Permission breakdown — `rw-rw-r--` = 664:**

```
-  rw-  rw-  r--
↑   ↑    ↑    ↑
│   │    │    └── Others → read only
│   │    └─────── Group  → read + write
│   └──────────── Owner  → read + write
└──────────────── File type → regular file
```

| Who | Permission | Can do |
|-----|-----------|--------|
| Owner (ubuntu) | rw- | Read + Write |
| Group (ubuntu) | rw- | Read + Write |
| Others | r-- | Read only |

**Observed:** No execute permission on any file. script.sh needs `x` to run. devops.txt and notes.txt only need read/write.

---

## Task 4: Modify Permissions

### Make script.sh executable
```bash
chmod +x script.sh
./script.sh
```

**Output:**
```
Hello DevOps
```

### Make devops.txt read-only
```bash
chmod -w devops.txt
```

### Set notes.txt to 640
```bash
chmod 640 notes.txt
```

### Create project directory with 755
```bash
mkdir project
chmod 755 project
```

### Verify all changes
```bash
ls -l devops.txt notes.txt script.sh
ls -ld project
```

**Output:**
```
-r--r--r-- 1 ubuntu ubuntu    0 Mar 11 23:12 devops.txt
-rw-r----- 1 ubuntu ubuntu   34 Mar 11 23:12 notes.txt
-rwxrwxr-x 1 ubuntu ubuntu   28 Mar 11 23:13 script.sh
drwxr-xr-x 2 ubuntu ubuntu 4096 Mar 11 23:22 project
```

**Permission changes summary:**

| File | Before | After | Change |
|------|--------|-------|--------|
| `devops.txt` | rw-rw-r-- | r--r--r-- | Write removed for all |
| `notes.txt` | rw-rw-r-- | rw-r----- | Group read only, others no access |
| `script.sh` | rw-rw-r-- | rwxrwxr-x | Execute added for all |
| `project/` | — | rwxr-xr-x | 755 — owner full, others read+execute |

---

## Task 5: Test Permissions

### Test 1 — Write to read-only file
```bash
echo "testing" >> devops.txt
```

**Output:**
```
-bash: devops.txt: Permission denied
```

**Observed:** Cannot write to read-only file. `-w` flag removed write permission successfully.

### Test 2 — Execute without permission
```bash
chmod -x script.sh
./script.sh
```

**Output:**
```
-bash: ./script.sh: Permission denied
```

**Observed:** Cannot execute script without `x` permission. Error message is clear and immediate.

### Test 3 — Restore execute permission
```bash
chmod +x script.sh
```

**Output:** (silence = success) ✅

---

## Permission Changes Summary

| File | Final Permissions | Octal |
|------|------------------|-------|
| `devops.txt` | r--r--r-- | 444 |
| `notes.txt` | rw-r----- | 640 |
| `script.sh` | rwxrwxr-x | 775 |
| `project/` | rwxr-xr-x | 755 |

---

## Commands Used

| Command | Purpose |
|---------|---------|
| `touch devops.txt` | Create empty file |
| `echo "..." > notes.txt` | Create file with content |
| `vim script.sh` | Create file using text editor |
| `vim -R script.sh` | View file in read-only mode |
| `cat notes.txt` | Read full file |
| `head -n 5 /etc/passwd` | Read first 5 lines |
| `tail -n 5 /etc/passwd` | Read last 5 lines |
| `chmod +x script.sh` | Add execute permission |
| `chmod -w devops.txt` | Remove write permission |
| `chmod 640 notes.txt` | Set exact permissions |
| `chmod 755 project` | Set directory permissions |
| `ls -l` | Check file permissions |
| `ls -ld` | Check directory permissions |
| `./script.sh` | Execute shell script |

---

## What I Learned

- **Permission denied has 2 causes** — either no write permission on a file, or no execute permission on a script. `ls -l` tells you which one instantly
- **`chmod +x` is the most used command** in DevOps — every shell script needs execute permission before it can run
- **Numbers make permission setting precise** — `chmod 640` sets exact permissions in one command instead of multiple `+/-` operations
- **`vim -R`** is safer than `vim` when you just want to read — prevents accidental edits to important files
- **Read-only files** (`chmod -w`) protect important config files from accidental modification — used heavily in production

---
*Completed as part of DevOps Bootcamp — Day 10*
