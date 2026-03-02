Today I focused on understanding how Linux actually works internally.

Linux has three main parts: Kernel, User Space, and systemd (init).

The Kernel is the core of Linux. It manages CPU, memory, disk, and hardware.

User space is where we run commands and applications like bash.

When Linux starts, the first process is PID 1, which is usually systemd.
Processes

Every running program is called a process.

A process can be in different states:

Running (R) – currently using CPU

Sleeping (S) – waiting for something

Stopped (T) – paused

Zombie (Z) – finished but not cleared properly

systemd

systemd starts services when the system boots.

It can start, stop, and restart services.

It also keeps logs of services.

5 Beginner Commands I’ll Use Daily

pwd – check current directory

ls – list files

cd – change directory

ps – see running processes

top – see system usage (CPU/RAM)

Today I understood that if I know how processes and systemd work, I can slowly start troubleshooting problems in Linux systems with more confidence.
