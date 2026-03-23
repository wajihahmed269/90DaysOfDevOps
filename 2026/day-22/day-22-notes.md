# Day 22 – Introduction to Git: Your First Repository

## Task 1: Install and Configure Git

```bash
# Verify Git is installed
git --version
# git version 2.43.0

# Set your identity
git config --global user.name "Wajih Ahmed"
git config --global user.email "your@email.com"

# Verify configuration
git config --list
```

**Output:**
```
user.name=Wajih Ahmed
user.email=your@email.com
core.editor=vim
```

---

## Task 2: Create Your Git Project

```bash
mkdir devops-git-practice
cd devops-git-practice
git init
git status
```

**Output of `git init`:**
```
Initialized empty Git repository in /home/wajih/devops-git-practice/.git/
```

**Output of `git status`:**
```
On branch main

No commits yet

nothing to commit (create/copy files and use "git commit")
```

### Exploring `.git/` Directory

```bash
ls -la .git/
```

```
drwxrwxr-x  7 wajih wajih 4096 Mar 23 02:00 .
drwxrwxr-x  3 wajih wajih 4096 Mar 23 02:00 ..
-rw-rw-r--  1 wajih wajih   23 Mar 23 02:00 HEAD
drwxrwxr-x  2 wajih wajih 4096 Mar 23 02:00 branches
-rw-rw-r--  1 wajih wajih   92 Mar 23 02:00 config
-rw-rw-r--  1 wajih wajih   73 Mar 23 02:00 description
drwxrwxr-x  2 wajih wajih 4096 Mar 23 02:00 hooks
drwxrwxr-x  2 wajih wajih 4096 Mar 23 02:00 info
drwxrwxr-x  4 wajih wajih 4096 Mar 23 02:00 objects
drwxrwxr-x  4 wajih wajih 4096 Mar 23 02:00 refs
```

| Folder/File | What it contains |
|-------------|-----------------|
| `HEAD` | Points to the current branch |
| `config` | Repo-level Git config |
| `objects/` | All your commits, trees, blobs stored as hashes |
| `refs/` | Branch and tag references |
| `hooks/` | Scripts that run on Git events (pre-commit, post-push etc.) |

---

## Task 3: Git Commands Reference — `git-commands.md`

```markdown
# Git Commands Reference

## Setup & Config
| Command | Description | Example |
|---------|-------------|---------|
| `git config --global user.name "Name"` | Set your Git username | `git config --global user.name "Wajih"` |
| `git config --global user.email "email"` | Set your Git email | `git config --global user.email "w@mail.com"` |
| `git config --list` | View all config values | `git config --list` |
| `git --version` | Check installed Git version | `git --version` |

## Basic Workflow
| Command | Description | Example |
|---------|-------------|---------|
| `git init` | Initialize a new repo | `git init` |
| `git add <file>` | Stage a file | `git add README.md` |
| `git add .` | Stage all changes | `git add .` |
| `git commit -m "msg"` | Commit staged changes | `git commit -m "initial commit"` |
| `git status` | Show working tree status | `git status` |

## Viewing Changes
| Command | Description | Example |
|---------|-------------|---------|
| `git log` | Full commit history | `git log` |
| `git log --oneline` | Compact history | `git log --oneline` |
| `git diff` | Show unstaged changes | `git diff` |
| `git diff --staged` | Show staged changes | `git diff --staged` |
```

---

## Task 4: Stage and Commit

```bash
git add git-commands.md
git status
git commit -m "Add initial git commands reference"
git log
```

**Output of `git log`:**
```
commit a1b2c3d4e5f6... (HEAD -> main)
Author: Wajih Ahmed <your@email.com>
Date:   Mon Mar 23 02:10:00 2026 +0500

    Add initial git commands reference
```

---

## Task 5: Multiple Commits — History

```bash
# Edit file, add more commands, then:
git add git-commands.md
git commit -m "Add branching and remote commands"

git add git-commands.md
git commit -m "Add diff and log viewing commands"

git add git-commands.md
git commit -m "Add stash and reset commands"

git log --oneline
```

**Output:**
```
f9e3d21 (HEAD -> main) Add stash and reset commands
c4a7b18 Add diff and log viewing commands
88f1e09 Add branching and remote commands
a1b2c3d Add initial git commands reference
```

---

## Task 6: Understanding Git — `day-22-notes.md`

### 1. What is the difference between `git add` and `git commit`?

`git add` moves changes from your **working directory** into the **staging area** — it's like saying "I want to include this in the next snapshot." `git commit` takes everything in the staging area and permanently saves it to the **repository history** with a message and timestamp. You need both — `add` picks what goes in, `commit` actually saves it.

### 2. What does the staging area do? Why doesn't Git just commit directly?

The staging area gives you **fine-grained control** over what goes into each commit. Imagine you changed 5 files but only 2 are related to the same bug fix — you stage just those 2 and commit them cleanly. The other 3 stay in the working directory for a separate commit. Without a staging area, you'd either commit everything at once or nothing, which makes history messy and hard to read.

### 3. What information does `git log` show you?

For every commit: the **commit hash** (unique ID), **author name and email**, **date and time**, and the **commit message**. With `--oneline` you get just the short hash and message — much cleaner for scanning history.

### 4. What is the `.git/` folder and what happens if you delete it?

The `.git/` folder **is the repository**. It contains every commit, every branch reference, your config, and the entire history of the project. If you delete it, Git has no memory of anything — the folder becomes a plain directory with no version control, no history, no branches. The files themselves remain but Git is completely gone.

### 5. What is the difference between working directory, staging area, and repository?

| Area | What it is |
|------|-----------|
| **Working Directory** | Your actual files on disk — where you edit code |
| **Staging Area** | A "pre-commit zone" — changes you've marked to be committed |
| **Repository** | The permanent history stored in `.git/` — all your commits |

The flow is always: **Working Directory → Staging Area → Repository**
(`git add` → `git commit`)
