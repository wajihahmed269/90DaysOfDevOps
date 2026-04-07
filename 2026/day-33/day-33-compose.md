# Day 33 – Docker Compose: Multi-Container Basics

## Task 1: Install & Verify

```bash
# Docker Compose v2 is bundled with Docker Desktop and recent Docker Engine installs
docker compose version
# Docker Compose version v2.24.1
```

---

## Task 2: Your First Compose File

### Directory Structure
```
compose-basics/
└── docker-compose.yml
```

### `docker-compose.yml`
```yaml
services:
  web:
    image: nginx
    ports:
      - "8080:80"
```

```bash
# Start
docker compose up

# Access http://localhost:8080 → Nginx welcome page

# Stop and remove containers + network
docker compose down
```

**What Compose did automatically:**
- Created a network called `compose-basics_default`
- Pulled the nginx image
- Started the container
- Mapped port 8080 → 80
- `docker compose down` removed the container and network cleanly

---

## Task 3: Two-Container Setup — WordPress + MySQL

### `docker-compose.yml`
```yaml
services:
  db:
    image: mysql:8.0
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppass
    volumes:
      - mysql-data:/var/lib/mysql

  wordpress:
    image: wordpress:latest
    restart: always
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db        # service name = DNS name
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppass
    depends_on:
      - db

volumes:
  mysql-data:
```

```bash
docker compose up -d
# Access http://localhost:8080 → WordPress setup page

# Complete WordPress setup, create a post

# Stop everything
docker compose down

# Start again — data is still there because of mysql-data volume
docker compose up -d
```

**Result:** WordPress data survived because MySQL's data is in a named volume. The container is gone, the data is not.

**Key point:** `WORDPRESS_DB_HOST: db` — the value `db` is the service name from compose. Compose's default network automatically registers it as a DNS name. No IPs needed.

---

## Task 4: Compose Commands

```bash
# Start in detached mode
docker compose up -d

# View running services
docker compose ps

# View logs of ALL services
docker compose logs

# View logs of a specific service, follow mode
docker compose logs -f wordpress

# Stop containers (without removing them)
docker compose stop

# Start stopped containers
docker compose start

# Remove containers and network (keeps volumes)
docker compose down

# Remove containers, network AND volumes
docker compose down -v

# Rebuild images if Dockerfile changed
docker compose up --build

# Pull latest images
docker compose pull
```

**Output of `docker compose ps`:**
```
NAME                    IMAGE               STATUS          PORTS
compose-basics-db-1     mysql:8.0           Up 2 minutes    3306/tcp
compose-basics-wp-1     wordpress:latest    Up 2 minutes    0.0.0.0:8080->80/tcp
```

---

## Task 5: Environment Variables

### Method 1 — Direct in compose file
```yaml
services:
  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: myapp
```

### Method 2 — `.env` file

**.env**
```
MYSQL_ROOT_PASSWORD=rootpass
MYSQL_DATABASE=myapp
MYSQL_USER=appuser
MYSQL_PASSWORD=apppass
WP_PORT=8080
```

**docker-compose.yml**
```yaml
services:
  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}

  wordpress:
    image: wordpress:latest
    ports:
      - "${WP_PORT}:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_NAME: ${MYSQL_DATABASE}
      WORDPRESS_DB_USER: ${MYSQL_USER}
      WORDPRESS_DB_PASSWORD: ${MYSQL_PASSWORD}
```

```bash
# Verify variables are picked up
docker compose config
# Shows the fully resolved compose file with all variables substituted
```

> Always add `.env` to your `.gitignore` — never commit passwords to version control.

---

## Summary: Commands Used Today

```bash
docker compose version              # check version
docker compose up                   # start services (attached)
docker compose up -d                # start services (detached)
docker compose down                 # stop and remove containers + network
docker compose down -v              # also remove volumes
docker compose ps                   # list running services
docker compose logs                 # all service logs
docker compose logs -f <service>    # follow specific service logs
docker compose stop                 # stop without removing
docker compose start                # start stopped services
docker compose pull                 # pull latest images
docker compose up --build           # rebuild images then start
docker compose config               # show resolved compose file
```
