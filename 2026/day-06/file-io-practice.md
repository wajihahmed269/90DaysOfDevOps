
---

## 1. Create the File

```bash
touch notes.txt
ls -l notes.txt
```

**Output:**
```
-rw-rw-r-- 1 wajih wajih 0 Mar 6 21:12 notes.txt
```

**Observed:** Empty file created successfully. Size is 0 bytes — nothing written yet.

---

## 2. Write First Line (Overwrite with `>`)

```bash
echo "This is my first DevOps note" > notes.txt
```

**Output:** (no output — silence means success)

**Observed:** `>` redirects echo output into the file. Overwrites any existing content. First line written successfully.

---

## 3. Append Second Line (Append with `>>`)

```bash
echo "Learning file I/O on Ubuntu 24.04" >> notes.txt
```

**Output:** (no output — silence means success)

**Observed:** `>>` appends to the end of the file without deleting existing content. Second line added safely.

---

## 4. Write with `tee` (Append + Display)

```bash
echo "Tee writes and displays at the same time" | tee -a notes.txt
```

**Output:**
```
Tee writes and displays at the same time
```

**Observed:** `tee -a` both appended the line to the file AND displayed it on screen simultaneously. The `|` pipe sent echo's output into tee.

---

## 5. Read Full File with `cat`

```bash
cat notes.txt
```

**Output:**
```
This is my first DevOps note
Learning file I/O on Ubuntu 24.04
Tee writes and displays at the same time
head shows top of file
tail shows bottom of file
tee writes and displays simultaneously
greater than sign overwrites file
double greater than sign appends to file
```

**Observed:** All 8 lines present. cat reads and displays the entire file from top to bottom.

---

## 6. Read Top of File with `head`

```bash
head -n 2 notes.txt
```

**Output:**
```
This is my first DevOps note
Learning file I/O on Ubuntu 24.04
```

**Observed:** Only the first 2 lines shown. Useful for checking how a file starts without reading everything.

---

## 7. Read Bottom of File with `tail`

```bash
tail -n 2 notes.txt
```

**Output:**
```
tee writes and displays simultaneously
greater than sign overwrites file
```

**Observed:** Only the last 2 lines shown. In DevOps, `tail -f` is used to watch log files live in real time.

---

## 8. Key Differences Summary

| Method | Symbol/Command | Behaviour |
|--------|---------------|-----------|
| Overwrite | `>` | Deletes existing content, writes new |
| Append | `>>` | Keeps existing content, adds at end |
| Tee append | `tee -a` | Appends to file AND displays on screen |
| Read all | `cat` | Shows entire file |
| Read top | `head -n` | Shows first N lines |
| Read bottom | `tail -n` | Shows last N lines |

---

## 9. Why This Matters for DevOps

- Logs, configs, and scripts are all text files
- `tail -f` watches live logs during incidents
- `tee` saves output to file while still seeing it on screen
- `>` and `>>` are used constantly in shell scripts and automation
- Fast file reading = faster debugging in production

---
*Practice completed as part of DevOps Bootcamp — Day 06*
