# Day 27 – GitHub Profile Makeover: Build Your Developer Identity

## Task 1: Profile Audit

Visited my own profile as a stranger and assessed honestly:

| Area | Before | Status |
|------|--------|--------|
| Profile picture | Generic / not set | ❌ Needed update |
| Bio | Empty | ❌ Needed update |
| Pinned repos | Random forks, no descriptions | ❌ Needed cleanup |
| Repo descriptions | Most blank | ❌ Needed update |
| README | None | ❌ Needed to create |

**Honest assessment:** A recruiter landing on my profile would have no idea what I do, what I'm learning, or what my best work is. The repos look abandoned even though I've been actively building.

---

## Task 2: Profile README

Created repository: `wajihahmed269/wajihahmed269`

File: `README.md`

```markdown
# Hey, I'm Wajih 👋

I'm a DevOps student currently working through the **#90DaysOfDevOps** challenge by Shubham Londhe.
Building real skills in Linux, shell scripting, Git, and cloud infrastructure — one day at a time.

---

## 🔧 What I'm Working On

- 📅 **90 Days of DevOps** — actively progressing through daily challenges
- 🐚 Shell scripting projects: log analyzers, backup scripts, system info reporters
- 🌿 Git & GitHub: branching strategies, GitHub CLI, workflow automation
- ☁️ Coming next: Docker, Kubernetes, CI/CD, Terraform

---

## 🛠️ Skills & Tools

**Linux:** File system, permissions, users/groups, LVM, networking, process management  
**Shell:** Bash scripting, functions, strict mode, cron jobs, text processing (grep, awk, sed)  
**Git:** Branching, merge, rebase, stash, cherry-pick, reset, revert  
**GitHub:** Pull requests, issues, GitHub CLI, profile management  
**Python:** Basics, scripting, file I/O, APIs (learning)  
**Cloud:** AWS EC2, Ubuntu server setup (learning)

---

## 📁 Key Repositories

| Repo | What's Inside |
|------|--------------|
| [90DaysOfDevOps](https://github.com/wajihahmed269/90DaysOfDevOps) | Daily submissions, notes, and scripts from the challenge |
| [shell-scripts](https://github.com/wajihahmed269/shell-scripts) | Bash scripts: system info, log rotation, backup, log analyzer |
| [python-scripts](https://github.com/wajihahmed269/python-scripts) | Python projects from Days 7–15 |
| [devops-notes](https://github.com/wajihahmed269/devops-notes) | Cheat sheets, references: shell scripting, Git commands |

---

## 📬 Connect

- 💼 [LinkedIn](https://linkedin.com/in/wajih-ahmed-041235391)
- 🐙 GitHub: [@wajihahmed269](https://github.com/wajihahmed269)

---

*Week 4 of 90 — keeping the streak going.*
```

---

## Task 3: Repository Organization

### 90DaysOfDevOps

**Structure:**
```
90DaysOfDevOps/
├── README.md
└── 2026/
    ├── day-01/
    ├── day-02/
    ...
    └── day-27/
```

**README.md:**
```markdown
# 90 Days of DevOps — Wajih's Fork

My daily submissions for the #90DaysOfDevOps challenge by @shubhamlondhe1996.

Each folder contains:
- A markdown notes/solution file
- Any scripts written that day
- Screenshots where required

**Progress:** Day 27 / 90 ✅

Following: https://github.com/LondheShubham153/90DaysOfDevOps
```

---

### shell-scripts

```markdown
# Shell Scripts

Bash scripts written during Days 16–21 of #90DaysOfDevOps.

| Script | Description |
|--------|-------------|
| `functions.sh` | Function basics — greet, add |
| `disk_check.sh` | Disk and memory usage reporter |
| `strict_demo.sh` | Demonstrates set -euo pipefail behavior |
| `local_demo.sh` | Local vs global variable scoping |
| `system_info.sh` | Full system info reporter with functions |
| `log_rotate.sh` | Compress logs older than 7d, delete .gz older than 30d |
| `backup.sh` | Timestamped tar.gz backup with cleanup |
| `maintenance.sh` | Calls log rotation + backup, logs to /var/log/maintenance.log |
| `log_analyzer.sh` | Analyzes log files, reports errors and CRITICAL events |
```

---

### python-scripts

```markdown
# Python Scripts

Python projects from Days 7–15 of #90DaysOfDevOps.

Covers: variables, functions, file I/O, OOP basics, error handling, and scripting.
```

---

### devops-notes

```
devops-notes/
├── README.md
├── linux/
│   └── linux-commands.md
├── shell/
│   └── shell_scripting_cheatsheet.md
└── git/
    └── git-commands.md
```

---

## Task 4: Pinned Repositories

Selected 6 pinned repos:

1. `90DaysOfDevOps` — main challenge repo, shows commitment
2. `shell-scripts` — practical automation work
3. `log-analyzer` (Day 20 script promoted to its own repo)
4. `python-scripts` — shows Python learning
5. `devops-notes` — cheat sheets, useful to others
6. `devops-git-practice` — Git workflow practice from Days 22–26

Each has a clear description and README.

---

## Task 5: Cleanup Done

- ✅ Deleted 2 empty repos created during testing
- ✅ Renamed `test1` → deleted, `myrepo` → deleted
- ✅ Checked commit history on all repos — no `.env` files, no API keys, no passwords exposed
- ✅ Added `.gitignore` to active repos

---

## Task 6: Before & After

### 3 Things I Improved

1. **Profile README** — Before: nothing. After: a clear intro, what I'm learning, links to my best repos, and how to reach me. Any recruiter can now understand who I am in 30 seconds.

2. **Repo descriptions** — Before: every repo was blank. After: every pinned repo has a one-line description and a proper README. A blank description signals an abandoned project.

3. **Pinned repos** — Before: random forks that didn't represent my actual work. After: 6 curated repos that tell the story of what I've been building over the last 27 days.

> Screenshots: taken before starting Task 1 and after completing Task 5 — added to `2026/day-27/` folder.
