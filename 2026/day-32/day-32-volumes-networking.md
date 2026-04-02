# Day 32 – Docker Volumes & Networking

## Task 1: The Problem — Containers Are Ephemeral

```bash
# Run a Postgres container
docker run -d \
  --name pg-test \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=testdb \
  postgres

# Wait for it to start, then connect
docker exec -it pg-test psql -U postgres -d testdb

# Create a table and insert data
CREATE TABLE users (id SERIAL PRIMARY KEY, name VARCHAR(50));
INSERT INTO users (name) VALUES ('Wajih'), ('DevOps');
SELECT * FROM users;
# 1 | Wajih
# 2 | DevOps
\q

# Stop and remove the container
docker stop pg-test && docker rm pg-test

# Spin up a brand new container
docker run -d \
  --name pg-test2 \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=testdb \
  postgres

docker exec -it pg-test2 psql -U postgres -d testdb
SELECT * FROM users;
```

```
ERROR:  relation "users" does not exist
```

**What happened and why:**

The data is gone. Completely. When a container is removed, its writable layer — the thin layer on top of the read-only image layers where all runtime changes live — is deleted with it. The Postgres image has no idea about any data you created because that data never made it into the image. It only existed in the container's ephemeral filesystem.

This is by design. Containers are meant to be disposable. The problem is databases are not.

---

## Task 2: Named Volumes

```bash
# Create a named volume
docker volume create pg-data

# Verify
docker volume ls
```

```
DRIVER    VOLUME NAME
local     pg-data
```

```bash
# Run Postgres with the volume attached
docker run -d \
  --name pg-vol \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=testdb \
  -v pg-data:/var/lib/postgresql/data \
  postgres

# Add data
docker exec -it pg-vol psql -U postgres -d testdb
CREATE TABLE users (id SERIAL PRIMARY KEY, name VARCHAR(50));
INSERT INTO users (name) VALUES ('Wajih'), ('DevOps'), ('Docker');
SELECT * FROM users;
# 1 | Wajih
# 2 | DevOps
# 3 | Docker
\q

# Stop and REMOVE the container
docker stop pg-vol && docker rm pg-vol

# Spin up a brand new container — attach the SAME volume
docker run -d \
  --name pg-vol2 \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=testdb \
  -v pg-data:/var/lib/postgresql/data \
  postgres

docker exec -it pg-vol2 psql -U postgres -d testdb
SELECT * FROM users;
```

```
 id | name
----+--------
  1 | Wajih
  2 | DevOps
  3 | Docker
(3 rows)
```

Data survived. Container is gone, data is not.

```bash
# Inspect the volume
docker volume inspect pg-data
```

```json
[
  {
    "Name": "pg-data",
    "Driver": "local",
    "Mountpoint": "/var/lib/docker/volumes/pg-data/_data",
    "Scope": "local"
  }
]
```

The data lives at `/var/lib/docker/volumes/pg-data/_data` on the host — managed by Docker, completely separate from any container.

---

## Task 3: Bind Mounts

```bash
# Create host folder and HTML file
mkdir -p ~/docker-web
cat > ~/docker-web/index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
  <style>
    body { background: #0a0e1a; color: #00ffb2; font-family: monospace;
           display: flex; justify-content: center; align-items: center; height: 100vh; margin: 0; }
    h1 { font-size: 2.5rem; }
  </style>
</head>
<body>
  <div>
    <h1>🐳 Bind Mount — Day 32</h1>
    <p style="color:rgba(255,255,255,0.5)">#90DaysOfDevOps</p>
  </div>
</body>
</html>
EOF

# Run Nginx with bind mount
docker run -d \
  --name bind-demo \
  -p 8080:80 \
  -v ~/docker-web:/usr/share/nginx/html \
  nginx
```

Access `http://localhost:8080` → custom page loads.

```bash
# Edit file on host — no container restart needed
echo "<p style='color:#fff'>Live edit — $(date)</p>" >> ~/docker-web/index.html
```

Refresh browser → change appears immediately. The container reads directly from the host filesystem.

---

### Named Volume vs Bind Mount

| | Named Volume | Bind Mount |
|--|-------------|-----------|
| **Managed by** | Docker | You (host path) |
| **Location** | `/var/lib/docker/volumes/` | Any path you specify |
| **Use case** | Database data, persistent app state | Dev workflows, live file editing |
| **Portability** | Works the same on any Docker host | Depends on host path existing |
| **Performance** | Better on Linux | Slightly slower on Mac/Windows |
| **Best for** | Production data | Development / local testing |

