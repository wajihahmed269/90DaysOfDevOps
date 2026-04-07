# Day 34 – Docker Compose: Real-World Multi-Container Apps

## Task 1: 3-Service App Stack

### Directory Structure
```
app-stack/
├── app/
│   ├── app.py
│   ├── requirements.txt
│   └── Dockerfile
├── .env
└── docker-compose.yml
```

### `app/app.py`
```python
from flask import Flask, jsonify
import psycopg2
import redis
import os

app = Flask(__name__)

@app.route("/")
def index():
    # Redis
    r = redis.Redis(host=os.getenv("REDIS_HOST", "redis"), port=6379)
    visits = r.incr("visits")

    # Postgres
    conn = psycopg2.connect(
        host=os.getenv("DB_HOST", "db"),
        database=os.getenv("DB_NAME", "appdb"),
        user=os.getenv("DB_USER", "appuser"),
        password=os.getenv("DB_PASSWORD", "apppass")
    )
    cur = conn.cursor()
    cur.execute("SELECT version();")
    db_version = cur.fetchone()[0]
    conn.close()

    return jsonify({
        "message": "Hello from Docker!",
        "visits": visits,
        "db": db_version
    })

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

### `app/requirements.txt`
```
flask
psycopg2-binary
redis
```

### `app/Dockerfile`
```dockerfile
FROM python:3.11-alpine

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```

### `.env`
```
DB_NAME=appdb
DB_USER=appuser
DB_PASSWORD=apppass
DB_ROOT_PASSWORD=rootpass
REDIS_HOST=redis
DB_HOST=db
```

---

## Task 2: depends_on & Healthchecks

### `docker-compose.yml`
```yaml
services:

  db:
    image: postgres:15-alpine
    restart: always
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - pg-data:/var/lib/postgresql/data
    networks:
      - app-net
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER} -d ${DB_NAME}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s

  redis:
    image: redis:7-alpine
    restart: always
    networks:
      - app-net
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3

  app:
    build: ./app
    ports:
      - "5000:5000"
    environment:
      DB_HOST: db
      DB_NAME: ${DB_NAME}
      DB_USER: ${DB_USER}
      DB_PASSWORD: ${DB_PASSWORD}
      REDIS_HOST: redis
    networks:
      - app-net
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    restart: on-failure

networks:
  app-net:
    driver: bridge

volumes:
  pg-data:
```

**What `condition: service_healthy` does:**
Without this, `depends_on` only waits for the container to *start* — not for the process inside to be ready. Postgres takes 5–10 seconds to actually accept connections after the container starts. `service_healthy` waits for the healthcheck to pass before starting the app. No more "connection refused" on startup.

```bash
docker compose up
```

```
[+] Running 3/3
 ✔ Container app-stack-redis-1  Healthy
 ✔ Container app-stack-db-1     Healthy
 ✔ Container app-stack-app-1    Started
```

---

## Task 3: Restart Policies

```yaml
restart: always       # always restart, even after docker daemon restart
restart: on-failure   # only restart if exit code is non-zero
restart: unless-stopped  # restart always unless manually stopped
restart: "no"         # never restart (default)
```

```bash
# Test restart: always on db
docker kill app-stack-db-1
# Container comes back automatically within seconds

docker ps
# app-stack-db-1   Up 3 seconds (just restarted)
```

**`restart: always` vs `restart: on-failure`:**

| Policy | Restarts when | Use case |
|--------|--------------|---------|
| `always` | Always — crash, manual kill, daemon restart | Databases, critical services |
| `on-failure` | Only if exits with error code | Apps where intentional exit (code 0) means "done" |
| `unless-stopped` | Always, except if you ran `docker stop` | Most production services |

---

## Task 4: Custom Dockerfiles in Compose

The `build:` key tells Compose to build from a Dockerfile instead of pulling an image:

```yaml
app:
  build:
    context: ./app          # build context (folder with Dockerfile)
    dockerfile: Dockerfile  # optional: specify filename if not "Dockerfile"
  ports:
    - "5000:5000"
```

```bash
# Make a code change in app/app.py, then:
docker compose up --build
# Rebuilds only the app image, pulls db and redis from cache
# Restarts only the app container
```

---

## Task 5: Named Networks & Volumes with Labels

```yaml
services:
  db:
    image: postgres:15-alpine
    labels:
      - "com.myapp.service=database"
      - "com.myapp.version=1.0"
    volumes:
      - pg-data:/var/lib/postgresql/data
    networks:
      - backend-net

  app:
    build: ./app
    labels:
      - "com.myapp.service=web"
    networks:
      - backend-net
      - frontend-net

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    networks:
      - frontend-net

networks:
  backend-net:
    driver: bridge
    labels:
      - "com.myapp.network=backend"
  frontend-net:
    driver: bridge
    labels:
      - "com.myapp.network=frontend"

volumes:
  pg-data:
    labels:
      - "com.myapp.data=postgres"
```

**Why explicit networks?** Separation of concerns. The database only exists on `backend-net` — Nginx can't reach it directly. The app sits on both networks, acting as the bridge. This mirrors real production architecture.

---

## Task 6: Scaling (Bonus)

```bash
docker compose up --scale app=3
```

```
Error: port is already allocated
```

**Why it breaks:** All 3 replicas try to bind to port `5000:5000` on the host. Only one process can own a host port at a time.

**Fix — remove host port mapping and put a load balancer in front:**
```yaml
app:
  build: ./app
  # Remove "ports:" — don't bind to host
  expose:
    - "5000"       # only expose inside Docker network
  networks:
    - app-net

nginx:
  image: nginx:alpine
  ports:
    - "80:80"      # only nginx is exposed to host
  networks:
    - app-net
```

Now `docker compose up --scale app=3` works. Nginx load-balances between the 3 app containers using service name `app` — Docker's DNS automatically round-robins between all replicas.

---

## Summary: New Commands Today

```bash
docker compose up --build           # rebuild images and start
docker compose up --scale app=3     # scale a service to N replicas
docker compose config               # validate and show resolved file
docker inspect <container> | grep Health  # check healthcheck status
```
