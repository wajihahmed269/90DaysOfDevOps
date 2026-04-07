# Docker Cheat Sheet

> Personal reference — built during Days 29–37 of #90DaysOfDevOps

---

## Quick Reference Table

| Task | Command |
|------|---------|
| Run a container | `docker run -d -p 8080:80 --name web nginx` |
| Enter a container | `docker exec -it <name> bash` |
| View logs live | `docker logs -f <name>` |
| Build an image | `docker build -t name:tag .` |
| Push to Hub | `docker push user/repo:tag` |
| Start compose stack | `docker compose up -d` |
| Tear down stack | `docker compose down -v` |
| Clean everything | `docker system prune -a` |

---

## Container Commands

```bash
docker run <image>                        # run a container
docker run -d <image>                     # detached (background)
docker run -it <image> bash               # interactive shell
docker run -p 8080:80 <image>             # port mapping host:container
docker run --name myapp <image>           # custom name
docker run -v vol:/path <image>           # attach volume
docker run --network mynet <image>        # attach to network
docker run --rm <image>                   # auto-remove on exit
docker run -e KEY=VALUE <image>           # set env variable

docker ps                                 # list running containers
docker ps -a                              # list all containers
docker stop <name/id>                     # graceful stop (SIGTERM)
docker kill <name/id>                     # immediate stop (SIGKILL)
docker start <name/id>                    # start a stopped container
docker restart <name/id>                  # stop then start
docker rm <name/id>                       # remove container
docker rm -f <name/id>                    # force remove (even running)

docker logs <name>                        # view logs
docker logs -f <name>                     # follow logs live
docker logs --tail 50 <name>             # last 50 lines

docker exec <name> <cmd>                  # run command in container
docker exec -it <name> bash              # interactive shell in container

docker inspect <name>                     # full metadata (JSON)
docker stats                              # live resource usage
docker top <name>                         # processes inside container
```

---

## Image Commands

```bash
docker pull <image>                       # pull from Docker Hub
docker images                             # list local images
docker image ls                           # same as above
docker image history <image>              # show layers
docker image inspect <image>              # full image metadata

docker build -t name:tag .               # build from Dockerfile in .
docker build -t name:tag -f path/Dockerfile .  # specify Dockerfile

docker tag <image> user/repo:tag          # tag an image
docker push user/repo:tag                 # push to Docker Hub
docker login                              # authenticate with Hub

docker rmi <image>                        # remove image
docker rmi -f <image>                     # force remove
```

---

## Volume Commands

```bash
docker volume create <name>               # create named volume
docker volume ls                          # list volumes
docker volume inspect <name>              # details + mountpoint
docker volume rm <name>                   # remove volume
docker volume prune                       # remove unused volumes

# Usage in docker run
-v vol-name:/container/path              # named volume
-v /host/path:/container/path           # bind mount
-v /host/path:/container/path:ro        # read-only bind mount
```

---

## Network Commands

```bash
docker network ls                         # list networks
docker network create <name>              # create bridge network
docker network inspect <name>             # details + connected containers
docker network rm <name>                  # remove network
docker network connect <net> <container>  # connect container to network
docker network disconnect <net> <container>

# Usage in docker run
--network <net-name>                      # attach to network
```

---

## Compose Commands

```bash
docker compose up                         # start all services
docker compose up -d                      # start detached
docker compose up --build                 # rebuild images then start
docker compose up --scale web=3          # scale service to N replicas

docker compose down                       # stop + remove containers + network
docker compose down -v                    # also remove volumes
docker compose stop                       # stop without removing
docker compose start                      # start stopped services
docker compose restart                    # restart all services

docker compose ps                         # list service status
docker compose logs                       # all service logs
docker compose logs -f <service>          # follow specific service
docker compose exec <service> bash        # shell into service

docker compose build                      # build images only
docker compose pull                       # pull latest images
docker compose config                     # validate + show resolved file
```

---

## Cleanup Commands

```bash
docker container prune                    # remove stopped containers
docker image prune                        # remove dangling images
docker image prune -a                     # remove all unused images
docker volume prune                       # remove unused volumes
docker network prune                      # remove unused networks

docker system prune                       # remove all unused resources
docker system prune -a                    # nuclear — removes everything unused
docker system df                          # show disk usage breakdown

# Stop and remove all containers
docker stop $(docker ps -q)
docker rm $(docker ps -aq)
```

