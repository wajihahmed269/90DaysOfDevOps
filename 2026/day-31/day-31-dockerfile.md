# Day 31 – Dockerfile: Build Your Own Images

## Task 1: Your First Dockerfile

### Directory Structure

```
my-first-image/
└── Dockerfile
```

### Dockerfile

```dockerfile
FROM ubuntu

RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*

CMD ["echo", "Hello from my custom image!"]
```

### Build and Run

```bash
docker build -t my-ubuntu:v1 .
```

```
[+] Building 23.4s (6/6) FINISHED
 => [internal] load build definition from Dockerfile
 => [internal] load .dockerignore
 => [1/2] FROM docker.io/library/ubuntu
 => [2/2] RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
 => exporting to image
 => => naming to docker.io/library/my-ubuntu:v1
```

```bash
docker run my-ubuntu:v1
```

```
Hello from my custom image!
```

---

## Task 2: All Dockerfile Instructions

### Directory Structure

```
full-demo/
├── Dockerfile
└── app.sh
```

### `app.sh`

```bash
#!/bin/bash
echo "App is running in: $(pwd)"
echo "Hostname: $(hostname)"
```

### Dockerfile

```dockerfile
# Base image
FROM ubuntu:22.04

# Run commands during build — install packages, setup environment
RUN apt-get update && \
    apt-get install -y curl bash && \
    rm -rf /var/lib/apt/lists/*

# Set working directory — all subsequent commands run from here
WORKDIR /app

# Copy files from host into image at build time
COPY app.sh .

# Make script executable
RUN chmod +x app.sh

# Document the port this app uses (informational only — doesn't actually publish it)
EXPOSE 8080

# Default command — what runs when container starts
CMD ["./app.sh"]
```

### Build and Run

```bash
docker build -t full-demo:v1 .
docker run full-demo:v1
```

```
App is running in: /app
Hostname: 7f2d1e4b3a9c
```

### What Each Instruction Does

| Instruction | What it does |
|-------------|-------------|
| `FROM` | Sets the base image — every Dockerfile must start with this |
| `RUN` | Executes a command **at build time** — result is baked into a new layer |
| `COPY` | Copies files from host build context into the image |
| `WORKDIR` | Sets the working directory — like `cd` but persistent for all following instructions |
| `EXPOSE` | Documents which port the app uses — does NOT publish it (use `-p` at runtime for that) |
| `CMD` | Default command run when container starts — can be overridden at `docker run` |

---

## Task 3: CMD vs ENTRYPOINT

### CMD Demo

```dockerfile
FROM ubuntu
CMD ["echo", "hello from CMD"]
```

```bash
docker build -t cmd-demo .

# Normal run — CMD executes
docker run cmd-demo
# hello from CMD

# Override CMD — your command replaces it entirely
docker run cmd-demo echo "overridden!"
# overridden!

# Override CMD with something else entirely
docker run cmd-demo ls /etc
# (lists /etc directory — CMD is completely replaced)
```

---

### ENTRYPOINT Demo

```dockerfile
FROM ubuntu
ENTRYPOINT ["echo"]
```

```bash
docker build -t ep-demo .

# Normal run — ENTRYPOINT runs with no args
docker run ep-demo
# (prints blank line)

# Pass arguments — they are APPENDED to ENTRYPOINT
docker run ep-demo "hello from entrypoint"
# hello from entrypoint

docker run ep-demo "DevOps" "is" "fun"
# DevOps is fun
```

---

### CMD vs ENTRYPOINT — When to Use Which

| | `CMD` | `ENTRYPOINT` |
|--|-------|-------------|
| **Behavior** | Default command — fully replaceable at runtime | Fixed executable — arguments appended at runtime |
| **Override** | Completely replaced by `docker run <image> <cmd>` | Arguments appended; entrypoint stays fixed |
| **Use case** | Flexible containers where the command might change | Containers that should always run one specific executable |
| **Example** | A dev container where you might run bash, python, or tests | A CLI tool image where the tool is always the entrypoint |

**Best practice — combine both:**
```dockerfile
ENTRYPOINT ["nginx"]
CMD ["-g", "daemon off;"]
```
`ENTRYPOINT` locks in `nginx`, `CMD` provides default flags. You can override just the flags without replacing the entrypoint.

---

## Task 4: Build a Simple Web App Image

### Directory Structure

```
my-website/
├── Dockerfile
└── index.html
```

### `index.html`

