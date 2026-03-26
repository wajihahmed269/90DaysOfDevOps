# Day 26 – GitHub CLI: Manage GitHub from Your Terminal

## Task 1: Install and Authenticate

```bash
# Install GitHub CLI (Ubuntu/Debian)
sudo apt update
sudo apt install gh -y

# Authenticate
gh auth login
```

**Auth flow output:**
```
? What account do you want to log into? GitHub.com
? What is your preferred protocol for Git operations? HTTPS
? Authenticate Git with your GitHub credentials? Yes
? How would you like to authenticate GitHub CLI? Login with a web browser

! First copy your one-time code: XXXX-XXXX
Press Enter to open github.com in your browser...
✓ Authentication complete.
- gh config set -h github.com git_protocol https
✓ Configured git protocol
✓ Logged in as wajihahmed269
```

```bash
# Verify login
gh auth status
```

```
github.com
  ✓ Logged in to github.com as wajihahmed269
  ✓ Git operations for github.com configured to use https protocol
  ✓ Token: gho_************************************
```

### Authentication Methods `gh` Supports

| Method | How |
|--------|-----|
| Web browser | One-time code, opens browser — easiest |
| Personal Access Token (PAT) | Paste token manually — good for servers/CI |
| SSH key | For SSH protocol git operations |
| OAuth app | For automated flows via `gh auth token` |

---

## Task 2: Working with Repositories

```bash
# Create a new public repo with README from terminal
gh repo create devops-test-repo --public --add-readme --description "Test repo via GitHub CLI"
```

```
✓ Created repository wajihahmed269/devops-test-repo on GitHub
  https://github.com/wajihahmed269/devops-test-repo
```

```bash
# Clone using gh
gh repo clone wajihahmed269/devops-test-repo

# View repo details
gh repo view wajihahmed269/devops-test-repo

# List all your repos
gh repo list

# Open repo in browser
gh repo view --web

# Delete test repo
gh repo delete wajihahmed269/devops-test-repo --confirm
```

**Output of `gh repo list`:**
```
wajihahmed269/90DaysOfDevOps          public   about 2 hours ago
wajihahmed269/devops-git-practice     public   about 1 day ago
wajihahmed269/shell-scripts           public   about 2 days ago
wajihahmed269/python-scripts          public   about 3 days ago
wajihahmed269/devops-notes            public   about 4 days ago
```

---

## Task 3: Issues

```bash
# Create an issue with title, body, and label
gh issue create \
  --title "Add disk usage alert to system_info.sh" \
  --body "The script should alert when disk usage goes above 80%" \
  --label "enhancement" \
  --repo wajihahmed269/shell-scripts
```

```
https://github.com/wajihahmed269/shell-scripts/issues/1
```

```bash
# List all open issues
gh issue list --repo wajihahmed269/shell-scripts

# View a specific issue by number
gh issue view 1 --repo wajihahmed269/shell-scripts

# Close an issue
gh issue close 1 --repo wajihahmed269/shell-scripts
```

**Output of `gh issue list`:**
```
#1  Add disk usage alert to system_info.sh  enhancement  about 1 minute ago
```

### How `gh issue` Can Be Used in Automation

```bash
# Auto-create an issue when a script fails in CI
if ! bash system_info.sh; then
  gh issue create \
    --title "system_info.sh failed on $(hostname)" \
    --body "Script failed at $(date). Check the logs." \
    --label "bug"
fi

# List issues and pipe to scripts for triage automation
gh issue list --json number,title,labels | jq '.[] | select(.labels[].name == "bug")'
```

---

## Task 4: Pull Requests

```bash
# Create a branch, make a change, push it
git switch -c feature/add-alert
echo "# Alert when disk > 80%" >> system_info.sh
git add system_info.sh
git commit -m "Add disk usage alert threshold"
git push origin feature/add-alert

# Create PR from terminal
gh pr create \
  --title "Add disk usage alert" \
  --body "Alerts when disk usage exceeds 80% — closes #1" \
  --base main \
  --head feature/add-alert
```

```
https://github.com/wajihahmed269/shell-scripts/pull/2
```

