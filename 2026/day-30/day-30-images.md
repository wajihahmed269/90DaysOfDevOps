# Day 30 – Docker Images & Container Lifecycle

## Task 1: Docker Images

### Pull Images

```bash
docker pull nginx
docker pull ubuntu
docker pull alpine
```

### List All Images

```bash
docker images
```

```
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
nginx        latest    a6bd71f48f68   2 weeks ago    187MB
ubuntu       latest    5a81c4b8502e   3 weeks ago    77.9MB
alpine       latest    c157a85ed455   4 weeks ago    7.38MB
```

### Ubuntu vs Alpine — Why is Alpine So Much Smaller?

| | Ubuntu | Alpine |
|--|--------|--------|
| **Size** | ~78MB | ~7MB |
| **Base** | Full GNU/Linux userland | Musl libc + BusyBox |
| **Package manager** | apt | apk |
| **Use case** | Familiar, great for dev/testing | Production images where size matters |

**The real difference:** Ubuntu ships with a full set of utilities, tools, and libraries — great for humans to work with. Alpine is built for machines. It uses `musl libc` instead of `glibc` and replaces the standard GNU coreutils with BusyBox — a single binary that mimics hundreds of Unix commands. The result is a 10x size reduction. In production, smaller images = faster pulls, smaller attack surface, lower storage costs.

---

### Inspect an Image

```bash
docker image inspect nginx
```

**Key info from output:**
```json
{
  "Id": "sha256:a6bd71f48f68...",
  "RepoTags": ["nginx:latest"],
  "Architecture": "amd64",
  "Os": "linux",
  "Size": 196458624,
  "RootFS": {
    "Type": "layers",
    "Layers": [
      "sha256:5498e8c22f69...",
      "sha256:a6d9e4b3c71f...",
      "sha256:9c1e4f2d8b3a..."
    ]
  },
  "Config": {
    "ExposedPorts": {"80/tcp": {}},
    "Cmd": ["nginx", "-g", "daemon off;"],
    "WorkingDir": ""
  }
}
```

### Remove an Image

```bash
docker rmi alpine
# Untagged: alpine:latest
# Deleted: sha256:c157a85ed455...
```

> Can't remove an image that has a running or stopped container using it — stop and remove the container first.

---

## Task 2: Image Layers

```bash
docker image history nginx
```

```
IMAGE          CREATED        CREATED BY                                      SIZE
a6bd71f48f68   2 weeks ago    CMD ["nginx" "-g" "daemon off;"]                0B
<missing>      2 weeks ago    STOPSIGNAL SIGQUIT                              0B
<missing>      2 weeks ago    EXPOSE 80                                       0B
<missing>      2 weeks ago    ENTRYPOINT ["/docker-entrypoint.sh"]            0B
<missing>      2 weeks ago    COPY 30-tune-worker-processes.sh /docker-en…   4.62kB
<missing>      2 weeks ago    COPY 20-envsubst-on-templates.sh /docker-en…   3.02kB
<missing>      2 weeks ago    RUN /bin/sh -c apt-get update && apt-get in…   61.1MB
<missing>      2 weeks ago    ENV NGINX_VERSION=1.25.3                       0B
<missing>      3 weeks ago    /bin/sh -c #(nop) ADD file:... in /          77.8MB
```

### What Are Layers and Why Does Docker Use Them?

Every `RUN`, `COPY`, and `ADD` instruction in a Dockerfile creates a new **read-only layer** on top of the previous one. The final image is the stack of all these layers.

**Why layers exist:**

1. **Caching** — if a layer hasn't changed, Docker reuses it from cache during rebuilds. Only changed layers and everything after them get rebuilt. This makes builds dramatically faster.

2. **Sharing** — if two images share the same base layer (e.g., both use `ubuntu:22.04`), Docker stores that layer once and both images reference it. Saves disk space.

3. **Efficiency** — when you pull an image you already have some layers of, Docker only downloads the missing layers.

Think of it like Git commits — each layer is a diff from the previous state.

---

## Task 3: Container Lifecycle

