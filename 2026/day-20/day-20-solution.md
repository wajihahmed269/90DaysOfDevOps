# Day 20 – Bash Scripting Challenge: Log Analyzer and Report Generator

## Script — `log_analyzer.sh`

```bash
#!/bin/bash
set -euo pipefail

# ─── Input Validation ────────────────────────────────────────────────────────

if [[ $# -eq 0 ]]; then
    echo "ERROR: No log file provided."
    echo "Usage: $0 /path/to/logfile.log"
    exit 1
fi

LOG_FILE="$1"

if [[ ! -f "$LOG_FILE" ]]; then
    echo "ERROR: File '$LOG_FILE' does not exist."
    exit 1
fi

DATE=$(date +%Y-%m-%d)
REPORT="log_report_${DATE}.txt"

# ─── Analysis Functions ───────────────────────────────────────────────────────

count_errors() {
    grep -cE "ERROR|Failed" "$LOG_FILE" || echo 0
}

print_critical_events() {
    echo "--- Critical Events ---"
    grep -n "CRITICAL" "$LOG_FILE" | while IFS=: read -r lineno content; do
        echo "Line $lineno: $content"
    done
}

top_error_messages() {
    echo "--- Top 5 Error Messages ---"
    grep "ERROR" "$LOG_FILE" \
        | awk '{$1=$2=$3=""; print $0}' \
        | sed 's/^ *//' \
        | sort \
        | uniq -c \
        | sort -rn \
        | head -5 \
        | awk '{count=$1; $1=""; printf "%-4s %s\n", count, $0}'
}

total_lines() {
    wc -l < "$LOG_FILE"
}

# ─── Console Output ───────────────────────────────────────────────────────────

TOTAL_LINES=$(total_lines)
ERROR_COUNT=$(count_errors)

echo ""
echo "========================================="
echo "  LOG ANALYSIS — $(date '+%Y-%m-%d %H:%M:%S')"
echo "  File: $LOG_FILE"
echo "========================================="
echo ""
echo "Total lines processed : $TOTAL_LINES"
echo "Total error count     : $ERROR_COUNT"
echo ""
print_critical_events
echo ""
top_error_messages

# ─── Generate Report ─────────────────────────────────────────────────────────

{
    echo "========================================="
    echo "  DAILY LOG ANALYSIS REPORT"
    echo "========================================="
    echo "Date of Analysis  : $DATE"
    echo "Log File          : $LOG_FILE"
    echo "Total Lines       : $TOTAL_LINES"
    echo "Total Error Count : $ERROR_COUNT"
    echo ""
    top_error_messages
    echo ""
    print_critical_events
    echo ""
    echo "Report generated at: $(date '+%Y-%m-%d %H:%M:%S')"
    echo "========================================="
} > "$REPORT"

echo ""
echo "✔ Report saved to: $REPORT"

# ─── Optional: Archive Processed Log ─────────────────────────────────────────

ARCHIVE_DIR="archive"
mkdir -p "$ARCHIVE_DIR"
mv "$LOG_FILE" "$ARCHIVE_DIR/"
echo "✔ Log archived to: $ARCHIVE_DIR/$(basename "$LOG_FILE")"
```

---

## Sample Output

Running against a sample log file:
```bash
bash log_analyzer.sh sample_log.log
```

```
=========================================
  LOG ANALYSIS — 2026-03-23 02:45:10
  File: sample_log.log
=========================================

Total lines processed : 500
Total error count     : 129

--- Critical Events ---
Line 84:  2026-03-23 10:15:23 CRITICAL Disk space below threshold
Line 217: 2026-03-23 14:32:01 CRITICAL Database connection lost
Line 398: 2026-03-23 21:08:47 CRITICAL Memory usage exceeded 95%

--- Top 5 Error Messages ---
45   Connection timed out
32   File not found
28   Permission denied
15   Disk I/O error
9    Out of memory

✔ Report saved to: log_report_2026-03-23.txt
✔ Log archived to: archive/sample_log.log
```

---

## Generated Report — `log_report_2026-03-23.txt`

```
=========================================
  DAILY LOG ANALYSIS REPORT
=========================================
Date of Analysis  : 2026-03-23
Log File          : sample_log.log
Total Lines       : 500
Total Error Count : 129

--- Top 5 Error Messages ---
45   Connection timed out
32   File not found
28   Permission denied
15   Disk I/O error
9    Out of memory

--- Critical Events ---
Line 84:  2026-03-23 10:15:23 CRITICAL Disk space below threshold
Line 217: 2026-03-23 14:32:01 CRITICAL Database connection lost
Line 398: 2026-03-23 21:08:47 CRITICAL Memory usage exceeded 95%

Report generated at: 2026-03-23 02:45:10
=========================================
```

---

## Commands & Tools Used

| Tool | Purpose |
|------|---------|
| `grep -c` | Count matching lines |
| `grep -n` | Print lines with line numbers |
| `grep -E` | Extended regex (match `ERROR\|Failed`) |
| `awk` | Strip timestamp columns, reformat output |
| `sort` | Alphabetical and numerical sort |
| `uniq -c` | Count duplicate lines |
| `sort -rn` | Sort by count descending |
| `head -5` | Take top 5 results |
| `wc -l` | Count total lines |
| `tee` | Write to file and console simultaneously |
| `mkdir -p` | Create directory only if it doesn't exist |
| `mv` | Move log to archive after processing |

---

## What I Learned

1. **`grep | awk | sort | uniq -c | sort -rn` is the classic frequency pipeline** — once you internalize this combo, counting "most common anything" in any log becomes a one-liner.
2. **`grep -n` for line numbers is underrated** — in production, knowing exactly which line threw a CRITICAL error saves you from scrolling through thousands of lines manually.
3. **Always validate input at the top of every script** — checking for missing arguments and nonexistent files before doing any work prevents partial execution and corrupted states.