---

## Dockerfile Instructions

```dockerfile
FROM image:tag          # base image (required, must be first)
RUN command             # run at BUILD time — creates a new layer
COPY src dest           # copy files from host build context into image
ADD src dest            # like COPY but can untar + fetch URLs
WORKDIR /path           # set working directory (like cd, but persistent)
EXPOSE port             # document port (informational only)
ENV KEY=VALUE           # set environment variable (available at runtime)
ARG NAME=default        # build-time variable (NOT in final image)
CMD ["exec", "arg"]     # default command — overridable at docker run
ENTRYPOINT ["exec"]     # fixed executable — arguments appended at runtime
USER username           # run as non-root user from this point
VOLUME ["/path"]        # declare a mount point
HEALTHCHECK CMD ...     # define a healthcheck for the container
```

---

## Multi-Stage Build Pattern

```dockerfile
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN go build -o app .

FROM alpine:3.19
COPY --from=builder /app/app .
CMD ["./app"]
```

---

# Day 37 – Revision Notes

## Self-Assessment Checklist

| Topic | Status |
|-------|--------|
| Run container — interactive + detached | ✅ Confident |
| List, stop, remove containers and images | ✅ Confident |
| Explain image layers and caching | ✅ Confident |
| Write Dockerfile from scratch | ✅ Confident |
| Explain CMD vs ENTRYPOINT | ✅ Confident |
| Build and tag a custom image | ✅ Confident |
| Create and use named volumes | ✅ Confident |
| Use bind mounts | ✅ Confident |
| Create custom networks, connect containers | ✅ Confident |
| Write docker-compose.yml for multi-container app | ✅ Confident |
| Use environment variables and .env in Compose | ✅ Confident |
| Write a multi-stage Dockerfile | ✅ Confident |
| Push image to Docker Hub | ✅ Confident |
| Use healthchecks and depends_on | ✅ Confident |

---

## Quick-Fire Answers

**1. Difference between an image and a container?**
An image is a read-only blueprint — a stack of layers baked at build time. A container is a running instance of an image — it adds a thin writable layer on top. Same relationship as a class (image) and an object (container).

**2. What happens to data inside a container when you remove it?**
Gone permanently. The container's writable layer is deleted with it. Solution: mount a named volume to the path where data lives — volume persists independently of the container.

**3. How do two containers on the same custom network communicate?**
Docker's embedded DNS server registers every container by its name. Any container on the same custom network can resolve others by service name. `app` can reach `db` just by using `db` as the hostname.

**4. What does `docker compose down -v` do differently from `docker compose down`?**
`docker compose down` removes containers and the default network. `-v` also removes the named volumes defined in the compose file. Without `-v`, your database data survives. With `-v`, it's gone.

**5. Why are multi-stage builds useful?**
Build tools, compilers, and SDKs are needed to build an app but not to run it. Multi-stage builds keep them in a temporary build stage that's discarded. Only the compiled artifact goes into the final image — resulting in images 10–100x smaller and with a much smaller attack surface.

**6. Difference between `COPY` and `ADD`?**
`COPY` just copies files. `ADD` does the same but also auto-extracts `.tar` archives and can fetch files from URLs. Best practice: always use `COPY` unless you specifically need the extra features of `ADD` — it's more predictable.

**7. What does `-p 8080:80` mean?**
Maps host port 8080 to container port 80. Traffic hitting `localhost:8080` on the host gets forwarded to port 80 inside the container. Format: `-p <host>:<container>`.

**8. How do you check how much disk space Docker is using?**
```bash
docker system df
```
Shows breakdown by images, containers, volumes, and build cache — including how much is reclaimable.

---

## Revisited Weak Spots

### Weak Spot 1: Healthchecks in Compose
Redid the `depends_on` + `condition: service_healthy` setup from Day 34. Key reminder: `depends_on` alone only waits for the container to start — not for the process to be ready. The healthcheck is what defines "ready". Without `start_period`, healthchecks fail immediately on slow-starting services like Postgres.

### Weak Spot 2: Multi-Stage with non-Go languages
Tested with Python — the pattern is slightly different. You install dependencies in Stage 1 then `COPY --from=builder /usr/local/lib/python3.11 ...` into Stage 2. It's less dramatic than Go (Python can't compile to a single binary) but still cuts image size by removing build headers, gcc, and package manager caches.
