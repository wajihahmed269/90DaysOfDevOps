# Day 24 – Advanced Git: Merge, Rebase, Stash & Cherry Pick

## Task 1: Git Merge — Hands-On

```bash
# Create feature-login branch and add commits
git switch -c feature-login
echo "login form" >> login.html
git add login.html && git commit -m "Add login form"
echo "login validation" >> login.html
git add login.html && git commit -m "Add login validation"

# Switch to main and merge
git switch main
git merge feature-login
```

**Fast-forward merge output:**
```
Updating a1b2c3d..f9e3d21
Fast-forward
 login.html | 2 ++
 1 file changed, 2 insertions(+)
```

```bash
# Now create feature-signup BUT also add a commit to main first
git switch -c feature-signup
echo "signup form" >> signup.html
git add signup.html && git commit -m "Add signup form"

git switch main
echo "hotfix" >> hotfix.txt
git add hotfix.txt && git commit -m "Add hotfix to main"

# Now merge — main has moved ahead, so Git creates a merge commit
git merge feature-signup
```

**Merge commit output:**
```
Merge made by the 'ort' strategy.
 signup.html | 1 +
 1 file changed, 1 insertion(+)
```

### Answers

**Fast-forward merge:** Happens when the branch you're merging has all the commits from the target branch plus additional ones — Git simply moves the pointer forward. No merge commit is created. History stays linear.

**Merge commit:** Git creates when both branches have diverged — each has commits the other doesn't. Git creates a new commit with two parents to tie the histories together.

**Merge conflict:** When two branches edit the **same line** of the same file, Git can't decide which version to keep. It marks the file with conflict markers and asks you to resolve manually:
```
<<<<<<< HEAD
your version
=======
their version
>>>>>>> feature-signup
```
Fix the file, then `git add` + `git commit` to complete the merge.

---

## Task 2: Git Rebase — Hands-On

```bash
# Create feature-dashboard and add commits
git switch -c feature-dashboard
echo "dashboard v1" >> dashboard.html
git add dashboard.html && git commit -m "Add dashboard v1"
echo "dashboard v2" >> dashboard.html
git add dashboard.html && git commit -m "Add dashboard v2"

# Add a new commit to main (main moves ahead)
git switch main
echo "new main work" >> main.txt
git add main.txt && git commit -m "New work on main"

# Rebase feature-dashboard onto main
git switch feature-dashboard
git rebase main
```

**Rebase output:**
```
Successfully rebased and updated refs/heads/feature-dashboard.
```

```bash
git log --oneline --graph --all
```

```
* c8d9e10 (HEAD -> feature-dashboard) Add dashboard v2
* b7e6f09 Add dashboard v1
* a1f2c3d (main) New work on main
* 9e8d7c6 Add hotfix to main
```

### Answers

**What rebase does:** Takes your commits from the branch and **replays them one by one on top of the target branch**. Each commit gets a new hash because its parent changed. The commits are the same in content but different objects in Git.

**History difference from merge:**
- Merge: history shows a fork and a merge point — honest but visually complex
- Rebase: history is **perfectly linear** — looks like everything was developed in sequence

**Never rebase shared/pushed commits:** Rebase rewrites commit hashes. If someone else has already pulled your original commits and you rebase them, their history and yours will diverge — causing massive conflicts when they try to pull.

**Rebase vs Merge:**
| Situation | Use |
|-----------|-----|
| Cleaning up local branch before merging | Rebase |
| Integrating finished feature into main | Merge |
| Team has already pulled your branch | Merge (never rebase) |
| Want linear, readable history | Rebase |

---

## Task 3: Squash Commit vs Merge Commit

```bash
# feature-profile: 5 small commits
git switch -c feature-profile
for i in 1 2 3 4 5; do
    echo "change $i" >> profile.html
    git add profile.html
    git commit -m "small change $i"
done

# Squash merge into main
git switch main
git merge --squash feature-profile
git commit -m "Add profile feature (squashed)"

git log --oneline
```