```html
<!DOCTYPE html>
<html>
<head>
  <title>Day 31 – Docker</title>
  <style>
    body { font-family: monospace; background: #0a0e1a; color: #00ffb2; display: flex; justify-content: center; align-items: center; height: 100vh; margin: 0; }
    h1 { font-size: 2rem; }
    p  { color: rgba(255,255,255,0.6); }
  </style>
</head>
<body>
  <div>
    <h1>🐳 Running in Docker</h1>
    <p>Day 31 – #90DaysOfDevOps</p>
    <p>Built with nginx:alpine + a custom Dockerfile</p>
  </div>
</body>
</html>
```

### Dockerfile

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html
```

### Build and Run

```bash
docker build -t my-website:v1 .
docker run -d -p 8080:80 --name my-site my-website:v1
```

Access `http://localhost:8080` → custom page displays.

```bash
# Verify the file is in the right place inside the container
docker exec my-site cat /usr/share/nginx/html/index.html
```

---

## Task 5: `.dockerignore`

```
my-project/
├── Dockerfile
├── .dockerignore
├── app.sh
├── .env               ← should NOT go into image
├── README.md          ← not needed in image
└── .git/              ← never send this to Docker daemon
```

### `.dockerignore`

```
node_modules
.git
*.md
.env
*.log
__pycache__
.DS_Store
```

**Why it matters:** When you run `docker build .`, Docker sends the entire build context (that folder) to the Docker daemon. Without `.dockerignore`, it sends everything — including your `.git` folder, `node_modules`, `.env` files with secrets, and gigabytes of dependencies. `.dockerignore` works exactly like `.gitignore` — list what to exclude.

```bash
docker build -t ignore-demo .
# [+] Building 1.2s
# => [internal] load build context
# => => transferring context: 1.24kB   ← small because .dockerignore excluded the big stuff
```

---

## Task 6: Build Optimization — Layer Caching

### Why Layer Order Matters

Docker builds layers top to bottom. When you rebuild, Docker reuses cached layers **until it finds a change**. Everything after the change gets rebuilt.

### Bad Order (slow rebuilds)

```dockerfile
FROM node:18-alpine
COPY . .                           # ← copies everything including app code
RUN npm install                    # ← runs AFTER copy, so ANY code change invalidates cache
CMD ["node", "app.js"]
```

Every time you change one line of app code, `COPY . .` invalidates the cache and `npm install` runs again — even though your dependencies didn't change.

### Good Order (fast rebuilds)

```dockerfile
FROM node:18-alpine
COPY package*.json ./              # ← copy only dependency files first
RUN npm install                    # ← runs ONLY when package.json changes
COPY . .                           # ← app code comes AFTER installs
CMD ["node", "app.js"]
```

Now `npm install` only re-runs when `package.json` changes. Changing your app code only rebuilds the last `COPY` layer — taking seconds instead of minutes.

### General Rule

> **Put things that change rarely at the top. Put things that change often at the bottom.**

```
FROM (never changes)
RUN apt-get install (rarely changes)
COPY dependencies (occasionally changes)
RUN install dependencies (occasionally changes)
COPY app code (changes every commit)
CMD (rarely changes)
```

### Cache Demo

```bash
# First build — everything runs
docker build -t cache-demo:v1 .
# [+] Building 24.3s

# Change nothing, rebuild — all cached
docker build -t cache-demo:v2 .
# [+] Building 0.8s  ← cache hit on every layer

# Change app.sh, rebuild — only last COPY rebuilds
docker build -t cache-demo:v3 .
# [+] Building 1.1s  ← only layers after the changed COPY re-run
```

---

## Summary: Dockerfile Instructions & Build Commands

```bash
# Build
docker build -t name:tag .          # build image from Dockerfile in current dir
docker build -t name:tag -f path    # specify a different Dockerfile path

# Tag
docker tag my-image:v1 my-image:latest   # add another tag to an image

# Dockerfile Instructions
FROM <image>                        # base image (required, must be first)
RUN <command>                       # run command at BUILD time (new layer)
COPY <src> <dest>                   # copy files from host to image
ADD <src> <dest>                    # like COPY but can untar and fetch URLs
WORKDIR /path                       # set working directory
EXPOSE <port>                       # document port (does not publish)
ENV KEY=VALUE                       # set environment variable
ARG NAME=default                    # build-time variable (not in final image)
CMD ["executable", "arg"]           # default command (overridable)
ENTRYPOINT ["executable"]           # fixed entrypoint (args appended)
```