**Simple rule:** Use named volumes for data you need to keep. Use bind mounts when you want the container to read live files from your machine.

---

## Task 4: Docker Networking Basics

```bash
# List all networks
docker network ls
```

```
NETWORK ID     NAME      DRIVER    SCOPE
a1b2c3d4e5f6   bridge    bridge    local
f7e8d9c0b1a2   host      host      local
g3h4i5j6k7l8   none      null      local
```

```bash
# Inspect default bridge
docker network inspect bridge
```

Key output:
```json
{
  "Name": "bridge",
  "Driver": "bridge",
  "IPAM": {
    "Config": [{"Subnet": "172.17.0.0/16", "Gateway": "172.17.0.1"}]
  },
  "Containers": {}
}
```

```bash
# Run two containers on default bridge
docker run -d --name c1 alpine sleep 3600
docker run -d --name c2 alpine sleep 3600

# Try pinging by NAME — fails
docker exec c1 ping c2
# ping: bad address 'c2'

# Try pinging by IP — works
docker inspect c2 | grep IPAddress
# "IPAddress": "172.17.0.3"

docker exec c1 ping 172.17.0.3
# PING 172.17.0.3: 56 data bytes
# 64 bytes from 172.17.0.3: seq=0 ttl=64 time=0.142 ms
```

**Result:** Default bridge allows IP-to-IP communication but not name resolution. You have to hardcode IPs — which defeats the purpose since IPs change every time containers restart.

---

## Task 5: Custom Networks

```bash
# Create custom bridge network
docker network create my-app-net

# Run two containers on the custom network
docker run -d --name app1 --network my-app-net alpine sleep 3600
docker run -d --name app2 --network my-app-net alpine sleep 3600

# Ping by NAME — works
docker exec app1 ping app2
```

```
PING app2 (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.098 ms
64 bytes from 172.18.0.3: seq=1 ttl=64 time=0.091 ms
```

**Why custom networks allow name resolution but default bridge doesn't:**

The default bridge network is a legacy feature — it predates Docker's built-in DNS. It uses basic Linux bridging with no service discovery.

Custom networks use Docker's **embedded DNS server** (runs at `127.0.0.11` inside containers). When you create a container on a custom network, Docker registers its name in this DNS server. Any other container on the same network can resolve it by name. The default bridge simply doesn't have this — it's kept as-is for backward compatibility.

In production, you always create custom networks. Hardcoding IPs is not an option when containers restart with new IPs constantly.

---

## Task 6: Put It Together — Full Stack Setup

```bash
# 1. Create custom network
docker network create app-stack

# 2. Run Postgres with volume on the network
docker run -d \
  --name postgres-db \
  --network app-stack \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=appdb \
  -v app-pgdata:/var/lib/postgresql/data \
  postgres

# 3. Run an app container on the same network
docker run -d \
  --name app-server \
  --network app-stack \
  alpine sleep 3600

# 4. Verify app-server can reach postgres-db by name
docker exec app-server ping postgres-db
```

```
PING postgres-db (172.19.0.2): 56 data bytes
64 bytes from 172.19.0.2: seq=0 ttl=64 time=0.213 ms
```

```bash
# Verify volume was created
docker volume ls | grep app-pgdata
# local     app-pgdata

# Verify both containers are on the same network
docker network inspect app-stack | grep -A3 '"Containers"'
```

```json
"Containers": {
  "...": {"Name": "postgres-db", "IPv4Address": "172.19.0.2/16"},
  "...": {"Name": "app-server",  "IPv4Address": "172.19.0.3/16"}
}
```

**What this setup gives you:**
- `app-server` talks to `postgres-db` by name — no hardcoded IPs
- Postgres data survives container restarts and replacements
- Both containers are isolated from everything else on the host
- This is exactly how Docker Compose works under the hood — it just automates these steps

---

## Summary: Commands Used Today

```bash
# Volumes
docker volume create <name>              # create named volume
docker volume ls                         # list volumes
docker volume inspect <name>             # inspect volume details
docker volume rm <name>                  # remove volume

# Running with volumes
docker run -v vol-name:/container/path   # named volume
docker run -v /host/path:/container/path # bind mount

# Networking
docker network ls                        # list networks
docker network create <name>             # create custom network
docker network inspect <name>            # inspect network
docker network rm <name>                 # remove network

# Running with networking
docker run --network <net-name>          # attach to network

# Verify connectivity
docker exec <c1> ping <c2>              # ping between containers
docker inspect <container> | grep IP    # find container IP
```
