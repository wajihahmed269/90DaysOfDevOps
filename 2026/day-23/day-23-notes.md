# Day 23 – Git Branching & Working with GitHub

## Task 1: Understanding Branches — `day-23-notes.md`

### 1. What is a branch in Git?

A branch is a **lightweight movable pointer** to a specific commit. When you create a branch, Git just creates a new pointer — it doesn't copy any files. The default branch is called `main` (or `master` in older repos). Every new commit on a branch moves that pointer forward automatically.

### 2. Why do we use branches instead of committing everything to `main`?

Because `main` is your **stable, production-ready code**. If you're building a new feature or fixing a bug, you work on a separate branch so you don't break what's already working. If the feature turns out bad, you just delete the branch — `main` is untouched. It also lets multiple people work simultaneously without stepping on each other.

### 3. What is `HEAD` in Git?

`HEAD` is a special pointer that always points to **the commit you're currently on** — which is usually the tip of your current branch. When you switch branches, `HEAD` moves to point at that branch. Think of it as "you are here" on the Git map.

### 4. What happens to your files when you switch branches?

Git **physically changes your working directory** to match the state of the branch you switch to. Files that exist on the new branch appear, files that don't exist there disappear, and files that differ are updated. This is why you should always commit or stash before switching.

---

## Task 2: Branching Commands — Hands-On

```bash
# List all branches
git branch

# Create a new branch
git branch feature-1

# Switch to a branch
git checkout feature-1
# OR (modern way)
git switch feature-1

# Create and switch in one command
git switch -c feature-2
# OR
git checkout -b feature-2

# Make a commit on feature-1
git switch feature-1
echo "feature 1 work" >> feature1.txt
git add feature1.txt
git commit -m "Add feature 1 work"

# Switch back to main — feature-1 commit is not visible here
git switch main
git log --oneline   # feature-1 commit won't appear

# Delete a branch
git branch -d feature-2       # safe delete (only if merged)
git branch -D feature-2       # force delete
```

### `git switch` vs `git checkout`

| | `git checkout` | `git switch` |
|--|---------------|-------------|
| Purpose | Branch switching + file restoration | Branch switching only |
| Introduced | Old, original command | Git 2.23+ (modern) |
| Risk | Can accidentally overwrite files | Safer, more explicit |
| Recommendation | Still works, widely used | Preferred for clarity |

---

## Task 3: Push to GitHub

```bash
# Connect local repo to GitHub remote
git remote add origin https://github.com/wajihahmed269/devops-git-practice.git

# Verify remote was added
git remote -v

# Push main branch
git push -u origin main

# Push feature-1 branch
git push -u origin feature-1
```

**Output of `git remote -v`:**
```
origin  https://github.com/wajihahmed269/devops-git-practice.git (fetch)
origin  https://github.com/wajihahmed269/devops-git-practice.git (push)
```

### `origin` vs `upstream`

| Term | What it means |
|------|--------------|
| `origin` | Your own remote — the repo you cloned from or connected to (usually your GitHub) |
| `upstream` | The original repo you forked from — someone else's repo |

In a fork workflow: `upstream` = original project, `origin` = your personal fork.

---

## Task 4: Pull from GitHub

```bash
# Made a change directly on GitHub, now pull it locally
git pull origin main
```

**Output:**
```
remote: Enumerating objects: 5, done.
remote: Counting objects: 100% (5/5), done.
Unpacking objects: 100% (3/3), done.
From https://github.com/wajihahmed269/devops-git-practice
   f9e3d21..a9c4b11  main -> origin/main
Updating f9e3d21..a9c4b11
Fast-forward
 README.md | 3 +++
 1 file changed, 3 insertions(+)
```

### `git fetch` vs `git pull`

| Command | What it does |
|---------|-------------|
| `git fetch` | Downloads changes from remote but does **not** merge them into your branch — you inspect first |
| `git pull` | Downloads **and** immediately merges into your current branch (fetch + merge in one step) |

> Prefer `git fetch` + review + `git merge` in team environments. `git pull` is fine for personal projects.

---

## Task 5: Clone vs Fork

```bash
# Clone a public repo
git clone https://github.com/someuser/some-repo.git

# Fork on GitHub first (via UI), then clone your fork
git clone https://github.com/wajihahmed269/some-repo.git
cd some-repo

# Add upstream to track original
git remote add upstream https://github.com/someuser/some-repo.git
git remote -v
```

### Clone vs Fork

| | Clone | Fork |
|--|-------|------|
| What it is | Copy repo to your local machine | Copy repo to **your GitHub account** |
| Where it lives | Your local disk | GitHub (remote) |
| Can you push? | Only if you have write access | Yes — it's your copy |
| Use case | Work on repos you have access to | Contribute to others' projects |

### Keeping your fork in sync with the original

```bash
git fetch upstream             # get latest from original repo
git switch main                # make sure you're on main
git merge upstream/main        # merge original's changes into your main
git push origin main           # push updated main to your fork
```
