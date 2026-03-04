 Day 04 – Linux: Processes, Services & Logs

---

## Commands Practiced

| Category | Commands |
|----------|----------|
| Process | `ps`, `ps aux`, `top`, `htop`, `pgrep` |
| Service | `systemctl status`, `systemctl list-units`, `systemctl restart` |
| Logs | `journalctl -u`, `journalctl -f`, `tail -n` |

---


```bash
systemctl status cron
# Main PID: 1032

sudo systemctl restart cron

systemctl status cron
# Main PID: 36982  ← changed!
```

**Restart = old process killed + new process spawned.**
Not a refresh — a completely new process with a new PID.

Logs confirmed it:
```
Stopped cron.service   ← PID 1032 killed
Started cron.service   ← PID 36982 spawned
```

---

*
