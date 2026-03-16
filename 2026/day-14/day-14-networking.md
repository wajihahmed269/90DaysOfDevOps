# Day 14 – Networking Fundamentals & Hands-on Checks

## Goal

Understand basic networking concepts and practice common troubleshooting commands used by DevOps engineers.

---

# Networking Concepts

## OSI Model (L1–L7)

The OSI model explains how network communication works through seven layers.

1. **Physical (L1)** – Cables, switches, hardware transmission.
2. **Data Link (L2)** – MAC addresses and local network communication.
3. **Network (L3)** – IP addressing and routing.
4. **Transport (L4)** – Reliable or unreliable data delivery (TCP/UDP).
5. **Session (L5)** – Maintains communication sessions.
6. **Presentation (L6)** – Data formatting, encryption.
7. **Application (L7)** – Applications like web browsers and APIs.

---

## TCP/IP Model

The TCP/IP model is a simplified networking model with four layers.

| TCP/IP Layer | Example Protocols |
|--------------|------------------|
| Link | Ethernet, Wi-Fi |
| Internet | IP |
| Transport | TCP, UDP |
| Application | HTTP, HTTPS, DNS |

---

## Protocol Placement

| Protocol | Layer |
|--------|-------|
| IP | Internet / Network Layer |
| TCP / UDP | Transport Layer |
| HTTP / HTTPS | Application Layer |
| DNS | Application Layer |

---

## Real Example

When running:

```
curl https://example.com
```

The request flows as:

```
Application (HTTP)
↓
Transport (TCP)
↓
Internet (IP)
↓
Link (Ethernet / WiFi)
```

---

# Hands-on Networking Checks

## 1. Identity – Check IP Address

Command:

```bash
hostname -I
```

or

```bash
ip addr show
```

Observation:

This command displays the system's IP address on the local network.

Example output:

```
192.168.1.25
```

---

## 2. Reachability – Ping Test

Command:

```bash
ping google.com
```

Observation:

- Packets successfully reached the target server.
- Latency was around **20–40 ms**.
- No packet loss observed.

---

## 3. Network Path – Traceroute

Command:

```bash
traceroute google.com
```

or

```bash
tracepath google.com
```

Observation:

Shows each router hop between the system and the destination server.

Some hops may show higher latency depending on network routing.

---

## 4. Check Listening Ports

Command:

```bash
ss -tulpn
```

Observation:

Lists all listening services and their ports.

Example service:

```
tcp LISTEN 0 128 0.0.0.0:22
```

Meaning:

- SSH service is listening on **port 22**.

---

## 5. DNS Resolution

Command:

```bash
dig google.com
```

or

```bash
nslookup google.com
```

Observation:

The domain name resolves to an IP address.

Example:

```
142.250.190.14
```

---

## 6. HTTP Check

Command:

```bash
curl -I https://google.com
```

Observation:

Returns HTTP headers and status code.

Example:

```
HTTP/1.1 200 OK
```

Meaning the website is reachable and responding successfully.

---

## 7. Network Connections Snapshot

Command:

```bash
netstat -an | head
```

Observation:

Displays network connections and states such as:

- LISTEN
- ESTABLISHED
- TIME_WAIT

This helps understand active connections on the system.

---

# Mini Task – Port Probe

Identified listening port:

```
SSH on port 22
```

Test command:

```bash
nc -zv localhost 22
```

Output:

```
Connection to localhost 22 port [tcp/ssh] succeeded
```

Conclusion:

The SSH service is reachable on the local machine.

If the port was unreachable, the next checks would be:

- Verify service status with `systemctl status ssh`
- Check firewall rules using `ufw status`

---

# Reflection

## Which command gives the fastest signal when something is broken?

`ping` is usually the fastest command to check if a host is reachable.

---

## If DNS fails, which layer should be checked?

Application layer (DNS service).

Possible checks:

- `dig domain.com`
- `/etc/resolv.conf`

---

## If HTTP 500 appears, which layer should be checked?

Application layer.

Possible causes:

- Web server issues
- Backend service failure
- Application errors

---

## Two follow-up checks during a real incident

1. Check service status:

```bash
systemctl status nginx
```

2. Check logs:

```bash
journalctl -u nginx
```

---

# Commands Practiced Today

```
hostname -I
ip addr show
ping
traceroute
tracepath
ss -tulpn
dig
nslookup
curl -I
netstat -an
nc -zv
```

---

# What I Learned

1. Networking troubleshooting starts with simple checks like **ping and DNS resolution**.
2. Commands like **ss and netstat** help identify running services and open ports.
3. Tools like **curl** allow quick testing of HTTP services from the command line.

---

# DevOps Journey

Day 14 completed as part of the **#90DaysOfDevOps challenge** 🚀

#90DaysOfDevOps  
#DevOpsKaJosh  
#TrainWithShubham
