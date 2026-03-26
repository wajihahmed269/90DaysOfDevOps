# Day 28 – Revision Day: Everything from Day 1 to Day 27

## Task 1: Self-Assessment Checklist

### Linux

| Topic | Status |
|-------|--------|
| Navigate file system, create/move/delete files | ✅ Confident |
| Manage processes — list, kill, background/foreground | ✅ Confident |
| Work with systemd — start, stop, enable, status | ✅ Confident |
| Read and edit files with vi/vim or nano | ✅ Confident |
| Troubleshoot CPU, memory, disk — top, free, df, du | ✅ Confident |
| Explain Linux file system hierarchy | ✅ Confident |
| Create users and groups, manage passwords | ✅ Confident |
| Set file permissions with chmod (numeric + symbolic) | ✅ Confident |
| Change ownership with chown and chgrp | ✅ Confident |
| Create and manage LVM volumes | 🔄 Need to revisit |
| Check network — ping, curl, netstat, ss, dig, nslookup | ✅ Confident |
| Explain DNS, IP addressing, subnets, common ports | 🔄 Need to revisit |

### Shell Scripting

| Topic | Status |
|-------|--------|
| Write script with variables, arguments, user input | ✅ Confident |
| if/elif/else and case statements | ✅ Confident |
| for, while, until loops | ✅ Confident |
| Functions with arguments and return values | ✅ Confident |
| grep, awk, sed, sort, uniq for text processing | ✅ Confident |
| Error handling — set -e, -u, pipefail, trap | ✅ Confident |
| Schedule scripts with crontab | ✅ Confident |

### Git & GitHub

| Topic | Status |
|-------|--------|
| Init, stage, commit, view history | ✅ Confident |
| Create and switch branches | ✅ Confident |
| Push to and pull from GitHub | ✅ Confident |
| Explain clone vs fork | ✅ Confident |
| Merge — fast-forward vs merge commit | ✅ Confident |
| Rebase and when to use vs merge | ✅ Confident |
| git stash and git stash pop | ✅ Confident |
| Cherry-pick a commit | ✅ Confident |
| Squash merge vs regular merge | ✅ Confident |
| git reset (soft, mixed, hard) and git revert | ✅ Confident |
| GitFlow, GitHub Flow, Trunk-Based Development | ✅ Confident |
| GitHub CLI — repos, PRs, issues | ✅ Confident |

---

## Task 2: Revisiting Weak Spots

### Weak Spot 1: LVM (Day 13)

Re-did the hands-on commands:

```bash
# Physical Volume
sudo pvcreate /dev/sdb
sudo pvdisplay

# Volume Group
sudo vgcreate myvg /dev/sdb
sudo vgdisplay

# Logical Volume
sudo lvcreate -L 5G -n mylv myvg
sudo lvdisplay

# Format and mount
sudo mkfs.ext4 /dev/myvg/mylv
sudo mount /dev/myvg/mylv /mnt/data

# Extend LV if you add more disk
sudo lvextend -L +2G /dev/myvg/mylv
sudo resize2fs /dev/myvg/mylv
```

**What I re-learned:** The key advantage of LVM is flexibility — you can extend or shrink logical volumes without downtime or repartitioning. Physical partitions are rigid; LVM is dynamic. In cloud environments, this matters when you need to scale storage on running servers.

---

### Weak Spot 2: DNS and Subnets (Day 15)

Re-ran the diagnostic commands:

```bash
# DNS lookup
dig google.com
nslookup google.com 8.8.8.8

# Check open ports
ss -tulnp
netstat -tulnp

# Trace route
traceroute google.com

# Check which process uses port 8080
sudo lsof -i :8080
sudo ss -tulnp | grep 8080
```

**What I re-learned:** DNS resolution chain — browser checks local cache → `/etc/hosts` → `/etc/resolv.conf` → recursive DNS server → authoritative DNS server. Subnetting: a /24 gives 254 usable hosts, /25 gives 126, /28 gives 14. CIDR notation is just the number of fixed bits in the network portion.

---

### Weak Spot 3: git reflog as a recovery tool

