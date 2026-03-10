# Day 08 – Cloud Server Setup: Nginx & Web Deployment
*

## Part 1: Launch Cloud Instance & SSH Access

### Instance Details
- **Cloud Provider:** AWS EC2
- **Instance Type:** Free Tier
- **OS:** Ubuntu 24.04 LTS
- **Security Group:** launch-wizard-5

### SSH Connection
```bash
ssh -i your-key.pem ubuntu@51.21.167.65
```

**Observed:** Successfully connected to EC2 instance via SSH using .pem key file.

---

## Part 2: System Update & Nginx Installation

### Step 1 — Update System
```bash
sudo apt update
```


**Observed:** Package lists updated successfully. 103 packages available for upgrade. Server connected to EU North 1 AWS mirror for fast downloads.

---

### Step 2 — Install Nginx
```bash
sudo apt install nginx -y
```


**Observed:** Nginx 1.24.0 installed successfully. Only 1.6MB disk space used. Symlink created automatically — Nginx will start on every reboot.

---

### Step 3 — Verify Nginx is Running
```bash
sudo systemctl status nginx
```


**Observed:** Nginx is active and running. Only 2.4MB RAM used. Master process managing 2 worker processes. Enabled — will auto-start on reboot.

---

## Part 3: Security Group Configuration

### Inbound Rules Configured

| Type | Protocol | Port | Source |
|------|----------|------|--------|
| SSH | TCP | 22 | 0.0.0.0/0 |
| HTTP | TCP | 80 | 0.0.0.0/0 |

**Observed:** Initially only Port 22 was open. Added Port 80 (HTTP) to allow web traffic. Verified Nginx welcome page accessible from browser at http://51.21.167.65

**Challenge faced:** Accidentally deleted Port 22 while editing rules — fixed by adding SSH rule back immediately.

---

## Part 4: Nginx Log Extraction

### View Access Logs
```bash
sudo cat /var/log/nginx/access.log
```


**Observed:** 3 entries — all from my own IP (202.47.51.242). First visit returned 200 (success). favicon.ico returned 404 (normal — no icon configured). Second visit returned 304 (browser used cache).

---

### View Error Logs
```bash
sudo cat /var/log/nginx/error.log
```

**Output:**
```
2026/03/09 23:41:45 [notice] 1576#1576: using inherited sockets from "5;6;"
```

**Observed:** Only one notice — no real errors. Nginx is running cleanly.

---

### Save Logs to File
```bash
sudo cat /var/log/nginx/access.log > ~/nginx-logs.txt
cat ~/nginx-logs.txt
```

**Observed:** Logs saved successfully to nginx-logs.txt in home directory.

---

## Commands Used

| Command | Purpose |
|---------|---------|
| `sudo apt update` | Refresh package lists |
| `sudo apt install nginx -y` | Install Nginx web server |
| `sudo systemctl status nginx` | Verify Nginx is running |
| `sudo cat /var/log/nginx/access.log` | View web access logs |
| `sudo cat /var/log/nginx/error.log` | View error logs |
| `sudo cat /var/log/nginx/access.log > ~/nginx-logs.txt` | Save logs to file |

---

## Challenges Faced

1. **Port 22 accidentally deleted** — While editing security group rules to add Port 80, accidentally deleted the SSH rule (Port 22). Fixed immediately by adding SSH rule back through AWS Console.


---

## What I Learned

- Cloud servers need **Security Groups** configured correctly — without Port 80 open, nobody can visit your website even if Nginx is running
- Nginx uses a **master + worker process** model — master manages, workers handle requests
- **Access logs** record every visit with IP, timestamp, status code and browser info
- **304 status** means browser used cached version — faster for the user
- Always open **both Port 22 and Port 80** when launching a web server instance

---


---
*Completed as part of DevOps Bootcamp — Day 08*
