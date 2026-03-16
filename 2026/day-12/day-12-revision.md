# Day 12 – Breather & Revision (Days 01–11)

## Goal
Today was a revision day to consolidate the fundamentals learned during Days 01–11 of the DevOps journey.

The focus was on reviewing commands, re-running small tasks, and identifying areas that need improvement.

---

# Review Sections

## 1️⃣ Mindset & Plan (Day 01)

Revisited my learning plan and confirmed my main goal:

- Become comfortable with **Linux fundamentals**
- Build strong **troubleshooting habits**
- Practice commands daily

Small adjustment:
- Spend more time practicing **file permissions and ownership**.

---

## 2️⃣ Processes & Services (Day 04–05)

Commands revisited:

```bash
ps aux
systemctl status ssh
journalctl -u ssh
```

Observations:

- `ps aux` shows all running processes in the system.
- `systemctl status` helps check if a service is **active, inactive, or failed**.
- `journalctl -u` displays logs for a specific service which is useful for troubleshooting.

---

## 3️⃣ File Skills Practice (Days 06–11)

Practiced the following commands:

```bash
echo "DevOps practice" >> notes.txt
chmod 644 notes.txt
ls -l notes.txt
cp notes.txt backup-notes.txt
mkdir practice-folder
```

What I noticed:

- `echo >>` quickly appends text to files.
- `chmod` controls file permissions.
- `ls -l` shows ownership and permissions clearly.

---

## 4️⃣ Cheat Sheet Refresh (Day 03)

Top 5 commands I would use first in an incident:

```
ls
cd
pwd
ps aux
systemctl status
```

Reason:

These commands quickly help understand:
- where I am
- what files exist
- what processes/services are running

---

## 5️⃣ User & Group Practice (Day 09 / Day 11)

Recreated a small scenario:

```bash
sudo useradd testuser
sudo groupadd devteam
sudo chown testuser notes.txt
ls -l notes.txt
id testuser
```

Verification commands:

```
ls -l
id username
```

This confirmed the **owner and group changes correctly**.

---

# Mini Self-Check

### 1️⃣ Which 3 commands save you the most time right now?

**ls**
- Quickly shows files and directories.

**ps aux**
- Shows all running processes for troubleshooting.

**systemctl status**
- Instantly checks if a service is running or failing.

---

### 2️⃣ How do you check if a service is healthy?

Commands I would run:

```bash
systemctl status nginx
ps aux | grep nginx
journalctl -u nginx
```

These show:

- service state
- running process
- service logs

---

### 3️⃣ How do you safely change ownership and permissions?

Example command:

```bash
sudo chown user:group filename
chmod 644 filename
```

This ensures the correct user owns the file while keeping safe permissions.

---

### 4️⃣ What will you focus on improving in the next 3 days?

- Master **chmod numeric permissions**
- Practice **process monitoring commands**
- Improve **Linux troubleshooting workflow**

---

# Key Takeaways

- Linux troubleshooting always starts with **basic commands**.
- Service logs (`journalctl`) are extremely important.
- File ownership and permissions are critical for **system security and DevOps tasks**.

---

# Commands Revisited Today

```
ps aux
systemctl status
journalctl
echo >>
chmod
chown
ls -l
cp
mkdir
```

---

# Progress Reflection

Today helped reinforce the fundamentals instead of learning new concepts. Revisiting commands and small exercises helped build stronger command memory.

---

# Next Step

Continue the DevOps journey with **Day 13**, focusing on deeper Linux skills and system troubleshooting.

---

# DevOps Journey

Day 12 complete of the **#90DaysOfDevOps challenge** 🚀