```bash
# See everything Git has done — even after hard reset
git reflog

# Output example:
# f3c1a2b (HEAD -> main) HEAD@{0}: reset: moving to HEAD~1
# a9b2c3d HEAD@{1}: commit: Commit C
# b7e1f2c HEAD@{2}: commit: Commit B

# Recover a commit that was hard-reset
git reset --hard a9b2c3d
```

**What I re-learned:** `git reflog` is the safety net below the safety net. Even after `git reset --hard`, commits aren't immediately garbage collected. Reflog keeps a log of every HEAD movement for 90 days by default — you can recover almost anything if you act fast.

---

## Task 3: Quick-Fire Answers

**1. What does `chmod 755 script.sh` do?**
Sets permissions so the owner can read, write, and execute; group and others can read and execute. Numeric: owner=7(rwx), group=5(r-x), others=5(r-x). Standard permission for executable scripts.

**2. What is the difference between a process and a service?**
A process is any running program — identified by a PID. A service is a process managed by systemd (or init) — it has start/stop/restart lifecycle management, auto-restarts on failure, and runs as a daemon. All services are processes, but not all processes are services.

**3. How do you find which process is using port 8080?**
```bash
sudo ss -tulnp | grep 8080
# or
sudo lsof -i :8080
```

**4. What does `set -euo pipefail` do in a shell script?**
- `-e` — exit immediately if any command fails
- `-u` — exit if an undefined variable is used
- `-o pipefail` — fail if any command in a pipe fails, not just the last one
Together they turn silent failures into loud crashes — which is what you want in production scripts.

**5. What is the difference between `git reset --hard` and `git revert`?**
`reset --hard` moves the branch pointer back and permanently deletes commits + changes — rewrites history. `git revert` creates a new commit that undoes the changes — history is preserved. Reset is for local cleanup, revert is safe for shared/pushed branches.

**6. Which branching strategy for a team of 5 shipping weekly?**
GitHub Flow. It's simple — branch off main, PR, merge, deploy. No release branches, no develop branch overhead. Weekly shipping cadence fits perfectly with short-lived feature branches.

**7. What does `git stash` do and when would you use it?**
Saves your current work-in-progress (staged and unstaged) to a temporary stack so you can switch branches with a clean working tree. Use it when you need to urgently switch context mid-feature without committing half-done work.

**8. How do you schedule a script to run every day at 3 AM?**
```bash
crontab -e
# Add:
0 3 * * * /path/to/script.sh >> /var/log/script.log 2>&1
```

**9. What is the difference between `git fetch` and `git pull`?**
`git fetch` downloads changes from the remote but doesn't merge them — you stay on your current commit and can review before integrating. `git pull` does fetch + merge in one step. Fetch is safer in team environments.

**10. What is LVM and why use it instead of regular partitions?**
LVM (Logical Volume Manager) is an abstraction layer over disk partitions. Unlike regular partitions (fixed size, require downtime to resize), LVM volumes can be extended while mounted, span multiple physical disks, and be snapshotted for backups. Essential for servers where storage needs change over time.

---

## Task 4: Work Organization Check

- ✅ Days 1–27 all committed and pushed to fork
- ✅ `git-commands.md` updated through Day 26 (includes gh CLI commands)
- ✅ `shell_scripting_cheatsheet.md` complete — Day 21
- ✅ GitHub profile cleaned up — Day 27
- ✅ All repos have descriptions and READMEs

---

## Task 5: Teach It Back — Git Branching for a Non-Developer

**"What is Git branching and why do developers use it?"**

Imagine you're writing a book. You have a finished draft — that's your `main` branch. Now you want to try a completely different ending without messing up the original. You make a photocopy and start writing the alternative ending on that. The original stays safe.

That photocopy is a branch in Git.

When you're happy with the new ending, you "merge" it back into the original — Git brings the two versions together. If both versions changed the same paragraph, Git asks you to decide which one to keep. That's a merge conflict.

In real software teams, every new feature, bug fix, or experiment gets its own branch. The `main` branch always stays clean and working — it's what users see. Developers only merge their branch into main after it's reviewed and tested.

This is how a team of 50 engineers can all work on the same codebase at the same time without stepping on each other.
