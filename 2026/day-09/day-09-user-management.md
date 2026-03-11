

## Users & Groups Created

### Users
| User | Home Directory | Shell |
|------|---------------|-------|
| `tokyo` | /home/tokyo | /bin/sh |
| `berlin` | /home/berlin | /bin/sh |
| `professor` | /home/professor | /bin/sh |
| `nairobi` | /home/nairobi | /bin/sh |

### Groups
| Group | GID | Members |
|-------|-----|---------|
| `developers` | 1004 | tokyo, berlin |
| `admins` | 1005 | berlin, professor |
| `project-team` | — | nairobi, tokyo |

---

## Task 1: Create Users

```bash
sudo useradd -m tokyo
sudo passwd tokyo

sudo useradd -m berlin
sudo passwd berlin

sudo useradd -m professor
sudo passwd professor
```

**Verification:**
```bash
cat /etc/passwd | grep -E "tokyo|berlin|professor"
```

**Output:**
```
tokyo:x:1001:1001::/home/tokyo:/bin/sh
berlin:x:1002:1002::/home/berlin:/bin/sh
professor:x:1003:1003::/home/professor:/bin/sh
```

```bash
ls -l /home
```

**Output:**
```
drwxr-x--- 2 berlin    berlin    4096 Mar 11 00:06 berlin
drwxr-x--- 2 professor professor 4096 Mar 11 00:06 professor
drwxr-x--- 2 tokyo     tokyo     4096 Mar 11 00:05 tokyo
drwxr-x--- 4 ubuntu    ubuntu    4096 Mar 11 00:05 ubuntu
```

**Observed:** All 3 users created with home directories. Each user has UID starting from 1001. Passwords set successfully.

---

## Task 2: Create Groups

```bash
sudo groupadd developers
sudo groupadd admins
```

**Verification:**
```bash
cat /etc/group | grep -E "developers|admins"
```

**Output:**
```
developers:x:1004:
admins:x:1005:
```

**Observed:** Both groups created with GIDs 1004 and 1005. Member list empty — users not yet assigned.

---

## Task 3: Assign Users to Groups

```bash
sudo usermod -aG developers tokyo
sudo usermod -aG developers,admins berlin
sudo usermod -aG admins professor
```

**Verification:**
```bash
groups tokyo
groups berlin
groups professor
```

**Output:**
```
tokyo     : tokyo developers
berlin    : berlin developers admins
professor : professor admins
```

**Observed:** tokyo assigned to developers. berlin assigned to both developers and admins. professor assigned to admins. The `-aG` flag appends groups without removing existing ones.

---

## Task 4: Shared Directory — /opt/dev-project

```bash
sudo mkdir /opt/dev-project
sudo chgrp developers /opt/dev-project
sudo chmod 775 /opt/dev-project
```

**Verification:**
```bash
ls -ld /opt/dev-project
```

**Output:**
```
drwxrwxr-x 2 root developers 4096 Mar 11 00:14 /opt/dev-project
```

**Testing file creation:**
```bash
sudo -u tokyo touch /opt/dev-project/tokyo-file.txt
sudo -u berlin touch /opt/dev-project/berlin-file.txt
ls -l /opt/dev-project
```

**Output:**
```
-rw-rw-r-- 1 berlin berlin 0 Mar 11 00:37 berlin-file.txt
-rw-rw-r-- 1 tokyo  tokyo  0 Mar 11 00:36 tokyo-file.txt
```

**Observed:** Directory created with 775 permissions. Group set to developers. Both tokyo and berlin (developers members) successfully created files. Permission 775 means owner=rwx, group=rwx, others=r-x.

---

## Task 5: Team Workspace — /opt/team-workspace

```bash
sudo useradd -m nairobi
sudo passwd nairobi
sudo groupadd project-team
sudo usermod -aG project-team nairobi
sudo usermod -aG project-team tokyo
sudo mkdir /opt/team-workspace
sudo chgrp project-team /opt/team-workspace
sudo chmod 775 /opt/team-workspace
sudo -u nairobi touch /opt/team-workspace/nairobi-file.txt
```

**Verification:**
```bash
groups nairobi
groups tokyo
ls -ld /opt/team-workspace
ls -l /opt/team-workspace
```

**Output:**
```
nairobi : nairobi project-team
tokyo   : tokyo developers project-team
drwxrwxr-x 2 root project-team 4096 Mar 11 00:40 /opt/team-workspace
-rw-rw-r-- 1 nairobi nairobi 0 Mar 11 00:40 nairobi-file.txt
```

**Observed:** nairobi created and added to project-team. tokyo added to project-team (now in 2 groups). Workspace created with 775 permissions. nairobi successfully created a file — group permissions working correctly.

---

## Group Assignments Summary

| User | Primary Group | Additional Groups |
|------|--------------|------------------|
| `tokyo` | tokyo | developers, project-team |
| `berlin` | berlin | developers, admins |
| `professor` | professor | admins |
| `nairobi` | nairobi | project-team |

---

## Directories Created

| Directory | Owner | Group | Permissions |
|-----------|-------|-------|------------|
| `/opt/dev-project` | root | developers | 775 (drwxrwxr-x) |
| `/opt/team-workspace` | root | project-team | 775 (drwxrwxr-x) |

---

## Commands Used

| Command | Purpose |
|---------|---------|
| `sudo useradd -m username` | Create user with home directory |
| `sudo passwd username` | Set user password |
| `sudo groupadd groupname` | Create a new group |
| `sudo usermod -aG group user` | Add user to group (append) |
| `sudo mkdir /path` | Create directory |
| `sudo chgrp group /path` | Change group owner of directory |
| `sudo chmod 775 /path` | Set directory permissions |
| `sudo -u username command` | Run command as different user |
| `groups username` | Check user's group memberships |
| `cat /etc/passwd` | View all users |
| `cat /etc/group` | View all groups |
| `ls -ld /path` | Check directory permissions |

---

## What I Learned

- **`-aG` flag is critical** — without `-a`, usermod replaces all groups instead of appending. Always use `-aG` together, never just `-G`
- **Groups make permission management easy** — set permissions once on a directory for a group, all members automatically get access
- **775 permissions** — perfect for shared team directories. Group members can read and write, others can only read
- **`/etc/passwd` and `/etc/group`** are the source of truth for users and groups on any Linux system
- **`sudo -u username`** lets you test permissions as a different user without logging in as them — very useful for verifying setups in production

---
*Completed as part of DevOps Bootcamp — Day 09*
