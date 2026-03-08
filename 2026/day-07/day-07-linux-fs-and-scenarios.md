
---

## Part 1: Linux File System Hierarchy

---

### / (Root Directory)
The starting point of everything in Linux. Every file and folder on the system branches out from here.

**Command run:**
```bash
ls -l /
```

**Output snapshot:**
```
lrwxrwxrwx   root root   bin -> usr/bin
drwxr-xr-x   root root   etc
drwxr-xr-x   root root   home
drwx------   root root   root
drwxrwxrwt   root root   tmp
drwxr-xr-x   root root   usr
drwxr-xr-x   root root   var
```



### /etc – Configuration Files
Stores configuration files for every service, tool, and program on the system. If something is broken, check here first.

**Command run:**
```bash
ls -l /etc | head -20
cat /etc/hostname
```

**Output snapshot:**
```
drwxr-xr-x  root  apparmor
drwxr-xr-x  root  apt
-rw-r--r--  root  bash.bashrc
drwxr-xr-x  root  chrony

hostname: ip-172-31-38-212
```



### /var/log – Log Files
The most important directory for DevOps. Every service writes its activity here — crashes, logins, errors, and more.

**Command run:**
```bash
ls -l /var/log | head -20
du -sh /var/log/* 2>/dev/null | sort -h | tail -5
```

**Output snapshot:**
```
-rw-r-----  syslog  auth.log
-rw-r-----  syslog  cloud-init.log
-rw-r-----  root    dmesg
drwxr-sr-x  root    journal

Largest logs:
48K    /var/log/dmesg.0
180K   /var/log/kern.log.1
344K   /var/log/cloud-init.log
412K   /var/log/syslog.1
33M    /var/log/journal
```


---

### /tmp – Temporary Files
A shared scratch space. Anyone can write files here but files are wiped on every reboot.

**Command run:**
```bash
ls -l /tmp
```

**Output snapshot:**
```
drwx------  root  snap-private-tmp
drwx------  root  systemd-private-...-chrony.service
drwx------  root  systemd-private-...-ModemManager.service
```

**Observed:** Only system services using private temp folders. Empty on a fresh server. The sticky bit (drwxrwxrwt) means everyone can write but cannot delete each other's files.

**I would use this when:** Storing throwaway files during a task — test scripts, temp downloads, quick demos like our runbook-demo from Day 05.

---

### /home – User Home Directories
Each user on the system gets their own folder here. Stores personal files, hidden config files, and SSH keys.

**Command run:**
```bash
ls -la ~
```

**Output snapshot:**
```
-rw-------  ubuntu  .bash_history
-rw-r--r--  ubuntu  .bashrc
-rw-r--r--  ubuntu  .profile
drwx------  ubuntu  .ssh
drwx------  ubuntu  .cache
```

**Observed:** Hidden files (starting with .) store user settings. .ssh folder is locked (drwx------) — only the owner can access SSH keys. .bash_history records every command ever typed.

**I would use this when:** Storing my scripts, checking my shell config, or managing SSH keys for server access.

---

### /root – Root User's Home Directory
The home directory of the root (admin) user. Completely locked from regular users.

**Command run:**
```bash
ls -l /root        # Permission denied
sudo ls -l /root   # Works with sudo
```

**Output snapshot:**
```
ls: cannot open directory '/root': Permission denied

sudo output:
drwx------ 3 root root 4096 snap
```

**Observed:** Regular ubuntu user cannot access /root at all. Only sudo (temporary root access) allowed entry. This is Linux security working correctly.

**I would use this when:** Never directly — always use sudo for admin tasks instead of entering /root.

---

### /bin and /usr/bin – Command Binaries
Where all Linux commands live as executable files. When you type ls or cat, Linux finds them here.

**Command run:**
```bash
ls -l /bin
ls -l /usr/bin | head -15
```

**Output snapshot:**
```
/bin → usr/bin  (symlink — merged in modern Ubuntu)

/usr/bin contents:
-rwxr-xr-x  root  VGAuthService
-rwxr-xr-x  root  aa-enabled
-rwxr-xr-x  root  add-apt-repository
-rwxr-xr-x  root  apport-cli
```

**Observed:** /bin is just a symlink to /usr/bin in Ubuntu 24.04. Hundreds of executable files (-rwxr-xr-x). The x permission is what makes them runnable as commands.

**I would use this when:** Checking if a command exists on the system, or understanding where a tool is installed.

---

### /opt – Optional/Third-Party Applications
Used for manually installed or third-party software that doesn't belong in standard system directories.

**Command run:**
```bash
ls -l /opt
```

**Output snapshot:**
```
total 0
(empty)
```

**Observed:** Empty on this fresh AWS server. In production you'd find Jenkins, Docker Desktop, or custom company tools installed here.

**I would use this when:** Installing third-party software like Jenkins or a custom monitoring agent that needs its own isolated directory.

---

## Quick Reference Summary

| Directory | Contains | Use When |
|-----------|----------|----------|
| `/` | Everything | First look on unknown server |
| `/etc` | Config files | Service is broken or needs config change |
| `/var/log` | Log files | Something crashed or behaved unexpectedly |
| `/tmp` | Temp files | Need throwaway workspace |
| `/home` | User files | Managing personal scripts and SSH keys |
| `/root` | Root's home | Never directly — use sudo |
| `/usr/bin` | Commands | Checking if a tool is installed |
| `/opt` | Third-party apps | Installing custom software |

---
*Notes completed as part of DevOps Bootcamp — Day 07 Part 1*
