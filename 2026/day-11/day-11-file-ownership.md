# Day 11 – File Ownership Challenge

## Understanding File Ownership in Linux

In Linux, every file and directory has two types of ownership:

- **Owner (User)** → The user who owns the file.
- **Group** → A group of users who share permissions for that file.

Command used to check ownership:

```bash
ls -l
```

Example output:

```
-rw-r--r-- 1 wajih wajih 0 Mar 16 devops-file.txt
```

Format:

```
permissions  links  owner  group  size  date  filename
```

### Difference Between Owner and Group

| Type | Description |
|-----|-------------|
| Owner | The main user who controls the file |
| Group | Multiple users that can share access |

---

# Files & Directories Created

```
devops-file.txt
team-notes.txt
project-config.yaml
app-logs/

heist-project/
heist-project/vault/gold.txt
heist-project/plans/strategy.conf

bank-heist/
bank-heist/access-codes.txt
bank-heist/blueprints.pdf
bank-heist/escape-plan.txt
```

---

# Ownership Changes

### devops-file.txt
```
user:user → tokyo:user
tokyo:user → berlin:user
```

### team-notes.txt
```
user:user → user:heist-team
```

### project-config.yaml
```
user:user → professor:heist-team
```

### app-logs/
```
user:user → berlin:heist-team
```

### heist-project directory
```
user:user → professor:planners
```

### bank-heist files

```
access-codes.txt → tokyo:vault-team
blueprints.pdf → berlin:tech-team
escape-plan.txt → nairobi:vault-team
```

---

# Commands Used

```bash
ls -l
touch
mkdir -p

sudo useradd tokyo
sudo useradd berlin
sudo useradd nairobi

sudo groupadd heist-team
sudo groupadd planners
sudo groupadd vault-team
sudo groupadd tech-team

sudo chown tokyo devops-file.txt
sudo chown berlin devops-file.txt

sudo chgrp heist-team team-notes.txt

sudo chown professor:heist-team project-config.yaml
sudo chown berlin:heist-team app-logs

sudo chown -R professor:planners heist-project/

sudo chown tokyo:vault-team bank-heist/access-codes.txt
sudo chown berlin:tech-team bank-heist/blueprints.pdf
sudo chown nairobi:vault-team bank-heist/escape-plan.txt
```

---

# Screenshots

Add screenshots of:

- `ls -l devops-file.txt`
- `ls -l team-notes.txt`
- `ls -l project-config.yaml`
- `ls -lR heist-project/`
- `ls -l bank-heist/`

---

# What I Learned

1. Every Linux file has an **owner and a group**.
2. The `chown` command changes **file ownership**.
3. The `chgrp` command changes **group ownership**.
4. The `-R` flag allows recursive ownership changes for directories.

---
