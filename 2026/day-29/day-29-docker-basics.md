# Day 29 – Introduction to Docker

## Task 1: What is Docker?

### What is a Container and Why Do We Need It?

A container is a **lightweight, isolated environment** that packages an application together with everything it needs to run — code, runtime, libraries, config — into a single unit. It runs directly on the host OS kernel without needing a full operating system of its own.

**Why we need them:**
The classic problem: *"It works on my machine."* A developer writes code on Ubuntu, the server runs CentOS, a teammate uses macOS — and something always breaks in the gap between environments. Containers eliminate that gap. The app runs in the exact same environment everywhere — dev laptop, CI server, production.

---

### Containers vs Virtual Machines

| | Virtual Machine | Container |
|--|----------------|-----------|
| **What it virtualizes** | Full hardware (CPU, RAM, disk) | Just the process environment |
| **Includes** | Full OS + kernel | App + libraries only |
| **Size** | GBs | MBs |
| **Boot time** | Minutes | Seconds (usually < 1s) |
| **Isolation** | Strong — separate kernel | Process-level — shared kernel |
| **Resource use** | Heavy | Very lightweight |
| **Use case** | Full OS isolation needed | App packaging and deployment |

**Simple analogy:** A VM is renting an entire house. A container is renting a room in a shared house — same building, shared infrastructure, but your space is yours.

---

### Docker Architecture

```
┌─────────────────────────────────────────────────────┐
│                   Docker Client                      │
│         (docker run, docker build, docker ps)        │
└───────────────────────┬─────────────────────────────┘
                        │ REST API
                        ▼
┌─────────────────────────────────────────────────────┐
│                  Docker Daemon (dockerd)             │
│                                                      │
│   ┌──────────────┐   ┌──────────────────────────┐   │
│   │    Images    │   │       Containers          │   │
│   │  (templates) │   │  (running instances)      │   │
│   └──────────────┘   └──────────────────────────┘   │
└─────────────────────────────┬───────────────────────┘
                              │ pull / push
                              ▼
┌─────────────────────────────────────────────────────┐
│               Docker Registry (Docker Hub)           │
│         (stores and distributes images)              │
└─────────────────────────────────────────────────────┘
```

**In plain English:**
- **Docker Client** — the CLI you type commands into (`docker run`, `docker build`)
- **Docker Daemon (`dockerd`)** — the background service that actually does the work; the client talks to it via REST API
- **Image** — a read-only template/blueprint (like a class in programming)
- **Container** — a running instance of an image (like an object created from a class)
- **Registry** — a storage hub for images; Docker Hub is the public default, you can run private ones too

---

## Task 2: Install Docker

```bash
# Install Docker (Ubuntu)
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker

# Add your user to docker group so you don't need sudo every time
sudo usermod -aG docker $USER
newgrp docker

# Verify installation
docker --version
# Docker version 24.0.7, build afdd53b

docker info
```

### Run hello-world

```bash
docker run hello-world
```

**Output:**
```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
719385e32844: Pull complete
Digest: sha256:88ec0acaa3ec...
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.
```

**What just happened:** Docker couldn't find the image locally → pulled it from Docker Hub → created a container → ran it → container printed the message → exited. The output itself is Docker explaining its own architecture — read it carefully.

---

## Task 3: Run Real Containers

### Nginx Container

```bash
# Run Nginx with port mapping
docker run -d -p 8080:80 --name my-nginx nginx
```

```
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
...
Status: Downloaded newer image for nginx:latest
a3f5d9c1e8b2...
```

Access `http://localhost:8080` in browser → Nginx welcome page appears.

---

### Ubuntu Interactive Container

```bash
docker run -it ubuntu bash
```

```
root@3f7d9c2e1a4b:/# ls
bin  boot  dev  etc  home  lib  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

root@3f7d9c2e1a4b:/# cat /etc/os-release
PRETTY_NAME="Ubuntu 22.04.3 LTS"

root@3f7d9c2e1a4b:/# exit
```

It's a full Linux shell inside a container. Completely isolated — anything you install or break here doesn't touch the host.

---

### List, Stop, Remove

```bash
# List running containers
docker ps

# List ALL containers (including stopped)
docker ps -a

# Stop a container
docker stop my-nginx

# Remove a container
docker rm my-nginx
```

**Output of `docker ps`:**
```
CONTAINER ID   IMAGE   COMMAND                  CREATED         STATUS         PORTS                  NAMES
a3f5d9c1e8b2   nginx   "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes   0.0.0.0:8080->80/tcp   my-nginx
```

---

## Task 4: Explore

```bash
# Detached mode — runs in background, returns container ID immediately
docker run -d --name bg-nginx -p 9090:80 nginx
# Returns: b7e2f1a4c3d9...

# Custom name
docker run -d --name my-webserver nginx

# Port mapping — host port 8888 → container port 80
docker run -d -p 8888:80 --name port-demo nginx

# Check logs
docker logs bg-nginx

# Real-time logs
docker logs -f bg-nginx

# Run command inside running container
docker exec bg-nginx ls /etc/nginx

# Enter interactive shell in running container
docker exec -it bg-nginx bash
```

**Output of `docker exec bg-nginx ls /etc/nginx`:**
```
conf.d          mime.types  nginx.conf  
fastcgi_params  modules     uwsgi_params
```

**Detached vs Interactive:**
- `-d` (detached) → container runs in background, you get your terminal back immediately, container keeps running
- `-it` (interactive) → your terminal is attached to the container's shell, container stops when you exit

---

## Summary: Commands Used Today

```bash
docker run hello-world                        # run a container
docker run -d -p 8080:80 --name name image   # detached + port map + name
docker run -it ubuntu bash                    # interactive shell
docker ps                                     # list running containers
docker ps -a                                  # list all containers
docker stop <name/id>                         # stop container
docker rm <name/id>                           # remove container
docker logs <name>                            # view logs
docker logs -f <name>                         # follow logs live
docker exec <name> <cmd>                      # run command in container
docker exec -it <name> bash                   # enter container shell
```
