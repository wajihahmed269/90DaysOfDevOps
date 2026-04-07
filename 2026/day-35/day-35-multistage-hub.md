# Day 35 – Multi-Stage Builds & Docker Hub

## Task 1: Single-Stage Build — The Problem

### Simple Go App — `main.go`
```go
package main

import "fmt"

func main() {
    fmt.Println("Hello from Docker — Day 35!")
}
```

### Single-Stage `Dockerfile`
```dockerfile
FROM golang:1.21

WORKDIR /app
COPY main.go .
RUN go build -o hello .

CMD ["./hello"]
```

```bash
docker build -t hello-single:v1 .
docker images hello-single
```

```
REPOSITORY       TAG    IMAGE ID       SIZE
hello-single     v1     a3b4c5d6e7f8   842MB
```

**842MB** to run a binary that prints one line. The entire Go SDK, build tools, and standard library are baked in — none of which are needed at runtime.

---

## Task 2: Multi-Stage Build — The Fix

### Multi-Stage `Dockerfile`
```dockerfile
# ── Stage 1: Build ──────────────────────────────────────────
FROM golang:1.21 AS builder

WORKDIR /app
COPY main.go .
RUN CGO_ENABLED=0 GOOS=linux go build -o hello .

# ── Stage 2: Run ─────────────────────────────────────────────
FROM alpine:3.19

WORKDIR /app
COPY --from=builder /app/hello .

CMD ["./hello"]
```

```bash
docker build -t hello-multi:v1 .
docker images | grep hello
```

```
REPOSITORY      TAG    IMAGE ID       SIZE
hello-single    v1     a3b4c5d6e7f8   842MB
hello-multi     v1     b9c8d7e6f5a4   7.61MB
```

**842MB → 7.6MB. 99% smaller.**

### Why Multi-Stage is So Much Smaller

The final image contains **only what Stage 2 needs** — the compiled binary + Alpine's minimal OS. Everything from Stage 1 (Go compiler, source code, build cache, standard library source) is discarded. Docker never includes previous stage layers in the final image unless you explicitly `COPY --from=` them.

Stage 1 is a temporary build environment. Stage 2 is the shipping container. The SDK stays in Stage 1 and never ships.

---

## Task 3: Push to Docker Hub

```bash
# Login
docker login
# Username: wajihahmed269
# Password: ********
# Login Succeeded

# Tag the image properly
docker tag hello-multi:v1 wajihahmed269/hello-go:v1
docker tag hello-multi:v1 wajihahmed269/hello-go:latest

# Push
docker push wajihahmed269/hello-go:v1
docker push wajihahmed269/hello-go:latest
```

```
The push refers to repository [docker.io/wajihahmed269/hello-go]
v1: digest: sha256:abc123... size: 1847
latest: digest: sha256:abc123... size: 1847
```

```bash
# Verify — remove local image and pull from Hub
docker rmi wajihahmed269/hello-go:v1
docker pull wajihahmed269/hello-go:v1
docker run wajihahmed269/hello-go:v1
# Hello from Docker — Day 35!
```

---

## Task 4: Docker Hub Repository

- Repo: `https://hub.docker.com/r/wajihahmed269/hello-go`
- Added description: "Minimal Go Hello World — built with multi-stage Dockerfile. Final image: ~7MB on Alpine."
- Tags visible: `v1`, `latest`

**`latest` vs specific tag:**
```bash
docker pull wajihahmed269/hello-go:latest   # always pulls most recent push
docker pull wajihahmed269/hello-go:v1       # pinned — always same image
```

In production: always pin to specific tags. `latest` is a moving target — pulling it in a CI pipeline can break builds when a new version is pushed. `v1`, `v1.2.3`, or a commit SHA are stable references.

---

## Task 5: Image Best Practices

### Before
```dockerfile
FROM ubuntu
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y wget
RUN apt-get clean
CMD ["bash"]
```

```
SIZE: 183MB   LAYERS: 6 extra layers
```

### After — All Best Practices Applied
```dockerfile
# 1. Specific tag — not latest
FROM alpine:3.19

# 2. Combine RUN commands — single layer, no cache bloat
RUN apk add --no-cache curl wget

# 3. Non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

WORKDIR /home/appuser

CMD ["sh"]
```

```bash
docker build -t optimized:v1 .
docker images optimized
```

```
REPOSITORY   TAG    SIZE
optimized    v1     9.23MB
```

**183MB → 9MB**

### Best Practices Summary

| Practice | Why |
|----------|-----|
| Use `alpine` or `slim` base | Smallest attack surface, fastest pulls |
| Pin base image tags (`alpine:3.19` not `alpine`) | Reproducible builds — base won't change unexpectedly |
| Combine `RUN` commands with `&&` | Fewer layers, smaller image |
| `--no-cache` on apk/apt | Don't bake package manager cache into image |
| Non-root `USER` | Containers running as root are a security risk |
| `.dockerignore` | Exclude build artifacts, `.git`, `.env` from context |
| Multi-stage builds | Zero build tools in production image |

---

## Summary: Commands Used Today

```bash
# Multi-stage build
docker build -t name:tag .
docker images                         # check sizes

# Docker Hub
docker login                          # authenticate
docker tag local:tag user/repo:tag    # tag for Hub
docker push user/repo:tag             # push to Hub
docker pull user/repo:tag             # pull from Hub
docker rmi user/repo:tag              # remove local copy

# Inspect
docker image history image:tag        # see all layers and sizes
docker inspect image:tag              # full image metadata
```

**Docker Hub repo:** `https://hub.docker.com/r/wajihahmed269/hello-go`
