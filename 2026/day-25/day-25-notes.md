# Day 25 – Git Reset vs Revert & Branching Strategies

## Task 1: Git Reset — Hands-On

### Setup: 3 commits to work with

```bash
echo "commit A" >> history.txt && git add . && git commit -m "Commit A"
echo "commit B" >> history.txt && git add . && git commit -m "Commit B"
echo "commit C" >> history.txt && git add . && git commit -m "Commit C"

git log --oneline
# f3c1a2b (HEAD -> main) Commit C
# e9d4b1a Commit B
# a7c3f2d Commit A
```

---

### `git reset --soft`

```bash
git reset --soft HEAD~1
git status
```

```
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   history.txt
```

**What happened:** Commit C is gone from history, but its changes are still **staged**. Ready to commit again immediately.

---

### `git reset --mixed` (default)

```bash
git commit -m "Commit C again"   # re-commit first
git reset --mixed HEAD~1
git status
```

```
On branch main
Changes not staged for commit:
        modified:   history.txt
```

**What happened:** Commit C is gone from history, changes are back in the **working directory** — unstaged. You need to `git add` again before committing.

---

### `git reset --hard`

```bash
git add . && git commit -m "Commit C final"
git reset --hard HEAD~1
git status
```

```
On branch main
nothing to commit, working tree clean
```

**What happened:** Commit C is completely gone. The changes are **deleted** — no staging area, no working directory. This is the destructive one.

---

### Answers

**Difference between `--soft`, `--mixed`, and `--hard`:**

| Flag | Commit removed? | Staging area | Working directory |
|------|----------------|-------------|-------------------|
| `--soft` | ✅ Yes | Changes kept staged | Untouched |
| `--mixed` | ✅ Yes | Changes unstaged | Untouched |
| `--hard` | ✅ Yes | Cleared | Changes deleted |

**Which is destructive?** `--hard` — it permanently deletes uncommitted changes with no recovery path (unless you use `git reflog` immediately).

**When to use each:**
- `--soft` → You want to redo the commit message or squash with the previous commit
- `--mixed` → You want to unstage changes and re-organize what goes into the commit
- `--hard` → You want to completely throw away work and go back to a clean state

**Should you reset pushed commits?**
No. Never use `git reset` on commits that are already pushed to a shared branch. It rewrites history, and anyone who pulled those commits will have a diverged repo. Use `git revert` instead.

---

## Task 2: Git Revert — Hands-On

```bash
echo "X" >> file.txt && git add . && git commit -m "Commit X"
echo "Y" >> file.txt && git add . && git commit -m "Commit Y"
echo "Z" >> file.txt && git add . && git commit -m "Commit Z"

git log --oneline
# c9f3b1a (HEAD -> main) Commit Z
# b7e2a1d Commit Y
# a4c1f2e Commit X
```

```bash
# Revert the middle commit (Y)
git revert b7e2a1d
```

```
[main d2e4f1c] Revert "Commit Y"
 1 file changed, 1 deletion(-)
```

```bash
git log --oneline
# d2e4f1c (HEAD -> main) Revert "Commit Y"
# c9f3b1a Commit Z
# b7e2a1d Commit Y     ← still here
# a4c1f2e Commit X
```

**Commit Y is still in the history.** Revert adds a *new* commit that undoes the changes — it doesn't erase anything.

---

### Answers

**How is `git revert` different from `git reset`?**
Reset moves the branch pointer backward and removes commits from history. Revert creates a brand new commit that does the opposite of the target commit — history is preserved.

**Why is revert safer for shared branches?**
Because it doesn't rewrite history. Everyone who already pulled the branch can still pull the revert commit cleanly — no divergence, no force-push required.

**When to use revert vs reset:**
- `revert` → Undoing something on a shared/public branch (main, develop)
- `reset` → Cleaning up local commits before pushing

---

## Task 3: Reset vs Revert — Comparison

| | `git reset` | `git revert` |
|--|------------|-------------|
| **What it does** | Moves branch pointer back, removes commits | Creates a new commit that undoes changes |
| **Removes commit from history?** | ✅ Yes | ❌ No — adds a new commit |
| **Safe for shared/pushed branches?** | ❌ No — rewrites history | ✅ Yes — non-destructive |
| **When to use** | Local cleanup before pushing | Undoing changes already on a shared branch |

---

## Task 4: Branching Strategies

### 1. GitFlow

**How it works:**
A structured model with dedicated branches for every stage of development.

```
main        ──────────────────────────────────────────► (production)
                  ↑ merge           ↑ merge
hotfix        ────┘                 │
                                    │
release     ────────────────────────┘
                  ↑ merge
develop     ──────────────────────────────────────────►
                  ↑ merge  ↑ merge  ↑ merge
feature/A   ──────┘
feature/B            ──────┘
feature/C                    ──────┘
```

