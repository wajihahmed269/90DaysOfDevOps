# Day 36 – Docker Project: Dockerize a Full Application

## The App: Flask Task Manager with Postgres

A simple REST API task manager — create, list, and delete tasks. Chose this because it covers the full stack: Python web app, relational database, proper multi-stage build, and real environment config.

---

## Project Structure

```
flask-taskmanager/
├── app/
│   ├── app.py
│   ├── requirements.txt
│   └── Dockerfile
├── .env
├── .dockerignore
├── docker-compose.yml
└── README.md
```

---

## `app/app.py`

```python
from flask import Flask, jsonify, request
import psycopg2
import os

app = Flask(__name__)

def get_db():
    return psycopg2.connect(
        host=os.getenv("DB_HOST"),
        database=os.getenv("DB_NAME"),
        user=os.getenv("DB_USER"),
        password=os.getenv("DB_PASSWORD")
    )

def init_db():
    conn = get_db()
    cur = conn.cursor()
    cur.execute("""
        CREATE TABLE IF NOT EXISTS tasks (
            id SERIAL PRIMARY KEY,
            title VARCHAR(200) NOT NULL,
            done BOOLEAN DEFAULT FALSE
        )
    """)
    conn.commit()
    conn.close()

@app.route("/tasks", methods=["GET"])
def get_tasks():
    conn = get_db()
    cur = conn.cursor()
    cur.execute("SELECT id, title, done FROM tasks ORDER BY id")
    tasks = [{"id": r[0], "title": r[1], "done": r[2]} for r in cur.fetchall()]
    conn.close()
    return jsonify(tasks)

@app.route("/tasks", methods=["POST"])
def create_task():
    data = request.get_json()
    conn = get_db()
    cur = conn.cursor()
    cur.execute("INSERT INTO tasks (title) VALUES (%s) RETURNING id", (data["title"],))
    task_id = cur.fetchone()[0]
    conn.commit()
    conn.close()
    return jsonify({"id": task_id, "title": data["title"], "done": False}), 201

@app.route("/tasks/<int:task_id>", methods=["DELETE"])
def delete_task(task_id):
    conn = get_db()
    cur = conn.cursor()
    cur.execute("DELETE FROM tasks WHERE id = %s", (task_id,))
    conn.commit()
    conn.close()
    return jsonify({"deleted": task_id})

if __name__ == "__main__":
    init_db()
    app.run(host="0.0.0.0", port=5000)
```

---

## `app/requirements.txt`

```
flask==3.0.0
psycopg2-binary==2.9.9
```

---

## `app/Dockerfile`

```dockerfile
# ── Stage 1: Dependencies ────────────────────────────────────
FROM python:3.11-alpine AS builder

WORKDIR /app

# Install build deps for psycopg2
RUN apk add --no-cache gcc musl-dev postgresql-dev

COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

# ── Stage 2: Runtime ─────────────────────────────────────────
FROM python:3.11-alpine

# Runtime postgres client lib only
RUN apk add --no-cache libpq

# Non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

# Copy installed packages from builder
COPY --from=builder /install /usr/local

# Copy app code
COPY app.py .

USER appuser

EXPOSE 5000

CMD ["python", "app.py"]
```

---

## `.env`

```
DB_HOST=db
DB_NAME=taskdb
DB_USER=taskuser
DB_PASSWORD=taskpass
POSTGRES_PASSWORD=taskpass
POSTGRES_DB=taskdb
POSTGRES_USER=taskuser
```

---

## `.dockerignore`

```
.git
.env
__pycache__
*.pyc
*.md
.DS_Store
```

---

## `docker-compose.yml`

```yaml
services:

  db:
    image: postgres:15-alpine
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - task-pgdata:/var/lib/postgresql/data
    networks:
      - app-net
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 20s

  app:
    build:
      context: ./app
      dockerfile: Dockerfile
    image: wajihahmed269/flask-taskmanager:v1
    ports:
      - "5000:5000"
    environment:
      DB_HOST: ${DB_HOST}
      DB_NAME: ${DB_NAME}
      DB_USER: ${DB_USER}
      DB_PASSWORD: ${DB_PASSWORD}
    networks:
      - app-net
    depends_on:
      db:
        condition: service_healthy
    restart: on-failure

networks:
  app-net:
    driver: bridge

volumes:
  task-pgdata:
```

---

## `README.md`

```markdown
# Flask Task Manager

Simple REST API for task management — Flask + Postgres, fully Dockerized.

## Run

```bash
git clone https://github.com/wajihahmed269/flask-taskmanager
cd flask-taskmanager
cp .env.example .env
docker compose up -d
```

API available at `http://localhost:5000`

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | /tasks | List all tasks |
| POST | /tasks | Create task `{"title": "..."}` |
| DELETE | /tasks/:id | Delete a task |

## Environment Variables

| Variable | Description |
|----------|-------------|
| DB_HOST | Database hostname (default: db) |
| DB_NAME | Database name |
| DB_USER | Database user |
| DB_PASSWORD | Database password |
```

---

## Task 4: Push to Docker Hub

```bash
docker compose up --build
docker push wajihahmed269/flask-taskmanager:v1
```

**Docker Hub:** `https://hub.docker.com/r/wajihahmed269/flask-taskmanager`

---

## Task 5: Full Flow Test

```bash
# Remove everything
docker compose down -v
docker rmi wajihahmed269/flask-taskmanager:v1

# Pull from Hub and run
docker compose up -d
# → Pulls app from Hub, pulls postgres from Hub, starts both
# → App waits for postgres healthcheck to pass
# → App initializes table, ready to receive requests

curl http://localhost:5000/tasks
# []

curl -X POST http://localhost:5000/tasks -H "Content-Type: application/json" -d '{"title":"Learn Docker"}'
# {"done":false,"id":1,"title":"Learn Docker"}

curl http://localhost:5000/tasks
# [{"done":false,"id":1,"title":"Learn Docker"}]
```

Works clean from Docker Hub. Zero local setup needed beyond Docker and the compose file.

---

## Challenges & Solutions

| Challenge | Solution |
|-----------|---------|
| `psycopg2` needs `gcc` to compile but it shouldn't be in final image | Multi-stage build — compile in builder stage, copy only the `.so` files |
| App starts before DB is ready, connection refused | `healthcheck` + `depends_on: condition: service_healthy` |
| `.env` accidentally added to git | Added to `.dockerignore` AND `.gitignore` |

## Final Image Size

```
wajihahmed269/flask-taskmanager:v1   38.4MB
```

Single-stage equivalent would be ~180MB. Multi-stage saved ~140MB.