**After squash:**
```
d4e5f67 (HEAD -> main) Add profile feature (squashed)
a1b2c3d New work on main
```
→ All 5 commits collapsed into **1 commit** on main.

```bash
# feature-settings: regular merge
git switch -c feature-settings
echo "settings" >> settings.html
git add settings.html && git commit -m "Add settings page"
echo "settings v2" >> settings.html
git add settings.html && git commit -m "Update settings page"

git switch main
git merge feature-settings
git log --oneline
```

**After regular merge:**
```
f9a8b7c (HEAD -> main) Merge branch 'feature-settings'
c3d4e5f Add settings page
b2c3d4e Update settings page
d4e5f67 Add profile feature (squashed)
```

### Answers

**Squash merge:** Combines all commits from a branch into a single new commit on the target branch. The branch's individual commit history is discarded.

**When to squash:** When your feature branch has messy "WIP", "fix typo", "oops" commits that clutter history. Keeps `main` log clean.

**Trade-off:** You lose granular commit history for that feature. If you need to bisect a bug introduced mid-feature later, it's harder.

---

## Task 4: Git Stash — Hands-On

```bash
# Start working on something, don't commit
echo "work in progress..." >> wip.txt
git add wip.txt

# Need to switch branches urgently — stash first
git stash push -m "WIP: working on wip feature"

# Now you can switch freely
git switch main
# ... do some work ...
git switch feature-1

# Bring back your stashed work
git stash pop
```

**Stash list:**
```bash
git stash list
# stash@{0}: On feature-1: WIP: working on wip feature
# stash@{1}: On main: hotfix WIP
```

**Apply a specific stash without removing it:**
```bash
git stash apply stash@{1}
```

### `git stash pop` vs `git stash apply`

| Command | Behavior |
|---------|---------|
| `git stash pop` | Applies stash and **removes** it from the stash list |
| `git stash apply` | Applies stash but **keeps** it in the stash list |

> Use `apply` when you want to apply the same stash to multiple branches.

**Real-world use:** You're mid-feature and a colleague reports a critical production bug. Stash your WIP, fix the bug on `main`, push — then pop your stash and continue where you left off.

---

## Task 5: Cherry Picking

```bash
# Create feature-hotfix with 3 commits
git switch -c feature-hotfix
echo "fix 1" >> fix1.txt && git add fix1.txt && git commit -m "Fix issue #101"
echo "fix 2" >> fix2.txt && git add fix2.txt && git commit -m "Fix critical login bug"
echo "fix 3" >> fix3.txt && git add fix3.txt && git commit -m "Fix typo in readme"

# Find the commit hashes
git log --oneline
# abc1234 Fix typo in readme
# def5678 Fix critical login bug    ← want only this one
# ghi9012 Fix issue #101

# Cherry-pick only the second commit
git switch main
git cherry-pick def5678
```

**Output:**
```
[main 9f8e7d6] Fix critical login bug
 Date: Mon Mar 23 03:00:00 2026 +0500
 1 file changed, 1 insertion(+)
```

```bash
git log --oneline
# 9f8e7d6 (HEAD -> main) Fix critical login bug
# d4e5f67 Add profile feature (squashed)
```
Only the cherry-picked commit landed on `main`.

### Answers

**What cherry-pick does:** Takes a **specific commit** from any branch and applies it to your current branch. The commit gets a new hash (new context, new parent) but same changes.

**When to use:** You fixed a bug on a feature branch and need that exact fix on `main` immediately — without merging the entire unfinished feature.

**What can go wrong:**
- If the cherry-picked commit depends on code that doesn't exist on the target branch, you'll get a conflict
- Cherry-picking the same commit multiple times creates duplicate commits with different hashes
- Hard to track in history — the same change exists in two places with no relationship visible in `git log`
