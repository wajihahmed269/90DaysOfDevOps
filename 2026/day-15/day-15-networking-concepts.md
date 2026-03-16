# Day 15 – Networking Concepts: DNS, IP, Subnets & Ports

## Goal

Understand the core networking concepts that DevOps engineers use daily: **DNS resolution, IP addressing, subnetting (CIDR), and network ports**.

---

# Task 1 – DNS: How Names Become IPs

## What happens when you type `google.com` in a browser?

1. The browser asks the **DNS resolver** for the IP address of `google.com`.
2. The resolver queries DNS servers to find the correct record.
3. The DNS server returns the **IP address** for the domain.
4. The browser connects to that IP address using **HTTP or HTTPS**.

---

## DNS Record Types

| Record | Purpose |
|------|------|
| **A** | Maps a domain name to an IPv4 address |
| **AAAA** | Maps a domain name to an IPv6 address |
| **CNAME** | Alias that points one domain name to another |
| **MX** | Specifies mail servers for email delivery |
| **NS** | Defines the authoritative name servers for a domain |

---

## Command Used

```bash
dig google.com
```

Example Output (simplified):

```
google.com.   300  IN  A  142.250.190.14
```

Observation:

- **A Record:** 142.250.190.14  
- **TTL:** 300 seconds

TTL means the response can be cached for **300 seconds**.

---

# Task 2 – IP Addressing

## What is an IPv4 Address?

An **IPv4 address** is a 32-bit number used to identify devices on a network.

Example:

```
192.168.1.10
```

It consists of **four octets** separated by dots, each ranging from **0–255**.

---

## Public vs Private IP

| Type | Description | Example |
|----|----|----|
| Public IP | Accessible from the internet | 8.8.8.8 |
| Private IP | Used inside local networks | 192.168.1.10 |

---

## Private IP Ranges

```
10.0.0.0 – 10.255.255.255
172.16.0.0 – 172.31.255.255
192.168.0.0 – 192.168.255.255
```

---

## Command Used

```bash
ip addr show
```

Example observation:

```
inet 192.168.1.25/24
```

This IP is a **private IP address**.

---

# Task 3 – CIDR & Subnetting

## What does `/24` mean?

`/24` represents the **subnet mask length**.

It means **24 bits are used for the network portion** and the remaining bits are for hosts.

Example:

```
192.168.1.0/24
```

Subnet mask:

```
255.255.255.0
```

---

## CIDR Table

| CIDR | Subnet Mask | Total IPs | Usable Hosts |
|----|----|----|----|
| /24 | 255.255.255.0 | 256 | 254 |
| /16 | 255.255.0.0 | 65,536 | 65,534 |
| /28 | 255.255.255.240 | 16 | 14 |

---

## Why Do We Subnet?

Subnetting allows networks to:

- Divide large networks into **smaller manageable segments**
- Improve **security and performance**
- Efficiently allocate **IP address space**

---

# Task 4 – Ports: The Doors to Services

## What is a Port?

A **port** is a communication endpoint used by applications to send and receive data.

Ports allow multiple services to run on the same machine.

Example:

```
IP address + Port = Network service
```

Example:

```
192.168.1.25:22
```

---

## Common Ports

| Port | Service |
|----|----|
| 22 | SSH |
| 80 | HTTP |
| 443 | HTTPS |
| 53 | DNS |
| 3306 | MySQL |
| 6379 | Redis |
| 27017 | MongoDB |

---

## Command Used

```bash
ss -tulpn
```

Example output:

```
tcp LISTEN 0 128 0.0.0.0:22
tcp LISTEN 0 128 127.0.0.1:631
```

Observation:

- Port **22 → SSH service**
- Port **631 → Printing service**

---

# Task 5 – Putting It Together

## Scenario 1

Command:

```
curl http://myapp.com:8080
```

Networking concepts involved:

- DNS resolves `myapp.com` to an IP address.
- The connection uses **HTTP protocol over TCP**.
- The request is sent to **port 8080** on the server.

---

## Scenario 2

Problem:

```
App cannot reach database at 10.0.1.50:3306
```

Things to check:

1. Confirm database service is running
2. Verify port **3306 is open**
3. Check network connectivity using `ping` or `nc`

---

# Commands Used

```
dig google.com
ip addr show
ss -tulpn
```

---

# What I Learned

1. **DNS translates domain names into IP addresses**, allowing users to access websites easily.
2. **CIDR notation defines network sizes and host capacity** in subnetting.
3. **Ports allow multiple network services to operate on the same machine simultaneously**.

---

# DevOps Progress

Day 15 completed as part of the **#90DaysOfDevOps challenge** 🚀

#90DaysOfDevOps  
#DevOpsKaJosh  
#TrainWithShubham