```bash
# List all open PRs
gh pr list

# View PR details
gh pr view 2

# Merge the PR
gh pr merge 2 --squash --delete-branch
```

**Output of `gh pr view 2`:**
```
title:  Add disk usage alert
state:  OPEN
author: wajihahmed269
branch: feature/add-alert -> main
checks: ✓ All checks passed
```

### `gh pr merge` Merge Methods

| Method | Flag | What it does |
|--------|------|-------------|
| Merge commit | `--merge` | Creates a merge commit — preserves full history |
| Squash | `--squash` | Squashes all PR commits into one |
| Rebase | `--rebase` | Replays commits linearly onto base branch |

### Reviewing Someone Else's PR

```bash
# Check out their PR locally to test it
gh pr checkout 5

# Add a review comment
gh pr review 5 --comment --body "Looks good, but add error handling for empty dir"

# Approve
gh pr review 5 --approve

# Request changes
gh pr review 5 --request-changes --body "Please add unit tests"
```

---

## Task 5: GitHub Actions & Workflows (Preview)

```bash
# List workflow runs on a public repo
gh run list --repo kubernetes/kubernetes

# View status of a specific run
gh run view <run-id> --repo kubernetes/kubernetes

# Watch a run in real time
gh run watch <run-id>
```

### How `gh run` and `gh workflow` Are Useful in CI/CD

```bash
# Trigger a workflow manually from terminal
gh workflow run deploy.yml --ref main

# Check if the latest deploy succeeded before proceeding
STATUS=$(gh run list --workflow=deploy.yml --limit 1 --json status -q '.[0].status')
if [ "$STATUS" != "completed" ]; then
  echo "Deploy still running, aborting release script"
  exit 1
fi

# In a pipeline: wait for a workflow to finish
gh run watch $(gh run list --limit 1 --json databaseId -q '.[0].databaseId')
```

This becomes critical when you start building CI/CD pipelines — you can gate deploys on workflow success without ever opening a browser.

---

## Task 6: Useful `gh` Tricks

```bash
# Make a raw GitHub API call
gh api /user
gh api /repos/wajihahmed269/shell-scripts/issues

# Create a Gist
gh gist create system_info.sh --public --desc "System info reporter script"

# List your Gists
gh gist list

# Create a release
gh release create v1.0.0 --title "Week 4 Scripts" --notes "Log rotation, backup, and system info scripts"

# Create a command alias
gh alias set prl 'pr list'
gh prl   # now works as shortcut

# Search GitHub repos
gh search repos "devops shell scripts" --limit 10

# Get JSON output for scripting
gh repo list --json name,description,isPrivate | jq '.[] | select(.isPrivate == false)'
```

---

## Updated `git-commands.md` — GitHub CLI Section

```bash
# Auth
gh auth login                              # Authenticate with GitHub
gh auth status                             # Check login status

# Repos
gh repo create <name> --public             # Create repo from terminal
gh repo clone <owner>/<repo>               # Clone via gh
gh repo view                               # View current repo info
gh repo list                               # List all your repos
gh repo view --web                         # Open repo in browser
gh repo delete <repo> --confirm            # Delete a repo

# Issues
gh issue create --title "" --body ""       # Create issue
gh issue list                              # List open issues
gh issue view <number>                     # View specific issue
gh issue close <number>                    # Close issue

# Pull Requests
gh pr create --title "" --base main        # Create a PR
gh pr list                                 # List open PRs
gh pr view <number>                        # View PR details
gh pr merge <number> --squash              # Merge PR (squash)
gh pr checkout <number>                    # Check out PR locally
gh pr review <number> --approve            # Approve PR

# Workflows (Actions)
gh run list                                # List workflow runs
gh run view <id>                           # View run details
gh run watch <id>                          # Watch run live
gh workflow run <file> --ref main          # Trigger workflow

# Extras
gh api /user                               # Raw API call
gh gist create <file>                      # Create a Gist
gh release create <tag>                    # Create a release
gh alias set <name> '<command>'            # Create alias
gh search repos "<query>"                  # Search GitHub repos
```