```bash
# 1. Create container WITHOUT starting it
docker create --name lifecycle-demo nginx
# status: Created

docker ps -a
# STATUS: Created

# 2. Start the container
docker start lifecycle-demo
# status: Up

# 3. Pause the container
docker pause lifecycle-demo
# status: Paused (processes frozen, still in memory)

docker ps -a
# STATUS: Up X seconds (Paused)

# 4. Unpause
docker unpause lifecycle-demo
# status: Up (running again)

# 5. Stop (graceful — sends SIGTERM, waits, then SIGKILL)
docker stop lifecycle-demo
# status: Exited (0)

# 6. Restart
docker restart lifecycle-demo
# status: Up

# 7. Kill (immediate — sends SIGKILL, no grace period)
docker kill lifecycle-demo
# status: Exited (137)

# 8. Remove
docker rm lifecycle-demo
```

### State Diagram

```
         docker create
              │
              ▼
          [Created]
              │
        docker start
              │
              ▼
          [Running] ◄────────────────┐
              │                      │
        docker pause           docker restart
              │                      │
              ▼                      │
          [Paused]             [Exited/Stopped]
              │                      ▲
       docker unpause         docker stop / kill
              │                      │
              └──────────────────────┘
```

**Key difference — stop vs kill:**
- `docker stop` → sends `SIGTERM` first, gives the container 10 seconds to shut down gracefully, then sends `SIGKILL`. Clean.
- `docker kill` → sends `SIGKILL` immediately. No grace period. Use when a container is frozen.

---

## Task 4: Working with Running Containers

```bash
# Run Nginx detached
docker run -d -p 8080:80 --name web nginx

# View logs
docker logs web

# Follow logs in real time (like tail -f)
docker logs -f web

# Exec into container filesystem
docker exec -it web bash
# Inside: ls /etc/nginx, cat /etc/nginx/nginx.conf, exit

# Run single command without entering shell
docker exec web cat /usr/share/nginx/html/index.html

# Inspect container — get IP, ports, mounts, env vars
docker inspect web
```

**Finding IP address from inspect:**
```bash
docker inspect web | grep IPAddress
# "IPAddress": "172.17.0.2"
```

**Finding port mappings:**
```bash
docker inspect web | grep -A5 Ports
# "Ports": {
#   "80/tcp": [{"HostIp": "0.0.0.0", "HostPort": "8080"}]
# }
```

---

## Task 5: Cleanup

```bash
# Stop ALL running containers in one command
docker stop $(docker ps -q)

# Remove ALL stopped containers in one command
docker container prune
# WARNING! This will remove all stopped containers.
# Are you sure you want to continue? [y/N] y
# Deleted Containers: a3f5d9c1e8b2...
# Total reclaimed space: 2.4MB

# Remove unused images (not referenced by any container)
docker image prune
# OR remove ALL unused images (not just dangling ones)
docker image prune -a

# Check how much disk space Docker is using
docker system df
```

**Output of `docker system df`:**
```
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          4         1         456MB     301MB (66%)
Containers      2         1         1.2MB     0B (0%)
Local Volumes   0         0         0B        0B
Build Cache     12        0         84MB      84MB
```

**Nuclear option — remove everything unused:**
```bash
docker system prune -a
# Removes: stopped containers, unused networks, unused images, build cache
```

---

## Summary: Commands Used Today

```bash
docker pull <image>                   # pull image from Docker Hub
docker images                         # list all local images
docker image history <image>          # show image layers
docker image inspect <image>          # detailed image metadata
docker rmi <image>                    # remove image

docker create --name <n> <image>   # create without starting
docker start <n>                   # start a created/stopped container
docker pause <n>                   # freeze container processes
docker unpause <n>                 # resume frozen container
docker stop <n>                    # graceful stop (SIGTERM)
docker kill <n>                    # immediate stop (SIGKILL)
docker restart <n>                 # stop then start
docker rm <n>                      # remove container

docker logs <n>                    # view container logs
docker logs -f <n>                 # follow logs live
docker exec -it <n> bash           # interactive shell in container
docker exec <n> <cmd>              # run command in container
docker inspect <n>                 # full container metadata

docker stop $(docker ps -q)           # stop all running containers
docker container prune                # remove all stopped containers
docker image prune -a                 # remove unused images
docker system df                      # show disk usage
docker system prune -a                # remove all unused Docker data
```