- `main` — production-ready code only
- `develop` — integration branch for features
- `feature/*` — one branch per feature, merged into develop
- `release/*` — created from develop when ready to ship, merged to main + develop
- `hotfix/*` — created from main to fix urgent bugs, merged to main + develop

**Used by:** Teams with scheduled releases, enterprise software, versioned products.

**Pros:**
- Very structured — clear rules for every situation
- Supports parallel development well
- Good for versioned releases

**Cons:**
- Heavy — lots of branches to manage
- Slow — features sit in develop waiting for a release window
- Overkill for small teams or fast-moving projects

---

### 2. GitHub Flow

**How it works:**
Simple model — one stable `main` branch, short-lived feature branches, deploy directly from main.

```
main     ──────────────────────────────────────────► (always deployable)
              ↑ PR + merge   ↑ PR + merge
feature/A ────┘
feature/B              ──────┘
```

1. Branch off `main`
2. Make changes, open a Pull Request
3. Review, discuss, iterate
4. Merge to main → deploy

**Used by:** GitHub itself, most open-source projects, small-to-medium teams shipping continuously.

**Pros:**
- Simple to understand
- Fast — no release branches, no develop branch
- Continuous delivery friendly

**Cons:**
- Requires good CI/CD to make main always stable
- Harder to manage multiple concurrent versions

---

### 3. Trunk-Based Development

**How it works:**
Everyone commits directly to `main` (trunk). Feature branches exist but are very short-lived (hours, not days). Feature flags hide incomplete features in production.

```
main (trunk) ─────●─────●─────●─────●─────►  (continuous deployment)
                  ↑     ↑
short branch ─────┘     │  (merged same day or next day)
short branch ───────────┘
```

**Used by:** Google, Facebook, high-velocity startups — teams doing multiple deploys per day.

**Pros:**
- Eliminates long-lived branches and merge conflicts
- Forces small, frequent commits
- Fastest possible integration

**Cons:**
- Requires strong CI/CD and automated testing
- Feature flags add complexity
- Discipline-heavy — bad commits hit everyone immediately

---

### Strategy Recommendations

**Startup shipping fast?** → **GitHub Flow**. It's lean, fast, and gets features to users without ceremony. Main is always deployable.

**Large team with scheduled releases?** → **GitFlow**. The structure handles parallel work across multiple releases without chaos.

**Favorite open-source project:** React (facebook/react) uses **GitHub Flow** — feature branches, PRs into main, simple and fast for a globally distributed team.

---

## git-commands.md — Updated Reference

### Setup & Config
```bash
git config --global user.name "Name"     # Set username
git config --global user.email "email"   # Set email
git config --list                         # View all config
git --version                             # Check version
```

### Basic Workflow
```bash
git init                    # Initialize repo
git status                  # Show working tree status
git add <file>              # Stage a file
git add .                   # Stage all changes
git commit -m "msg"         # Commit staged changes
git log                     # Full history
git log --oneline           # Compact history
git diff                    # Unstaged changes
git diff --staged           # Staged changes
```

### Branching
```bash
git branch                  # List branches
git branch <name>           # Create branch
git switch <name>           # Switch to branch
git switch -c <name>        # Create and switch
git branch -d <name>        # Delete branch (safe)
git branch -D <name>        # Force delete branch
```

### Remote
```bash
git remote add origin <url>   # Connect to remote
git remote -v                  # List remotes
git push -u origin <branch>   # Push and set upstream
git pull origin <branch>       # Fetch + merge
git fetch origin               # Fetch only (no merge)
git clone <url>                # Clone a repo
```

### Merging & Rebasing
```bash
git merge <branch>             # Merge branch into current
git merge --squash <branch>    # Squash all commits then merge
git rebase <branch>            # Replay commits onto branch
git rebase --abort             # Abort a rebase in progress
git rebase --continue          # Continue after conflict fix
```

### Stash & Cherry Pick
```bash
git stash push -m "msg"        # Stash with label
git stash list                 # List all stashes
git stash pop                  # Apply and remove stash
git stash apply stash@{n}      # Apply specific stash (keep it)
git cherry-pick <hash>         # Apply specific commit
```

### Reset & Revert
```bash
git reset --soft HEAD~1        # Undo commit, keep staged
git reset --mixed HEAD~1       # Undo commit, unstage changes
git reset --hard HEAD~1        # Undo commit, delete changes
git revert <hash>              # Create undo commit (safe)
git reflog                     # View all Git actions (recovery)
```
