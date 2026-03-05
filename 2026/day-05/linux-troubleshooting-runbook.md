Linux Troubleshooting Runbook
Kernel: 6.17.0-14-generic
Target Service: cron (PID 1014)

1. Environment Basics
CommandOutput Summaryuname -aKernel 6.17.0-14-generic, x86_64, Ubuntu 24.04lsb_release -aUbuntu 24.04.4 LTS, Codename: noble
Observed: Modern, stable Ubuntu LTS system. 64-bit kernel. No anomalies.

2. Filesystem Sanity
CommandOutput Summarymkdir /tmp/runbook-demoDirectory created silently — successcp /etc/hosts /tmp/runbook-demo/hosts-copy && ls -lhosts-copy 231 bytes, 
owned by wajih, permissions -rw-r--r--
Observed: Filesystem is writable and healthy. Copy operation succeeded. No permission errors on /tmp.

3. CPU & Memory Snapshot
CommandOutput Summaryfree -hTotal: 11GB RAM, Used: 3.4GB, Free: 5.5GB, Swap used: 0Bps -o pid,pcpu,pmem,comm -p 1014cron PID 1014, CPU: 0.0%, MEM: 0.0%
Observed: System memory is healthy — only 30% RAM in use, zero swap activity. Cron is idle (0% CPU/MEM) as expected for a scheduler waiting between jobs.

4. Disk & IO Snapshot
CommandOutput Summarydf -hMain disk /dev/sda2: 233GB total, 22GB used, 200GB free (10%)du -sh /var/logLogs consuming 228MB — normal sizevmstat 1 5CPU 86–98% idle, 0 swap in/out, 
low disk IO (bi/bo near 0)
Observed: Disk is only 10% full — no space concerns. Log directory is a healthy 228MB. System IO is minimal, no disk pressure detected.

5. Network Snapshot
CommandOutput Summaryss -tulpnPorts open: 53 (DNS), 631 (CUPS printing), Firefox UDP ports. 
All TCP bound to 127.0.0.1 (localhost only)ping -c 4 google.com4/4 packets received, 0% packet loss, avg RTT 19.15ms
Observed: No suspicious open ports. All services bound to localhost only — secure configuration. Internet connectivity confirmed with low latency (19ms).

6. Logs Reviewed
CommandOutput Summaryjournalctl -u cron -n 50Cron running normally. debian-sa1 runs every 10 mins. Hourly jobs fire at :17.
One harmless EXTRA_OPTS warning.tail -n 50 /var/log/syslogDominated by AppArmor DENIED for Firefox MemoryPoller on /proc/pressure/memory. Known Snap sandbox issue — not a threat.
