# Day 19 – Shell Scripting Project: Log Rotation, Backup & Crontab

## Task 1: Log Rotation Script — `log_rotate.sh`

```bash
#!/bin/bash
set -euo pipefail

LOG_DIR="${1:-}"

if [[ -z "$LOG_DIR" ]]; then
    echo "ERROR: No log directory provided."
    echo "Usage: $0 /path/to/log/dir"
    exit 1
fi

if [[ ! -d "$LOG_DIR" ]]; then
    echo "ERROR: Directory '$LOG_DIR' does not exist."
    exit 1
fi

echo "=== Log Rotation: $LOG_DIR ==="
echo "Timestamp: $(date '+%Y-%m-%d %H:%M:%S')"
echo ""

# Compress .log files older than 7 days
COMPRESSED=0
while IFS= read -r -d '' file; do
    gzip "$file" && echo "Compressed: $file"
    (( COMPRESSED++ ))
done < <(find "$LOG_DIR" -name "*.log" -mtime +7 -print0)

# Delete .gz files older than 30 days
DELETED=0
while IFS= read -r -d '' file; do
    rm "$file" && echo "Deleted: $file"
    (( DELETED++ ))
done < <(find "$LOG_DIR" -name "*.gz" -mtime +30 -print0)

echo ""
echo "--- Summary ---"
echo "Files compressed : $COMPRESSED"
echo "Files deleted    : $DELETED"
```

**Sample Output:**
```
=== Log Rotation: /var/log/myapp ===
Timestamp: 2026-03-23 02:30:00

Compressed: /var/log/myapp/app-2026-03-14.log
Compressed: /var/log/myapp/app-2026-03-15.log
Deleted: /var/log/myapp/app-2026-02-10.log.gz

--- Summary ---
Files compressed : 2
Files deleted    : 1
```

---

## Task 2: Server Backup Script — `backup.sh`

```bash
#!/bin/bash
set -euo pipefail

SOURCE="${1:-}"
DEST="${2:-}"

if [[ -z "$SOURCE" || -z "$DEST" ]]; then
    echo "ERROR: Missing arguments."
    echo "Usage: $0 /source/dir /backup/dest"
    exit 1
fi

if [[ ! -d "$SOURCE" ]]; then
    echo "ERROR: Source directory '$SOURCE' does not exist."
    exit 1
fi

mkdir -p "$DEST"

TIMESTAMP=$(date +%Y-%m-%d)
ARCHIVE_NAME="backup-${TIMESTAMP}.tar.gz"
ARCHIVE_PATH="${DEST}/${ARCHIVE_NAME}"

echo "=== Server Backup ==="
echo "Source      : $SOURCE"
echo "Destination : $DEST"
echo "Archive     : $ARCHIVE_NAME"
echo ""

tar -czf "$ARCHIVE_PATH" "$SOURCE"

if [[ -f "$ARCHIVE_PATH" ]]; then
    SIZE=$(du -sh "$ARCHIVE_PATH" | cut -f1)
    echo "✔ Archive created: $ARCHIVE_NAME"
    echo "✔ Archive size   : $SIZE"
else
    echo "ERROR: Archive was not created."
    exit 1
fi

# Delete backups older than 14 days
PRUNED=0
while IFS= read -r -d '' old; do
    rm "$old" && echo "Pruned old backup: $(basename "$old")"
    (( PRUNED++ ))
done < <(find "$DEST" -name "backup-*.tar.gz" -mtime +14 -print0)

echo ""
echo "Old backups pruned: $PRUNED"
echo "Backup complete."
```

**Sample Output:**
```
=== Server Backup ===
Source      : /home/wajih/projects
Destination : /backups
Archive     : backup-2026-03-23.tar.gz

✔ Archive created: backup-2026-03-23.tar.gz
✔ Archive size   : 142M

Old backups pruned: 1
Backup complete.
```

---

## Task 3: Crontab — Syntax & Entries

### Cron Syntax

```
* * * * *  command
│ │ │ │ │
│ │ │ │ └── Day of week (0-7, 0 and 7 = Sunday)
│ │ │ └──── Month (1-12)
│ │ └────── Day of month (1-31)
│ └──────── Hour (0-23)
└────────── Minute (0-59)
```

### Cron Entries

```bash
# Run log_rotate.sh every day at 2 AM
0 2 * * * /home/wajih/scripts/log_rotate.sh /var/log/myapp >> /var/log/log_rotate.log 2>&1

# Run backup.sh every Sunday at 3 AM
0 3 * * 0 /home/wajih/scripts/backup.sh /home/wajih/projects /backups >> /var/log/backup.log 2>&1

# Run health check every 5 minutes
*/5 * * * * /home/wajih/scripts/health_check.sh >> /var/log/health_check.log 2>&1
```

> To apply: `crontab -e` — then paste the entries above.

---

## Task 4: Scheduled Maintenance Script — `maintenance.sh`

```bash
#!/bin/bash
set -euo pipefail

LOGFILE="/var/log/maintenance.log"
LOG_DIR="/var/log/myapp"
BACKUP_SRC="/home/wajih/projects"
BACKUP_DEST="/backups"

log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') | $1" | tee -a "$LOGFILE"
}

run_log_rotation() {
    log "START: Log rotation"
    bash /home/wajih/scripts/log_rotate.sh "$LOG_DIR" >> "$LOGFILE" 2>&1
    log "END: Log rotation"
}

run_backup() {
    log "START: Backup"
    bash /home/wajih/scripts/backup.sh "$BACKUP_SRC" "$BACKUP_DEST" >> "$LOGFILE" 2>&1
    log "END: Backup"
}

main() {
    log "====== Maintenance Started ======"
    run_log_rotation
    run_backup
    log "====== Maintenance Complete ======"
}

main
```

**Cron entry to run daily at 1 AM:**
```bash
0 1 * * * /home/wajih/scripts/maintenance.sh
```

**Sample `/var/log/maintenance.log`:**
```
2026-03-23 01:00:01 | ====== Maintenance Started ======
2026-03-23 01:00:01 | START: Log rotation
2026-03-23 01:00:03 | END: Log rotation
2026-03-23 01:00:03 | START: Backup
2026-03-23 01:00:47 | END: Backup
2026-03-23 01:00:47 | ====== Maintenance Complete ======
```

---

## What I Learned

1. **`find` with `-mtime` is the backbone of log management** — combining it with `-exec gzip` or `rm` gives you surgical control over old files without manual intervention.
2. **Crontab is simple but easy to break silently** — always redirect both stdout and stderr (`>> log 2>&1`) so you can debug cron jobs that don't run as expected.
3. **Scripts that accept arguments are infinitely more reusable** — hardcoding paths is a trap; parameterizing source/dest turns a one-use script into a tool you can run anywhere.
