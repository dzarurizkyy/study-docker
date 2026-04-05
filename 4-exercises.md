# 🐳 Docker Hands-On Practice — ShopFast Deployment

A complete hands-on scenario that covers all core Docker and Docker Compose concepts: image management, container lifecycle, port forwarding, environment variables, resource limits, bind mounts, volumes, backup & restore, networking, Dockerfile, multi-stage builds, and Docker Compose — all in one real-world e-commerce deployment use case.

---

## 📖 Scenario

You are a DevOps engineer deploying **ShopFast** — an e-commerce platform — using Docker. You will containerize and run:

- **nginx** — web server serving the storefront
- **MySQL** — relational database for orders and products
- **Redis** — in-memory cache for sessions and rate limiting
- **Adminer** — lightweight database GUI for management
- **Custom App Image** — a Node.js API backend built from a Dockerfile

By the end of this practice, you will have hands-on experience with every concept covered in the Docker guides.

---

## 🗂️ Table of Contents

1. [Setup & Verify](#1-setup--verify)
2. [Image Management](#2-image-management)
3. [Container Lifecycle](#3-container-lifecycle)
4. [Container Operations — Logs & Exec](#4-container-operations--logs--exec)
5. [Port Forwarding](#5-port-forwarding)
6. [Environment Variables](#6-environment-variables)
7. [Resource Management](#7-resource-management)
8. [Bind Mounts](#8-bind-mounts)
9. [Volume Management](#9-volume-management)
10. [Backup & Restore](#10-backup--restore)
11. [Network Management](#11-network-management)
12. [Dockerfile — Build Custom Image](#12-dockerfile--build-custom-image)
13. [Multi-Stage Build](#13-multi-stage-build)
14. [Docker Hub Registry](#14-docker-hub-registry)
15. [Docker Compose](#15-docker-compose)
16. [Utilities & Cleanup](#16-utilities--cleanup)
17. [Challenge Tasks](#-challenge-tasks)

---

## 1. Setup & Verify

### 1a. Check Docker Version

```bash
docker version
```

Expected output includes:
```
Client: Docker Engine - Community
 Version: 26.x.x
...
Server: Docker Engine - Community
 Engine:
  Version: 26.x.x
```

### 1b. Verify Docker is Running

```bash
docker container ls
# Should return an empty table (no containers yet)
```

---

## 2. Image Management

### 2a. Pull Required Images

Download all images ShopFast will need.

```bash
docker image pull nginx:alpine
docker image pull mysql:8.0
docker image pull redis:7-alpine
docker image pull adminer:latest
docker image pull node:18-alpine
docker image pull ubuntu:22.04
```

### 2b. List Downloaded Images

```bash
docker image ls
# Should show all 6 images with their sizes and tags
```

### 2c. Pull a Specific Version

```bash
docker image pull nginx:1.25
```

> 💡 Always pin specific versions in production. `nginx:latest` is not guaranteed to be the same image across machines.

### 2d. Remove an Image

```bash
docker image rm nginx:1.25
# (integer) 1 — image removed
```

### 2e. Verify It Was Removed

```bash
docker image ls
# nginx:1.25 should no longer appear
```

---

## 3. Container Lifecycle

### 3a. Create a Container (Without Starting It)

```bash
docker container create --name shopfast-web nginx:alpine
```

### 3b. Verify It Was Created

```bash
docker container ls -a
# Should show shopfast-web with status "Created"
```

### 3c. Start the Container

```bash
docker container start shopfast-web
```

### 3d. Verify It Is Running

```bash
docker container ls
# Should show shopfast-web with status "Up X seconds"
```

### 3e. Stop the Container

```bash
docker container stop shopfast-web
```

### 3f. Remove the Container

```bash
docker container rm shopfast-web
```

> ⚠️ You cannot remove a running container. Always stop it first, or use `docker container rm -f` to force remove.

---

## 4. Container Operations — Logs & Exec

### 4a. Create and Start nginx With a Name

```bash
docker container create --name shopfast-web --publish 8080:80 nginx:alpine
docker container start shopfast-web
```

### 4b. View Container Logs

```bash
docker container logs shopfast-web
```

### 4c. Follow Logs in Real Time

```bash
docker container logs -f shopfast-web
# Open http://localhost:8080 in a browser while watching
# Press Ctrl+C to stop following
```

### 4d. Execute a Command Inside the Container

```bash
docker container exec -i -t shopfast-web /bin/sh
```

Once inside the container:

```bash
# Check nginx version
nginx -v

# View the default HTML file
cat /usr/share/nginx/html/index.html

# Exit the container shell
exit
```

### 4e. View Resource Usage

```bash
docker container stats shopfast-web
# Press Ctrl+C to stop
```

---

## 5. Port Forwarding

### 5a. Remove the Previous Web Container

```bash
docker container stop shopfast-web
docker container rm shopfast-web
```

### 5b. Create Container With Port Mapping

```bash
docker container create \
  --name shopfast-web \
  --publish 8080:80 \
  nginx:alpine
```

### 5c. Start and Verify

```bash
docker container start shopfast-web
```

Open your browser at `http://localhost:8080` — you should see the nginx welcome page.

### 5d. Map Multiple Ports

```bash
docker container stop shopfast-web
docker container rm shopfast-web

docker container create \
  --name shopfast-web \
  --publish 8080:80 \
  --publish 8443:443 \
  nginx:alpine

docker container start shopfast-web
```

> 📌 Format: `--publish HOST_PORT:CONTAINER_PORT`
> The left side is your machine, the right side is inside the container.

---

## 6. Environment Variables

### 6a. Create MySQL Container With Environment Variables

```bash
docker container create \
  --name shopfast-db \
  --publish 3306:3306 \
  --env MYSQL_ROOT_PASSWORD=shopfast_root \
  --env MYSQL_DATABASE=shopfast \
  --env MYSQL_USER=shopfast_user \
  --env MYSQL_PASSWORD=shopfast_pass \
  mysql:8.0
```

### 6b. Start and Verify

```bash
docker container start shopfast-db

docker container logs -f shopfast-db
# Wait until you see: "ready for connections"
# Press Ctrl+C to stop following
```

### 6c. Create Redis Container With Auth

```bash
docker container create \
  --name shopfast-cache \
  --publish 6379:6379 \
  --env REDIS_PASSWORD=cache_secret \
  redis:7-alpine

docker container start shopfast-cache
```

---

## 7. Resource Management

### 7a. Create a Resource-Limited nginx Container

```bash
docker container stop shopfast-web
docker container rm shopfast-web

docker container create \
  --name shopfast-web \
  --publish 8080:80 \
  --memory="256m" \
  --cpus="0.5" \
  nginx:alpine

docker container start shopfast-web
```

### 7b. Create Resource-Limited MySQL

```bash
docker container stop shopfast-db
docker container rm shopfast-db

docker container create \
  --name shopfast-db \
  --publish 3306:3306 \
  --memory="1g" \
  --cpus="1.0" \
  --env MYSQL_ROOT_PASSWORD=shopfast_root \
  --env MYSQL_DATABASE=shopfast \
  --env MYSQL_USER=shopfast_user \
  --env MYSQL_PASSWORD=shopfast_pass \
  mysql:8.0

docker container start shopfast-db
```

### 7c. Monitor Resource Usage

```bash
docker container stats
# Shows CPU%, MEM USAGE/LIMIT, NET I/O for all running containers
# Press Ctrl+C to stop
```

---

## 8. Bind Mounts

### 8a. Create a Custom HTML Page

```bash
mkdir -p ~/shopfast/html

cat > ~/shopfast/html/index.html << 'EOF'
<!DOCTYPE html>
<html>
<head><title>ShopFast</title></head>
<body>
  <h1>Welcome to ShopFast!</h1>
  <p>Your one-stop e-commerce platform.</p>
</body>
</html>
EOF
```

### 8b. Mount the Folder Into nginx (Read-Only)

```bash
docker container stop shopfast-web
docker container rm shopfast-web

docker container create \
  --name shopfast-web \
  --publish 8080:80 \
  --memory="256m" \
  --cpus="0.5" \
  --mount "type=bind,source=$(pwd)/shopfast/html,destination=/usr/share/nginx/html,readonly" \
  nginx:alpine

docker container start shopfast-web
```

### 8c. Verify — Open Browser

Visit `http://localhost:8080` — you should see **"Welcome to ShopFast!"**.

### 8d. Edit the File on Host — See Changes Instantly

```bash
echo "<p>Flash Sale: 50% off everything today!</p>" >> ~/shopfast/html/index.html
```

Refresh the browser — the change appears immediately without restarting the container.

---

## 9. Volume Management

### 9a. Create a Volume for MySQL Data

```bash
docker volume create shopfast-db-data
```

### 9b. List Volumes

```bash
docker volume ls
# Should show shopfast-db-data
```

### 9c. Attach Volume to MySQL Container

```bash
docker container stop shopfast-db
docker container rm shopfast-db

docker container create \
  --name shopfast-db \
  --publish 3306:3306 \
  --memory="1g" \
  --cpus="1.0" \
  --env MYSQL_ROOT_PASSWORD=shopfast_root \
  --env MYSQL_DATABASE=shopfast \
  --env MYSQL_USER=shopfast_user \
  --env MYSQL_PASSWORD=shopfast_pass \
  --mount "type=volume,source=shopfast-db-data,destination=/var/lib/mysql" \
  mysql:8.0

docker container start shopfast-db
```

### 9d. Create a Volume for Redis Data

```bash
docker volume create shopfast-cache-data

docker container stop shopfast-cache
docker container rm shopfast-cache

docker container create \
  --name shopfast-cache \
  --publish 6379:6379 \
  --mount "type=volume,source=shopfast-cache-data,destination=/data" \
  redis:7-alpine

docker container start shopfast-cache
```

### 9e. Inspect a Volume

```bash
docker inspect shopfast-db-data
# Shows volume details including mount path on host
```

### 9f. Test Data Persistence

```bash
# Add some test data to MySQL
docker container exec -i -t shopfast-db mysql -u root -pshopfast_root shopfast -e \
  "CREATE TABLE products (id INT PRIMARY KEY, name VARCHAR(100)); \
   INSERT INTO products VALUES (1, 'Wireless Headphones');"

# Stop and remove the container (but NOT the volume)
docker container stop shopfast-db
docker container rm shopfast-db

# Recreate the container with the same volume
docker container create \
  --name shopfast-db \
  --publish 3306:3306 \
  --env MYSQL_ROOT_PASSWORD=shopfast_root \
  --env MYSQL_DATABASE=shopfast \
  --env MYSQL_USER=shopfast_user \
  --env MYSQL_PASSWORD=shopfast_pass \
  --mount "type=volume,source=shopfast-db-data,destination=/var/lib/mysql" \
  mysql:8.0

docker container start shopfast-db

# Wait ~20 seconds for MySQL to start, then verify the data survived
docker container exec -i -t shopfast-db mysql -u root -pshopfast_root shopfast -e \
  "SELECT * FROM products;"
```

Expected output:
```
+----+----------------------+
| id | name                 |
+----+----------------------+
|  1 | Wireless Headphones  |
+----+----------------------+
```

---

## 10. Backup & Restore

### 10a. Create Backup Directory

```bash
mkdir -p ~/shopfast/backup
```

### 10b. Backup the MySQL Volume

```bash
docker container stop shopfast-db

docker container run --rm \
  --name backup-shopfast-db \
  --mount "type=bind,source=$(pwd)/shopfast/backup,destination=/backup" \
  --mount "type=volume,source=shopfast-db-data,destination=/data" \
  ubuntu \
  tar cvf /backup/shopfast-db-backup.tar.gz /data

docker container start shopfast-db
```

### 10c. Verify the Backup File

```bash
ls -lh ~/shopfast/backup/
# Should show shopfast-db-backup.tar.gz
```

### 10d. Restore to a New Volume

```bash
# Create a new volume for restore testing
docker volume create shopfast-db-restored

# Restore backup into the new volume
docker container run --rm \
  --name restore-shopfast-db \
  --mount "type=bind,source=$(pwd)/shopfast/backup,destination=/backup" \
  --mount "type=volume,source=shopfast-db-restored,destination=/data" \
  ubuntu \
  bash -c "tar xvf /backup/shopfast-db-backup.tar.gz"
```

### 10e. Verify the Restored Volume

```bash
docker inspect shopfast-db-restored
# Confirm the volume exists and has the correct mount path
```

---

## 11. Network Management

### 11a. List Default Networks

```bash
docker network ls
# Should show bridge, host, none
```

### 11b. Create a Custom Network for ShopFast

```bash
docker network create --driver bridge shopfast-network
```

### 11c. Verify

```bash
docker network ls
# shopfast-network should now appear
```

### 11d. Recreate Containers on the Custom Network

Stop all existing containers and recreate them on the same network so they can communicate with each other using their container names as hostnames.

```bash
# Stop and remove all current containers
docker container stop shopfast-web shopfast-db shopfast-cache
docker container rm shopfast-web shopfast-db shopfast-cache

# nginx
docker container create \
  --name shopfast-web \
  --network shopfast-network \
  --publish 8080:80 \
  --memory="256m" \
  --cpus="0.5" \
  --mount "type=bind,source=$(pwd)/shopfast/html,destination=/usr/share/nginx/html,readonly" \
  nginx:alpine

# MySQL
docker container create \
  --name shopfast-db \
  --network shopfast-network \
  --publish 3306:3306 \
  --memory="1g" \
  --cpus="1.0" \
  --env MYSQL_ROOT_PASSWORD=shopfast_root \
  --env MYSQL_DATABASE=shopfast \
  --env MYSQL_USER=shopfast_user \
  --env MYSQL_PASSWORD=shopfast_pass \
  --mount "type=volume,source=shopfast-db-data,destination=/var/lib/mysql" \
  mysql:8.0

# Redis
docker container create \
  --name shopfast-cache \
  --network shopfast-network \
  --publish 6379:6379 \
  --mount "type=volume,source=shopfast-cache-data,destination=/data" \
  redis:7-alpine

# Adminer
docker container create \
  --name shopfast-adminer \
  --network shopfast-network \
  --publish 8081:8080 \
  adminer:latest

# Start all
docker container start shopfast-db shopfast-cache shopfast-web shopfast-adminer
```

### 11e. Test Container-to-Container Communication

```bash
# Ping the database from the web container (using container name as hostname)
docker container exec -i -t shopfast-web ping -c 3 shopfast-db
# Packets should be transmitted successfully
```

### 11f. Connect an Existing Container to a Network

```bash
docker network connect shopfast-network shopfast-adminer
# (already connected, but this shows how to add a running container to a network)
```

### 11g. Disconnect a Container From a Network

```bash
docker network disconnect shopfast-network shopfast-adminer
docker network connect shopfast-network shopfast-adminer
# Disconnect and reconnect to demonstrate the command
```

### 11h. Inspect a Network

```bash
docker inspect shopfast-network
# Shows all containers connected to the network
```

---

## 12. Dockerfile — Build Custom Image

### 12a. Create the Project Structure

```bash
mkdir -p ~/shopfast/api
cd ~/shopfast/api
```

### 12b. Create the Application File

```bash
cat > app.js << 'EOF'
const http = require('http');

const PORT = process.env.PORT || 3000;
const APP_ENV = process.env.APP_ENV || 'development';

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({
    service: 'ShopFast API',
    version: '1.0.0',
    environment: APP_ENV,
    status: 'running',
    timestamp: new Date().toISOString()
  }));
});

server.listen(PORT, () => {
  console.log(`ShopFast API running on port ${PORT} [${APP_ENV}]`);
});
EOF
```

### 12c. Create package.json

```bash
cat > package.json << 'EOF'
{
  "name": "shopfast-api",
  "version": "1.0.0",
  "description": "ShopFast API Service",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  }
}
EOF
```

### 12d. Create .dockerignore

```bash
cat > .dockerignore << 'EOF'
node_modules
npm-debug.log
.env
.git
.gitignore
README.md
EOF
```

### 12e. Create the Dockerfile

```dockerfile
# Base image
FROM node:18-alpine

# Add metadata
LABEL maintainer="devops@shopfast.com"
LABEL version="1.0.0"
LABEL description="ShopFast API Service"

# Set working directory
WORKDIR /app

# Set environment variables
ENV PORT=3000
ENV APP_ENV=production

# Create non-root group and user
RUN addgroup -S shopfast && \
    adduser -S -D -h /app shopfastuser shopfast

# Copy dependency files first (for better layer caching)
COPY package*.json ./

# Install dependencies
RUN npm install --omit=dev

# Copy application code
COPY app.js ./

# Set ownership
RUN chown -R shopfastuser:shopfast /app

# Switch to non-root user
USER shopfastuser

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000', r => r.statusCode === 200 ? process.exit(0) : process.exit(1))"

# Start command
CMD ["npm", "start"]
```

### 12f. Build the Image

```bash
docker build -t shopfast/api:1.0 .
```

### 12g. Build With Detailed Output

```bash
docker build -t shopfast/api:1.0 . --progress=plain
# Shows each build step in detail
```

### 12h. Build Without Cache

```bash
docker build -t shopfast/api:1.0 . --no-cache
# Forces a fresh build — useful when dependencies change
```

### 12i. Run the Custom Image

```bash
docker container create \
  --name shopfast-api \
  --network shopfast-network \
  --publish 3000:3000 \
  --env APP_ENV=production \
  --memory="256m" \
  --cpus="0.5" \
  shopfast/api:1.0

docker container start shopfast-api
```

### 12j. Verify the API Is Running

```bash
curl http://localhost:3000
# {"service":"ShopFast API","version":"1.0.0","environment":"production","status":"running",...}
```

---

## 13. Multi-Stage Build

Multi-stage builds produce smaller, cleaner images by separating the build environment from the runtime environment.

### 13a. Create a Multi-Stage Dockerfile

```bash
cat > ~/shopfast/api/Dockerfile.multistage << 'EOF'
# ========================================
# Stage 1: Builder
# Install ALL dependencies including devDependencies
# ========================================
FROM node:18-alpine AS builder

WORKDIR /build

COPY package*.json ./
RUN npm install

COPY app.js ./

# Run any build steps here (e.g., TypeScript compile, bundling)
RUN echo "Build complete"

# ========================================
# Stage 2: Runtime
# Copy only what's needed to run the app
# ========================================
FROM node:18-alpine AS runtime

LABEL maintainer="devops@shopfast.com"

WORKDIR /app

ENV PORT=3000
ENV APP_ENV=production

# Create non-root user
RUN addgroup -S shopfast && \
    adduser -S -D -h /app shopfastuser shopfast

# Copy only production artifacts from builder stage
COPY --from=builder /build/package*.json ./
COPY --from=builder /build/node_modules ./node_modules
COPY --from=builder /build/app.js ./

RUN chown -R shopfastuser:shopfast /app

USER shopfastuser

EXPOSE 3000

CMD ["npm", "start"]
EOF
```

### 13b. Build the Multi-Stage Image

```bash
docker build -f ~/shopfast/api/Dockerfile.multistage -t shopfast/api:1.0-slim ~/shopfast/api/
```

### 13c. Compare Image Sizes

```bash
docker image ls shopfast/api
# Compare the size of shopfast/api:1.0 vs shopfast/api:1.0-slim
```

> 💡 In real projects with many devDependencies (test frameworks, type checkers, build tools), multi-stage builds can reduce image size by 50–80%.

---

## 14. Docker Hub Registry

### 14a. Log In to Docker Hub

```bash
docker login -u your-dockerhub-username
# Enter your Personal Access Token when prompted (not your password)
```

### 14b. Tag the Image for Docker Hub

```bash
docker tag shopfast/api:1.0 your-dockerhub-username/shopfast-api:1.0
docker tag shopfast/api:1.0 your-dockerhub-username/shopfast-api:latest
```

### 14c. Push to Docker Hub

```bash
docker push your-dockerhub-username/shopfast-api:1.0
docker push your-dockerhub-username/shopfast-api:latest
```

### 14d. Pull the Image on Another Machine

```bash
docker pull your-dockerhub-username/shopfast-api:1.0
```

### 14e. Verify the Image Is Public

Visit `https://hub.docker.com/r/your-dockerhub-username/shopfast-api` in a browser.

---

## 15. Docker Compose

Now let's replace all manual `docker container create` commands with a single `docker-compose.yml`.

### 15a. Stop and Remove All Existing Containers

```bash
docker container stop shopfast-web shopfast-db shopfast-cache shopfast-api shopfast-adminer
docker container rm shopfast-web shopfast-db shopfast-cache shopfast-api shopfast-adminer
docker network rm shopfast-network
```

### 15b. Create Project Folder Structure

```bash
mkdir -p ~/shopfast/compose
cd ~/shopfast/compose
mkdir -p html
```

```bash
cat > html/index.html << 'EOF'
<!DOCTYPE html>
<html>
<head><title>ShopFast</title></head>
<body>
  <h1>Welcome to ShopFast!</h1>
  <p>Your one-stop e-commerce platform.</p>
</body>
</html>
EOF
```

### 15c. Create the .env File

```bash
cat > .env << 'EOF'
MYSQL_ROOT_PASSWORD=shopfast_root
MYSQL_DATABASE=shopfast
MYSQL_USER=shopfast_user
MYSQL_PASSWORD=shopfast_pass
APP_ENV=production
EOF
```

> ⚠️ Always add `.env` to `.gitignore`. Never commit secrets to version control.

### 15d. Create docker-compose.yml

```yaml
services:

  # ────────────────────────────────
  # Web Server
  # ────────────────────────────────
  webserver:
    container_name: shopfast-web
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - type: bind
        source: ./html
        target: /usr/share/nginx/html
        read_only: true
    networks:
      - shopfast-network
    depends_on:
      - api
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 3s
      start_period: 5s
      retries: 3
    deploy:
      resources:
        limits:
          cpus: "0.50"
          memory: "256M"

  # ────────────────────────────────
  # API Backend
  # ────────────────────────────────
  api:
    container_name: shopfast-api
    build:
      context: ../api
      dockerfile: Dockerfile
    image: shopfast/api:1.0
    ports:
      - "3000:3000"
    environment:
      APP_ENV: ${APP_ENV}
      PORT: 3000
    networks:
      - shopfast-network
    depends_on:
      database:
        condition: service_healthy
    restart: unless-stopped
    deploy:
      resources:
        limits:
          cpus: "0.50"
          memory: "256M"

  # ────────────────────────────────
  # Database
  # ────────────────────────────────
  database:
    container_name: shopfast-db
    image: mysql:8.0
    ports:
      - "3306:3306"
    env_file:
      - .env
    volumes:
      - type: volume
        source: shopfast-db-data
        target: /var/lib/mysql
    networks:
      - shopfast-network
    restart: always
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-pshopfast_root"]
      interval: 10s
      timeout: 5s
      start_period: 30s
      retries: 5
    deploy:
      resources:
        limits:
          cpus: "1.00"
          memory: "1G"

  # ────────────────────────────────
  # Cache
  # ────────────────────────────────
  cache:
    container_name: shopfast-cache
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - type: volume
        source: shopfast-cache-data
        target: /data
    networks:
      - shopfast-network
    restart: unless-stopped
    deploy:
      resources:
        limits:
          cpus: "0.25"
          memory: "256M"

  # ────────────────────────────────
  # Database GUI
  # ────────────────────────────────
  adminer:
    container_name: shopfast-adminer
    image: adminer:latest
    ports:
      - "8081:8080"
    networks:
      - shopfast-network
    depends_on:
      - database
    restart: unless-stopped

# ────────────────────────────────
# Volumes
# ────────────────────────────────
volumes:
  shopfast-db-data:
    name: shopfast-db-data
  shopfast-cache-data:
    name: shopfast-cache-data

# ────────────────────────────────
# Networks
# ────────────────────────────────
networks:
  shopfast-network:
    name: shopfast-network
    driver: bridge
```

### 15e. Start the Full Stack

```bash
docker compose up -d
```

### 15f. Check Running Containers

```bash
docker compose ps
# Should show all 5 services as "running"
```

### 15g. Follow Logs

```bash
docker compose logs -f
# Follow logs from all services at once
# Press Ctrl+C to stop

docker compose logs -f database
# Follow only database logs
```

### 15h. Access the Services

| Service | URL |
|---|---|
| Nginx Web | http://localhost:8080 |
| ShopFast API | http://localhost:3000 |
| Adminer (DB GUI) | http://localhost:8081 |

In Adminer, login with:
- **System:** MySQL
- **Server:** `database`
- **Username:** `root`
- **Password:** `shopfast_root`
- **Database:** ``

### 15i. Stop the Stack

```bash
docker compose stop
```

### 15j. Start Again

```bash
docker compose start
```

### 15k. Down — Remove Containers and Networks

```bash
docker compose down
# Removes containers and network, but keeps volumes
```

### 15l. Down With Volumes

```bash
docker compose down -v
# Also removes volumes — all persistent data is lost!
```

> ⚠️ Only use `down -v` if you intentionally want to wipe all data.

### 15m. Multiple Compose Files — Dev vs Production

Create a production override file:

```bash
cat > docker-compose.prod.yml << 'EOF'
services:
  webserver:
    restart: always
    deploy:
      resources:
        limits:
          cpus: "1.00"
          memory: "512M"

  api:
    restart: always
    environment:
      APP_ENV: production

  database:
    restart: always
EOF
```

Start with production config:

```bash
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

### 15n. Monitor Docker Events

```bash
docker events --filter 'container=shopfast-db'
# In another terminal, restart the db to see events:
# docker container restart shopfast-db
```

---

## 16. Utilities & Cleanup

### 16a. Inspect Any Docker Object

```bash
docker inspect shopfast-web          # Container info
docker inspect shopfast/api:1.0      # Image info
docker inspect shopfast-db-data      # Volume info
docker inspect shopfast-network      # Network info
```

### 16b. Prune Stopped Containers

```bash
docker container prune
# Removes all stopped containers
```

### 16c. Prune Unused Images

```bash
docker image prune
# Removes dangling images (no tag, not used by any container)

docker image prune -a
# Removes all unused images (including tagged ones not used by containers)
```

### 16d. Prune Unused Volumes

```bash
docker volume prune
# Removes volumes not attached to any container
```

### 16e. Prune Unused Networks

```bash
docker network prune
```

### 16f. Full System Cleanup

```bash
docker system prune
# Removes stopped containers, unused networks, dangling images

docker system prune -a --volumes
# Nuclear option — removes everything not currently in use
```

> ⚠️ Never run `docker system prune -a --volumes` on a production server without stopping services first.

---

## 🏆 Challenge Tasks

Once you have completed the practice above, try these on your own:

---

### Challenge 1 — Image & Container Basics

Pull `httpd:alpine` (Apache HTTP Server). Create a container named `challenge-apache` that maps host port `9090` to container port `80`. Start it, verify it's running in your browser, then stop and remove it.

---

### Challenge 2 — Bind Mount + Live Reload

Create a folder `~/challenge/html` with an `index.html` that displays your name and a favorite quote. Mount it into an nginx container on port `9091` as read-only. Edit the file on your host machine and verify the change reflects in the browser without restarting the container.

---

### Challenge 3 — Volume Persistence

Create a named volume `challenge-data`. Run an `ubuntu` container that mounts it at `/data` and writes a file:

```bash
echo "ShopFast was here" > /data/message.txt
```

Remove the container. Create a new `ubuntu` container with the same volume and verify the file still exists.

---

### Challenge 4 — Backup & Restore

Back up the `challenge-data` volume from Challenge 3 to `~/challenge/backup/` as `challenge-backup.tar.gz`. Create a new volume `challenge-data-restored`, restore the backup into it, and verify the file `message.txt` is present.

---

### Challenge 5 — Custom Network + Communication

Create a custom network `challenge-net`. Run two containers on the network:
- `challenge-redis` using `redis:7-alpine`
- `challenge-ubuntu` using `ubuntu:22.04`

From inside `challenge-ubuntu`, ping `challenge-redis` by name. Then disconnect `challenge-ubuntu` from the network and verify the ping no longer works.

---

### Challenge 6 — Dockerfile

Write a Dockerfile for a simple Python HTTP server with the following requirements:
- Base image: `python:3.11-alpine`
- A non-root user named `appuser` in group `appgroup`
- A file `index.html` with custom content served from `/app`
- `EXPOSE 8000`
- CMD that runs `python -m http.server 8000`
- A `HEALTHCHECK` using `wget`

Build it, run it on port `9092`, and verify via browser.

---

### Challenge 7 — Multi-Stage Build

Create a multi-stage Dockerfile for any language of your choice with:
- **Stage 1 (builder):** Install all dependencies including dev tools
- **Stage 2 (runtime):** Copy only the necessary output

Compare the image sizes of the single-stage vs multi-stage build and document the difference.

---

### Challenge 8 — Full Docker Compose Stack

Write a `docker-compose.yml` that runs:
- `PostgreSQL` (instead of MySQL) with a named volume
- `pgAdmin` as the database GUI, connected to PostgreSQL
- A custom network connecting both

Include healthcheck on PostgreSQL and `depends_on` with `condition: service_healthy` on pgAdmin. Start the stack and access pgAdmin in the browser.

---

### Challenge 9 — Resource Limits

Create a container with `--memory="64m"` and `--cpus="0.1"`. Run `docker container stats` and observe the limits being enforced. Then try to run a memory-intensive command inside the container and observe what happens.

---

### Challenge 10 — Override Files

Take the `docker-compose.yml` from Step 15d and create two override files:
- `docker-compose.dev.yml` — mounts source code as bind mount, sets `APP_ENV=development`, no resource limits
- `docker-compose.prod.yml` — sets `restart: always`, stricter resource limits, `APP_ENV=production`

Start the stack in dev mode and production mode separately and verify the `APP_ENV` value from the API endpoint.

---

## ✅ Concepts Covered

| Concept | Where Practiced |
|---|---|
| `docker version` | Step 1 |
| `docker image pull`, `ls`, `rm` | Step 2 |
| `docker container create`, `start`, `stop`, `rm` | Step 3 |
| `docker container ls`, `ls -a` | Step 3b, 3d |
| `docker container logs`, `logs -f` | Step 4b–4c |
| `docker container exec -i -t` | Step 4d |
| `docker container stats` | Step 4e, 7c |
| `--publish HOST:CONTAINER` | Step 5 |
| `--env KEY=VALUE` | Step 6 |
| `--memory`, `--cpus` | Step 7 |
| Bind mount (type=bind) | Step 8 |
| `docker volume create`, `ls`, `rm` | Step 9a–9b |
| Volume mount (type=volume) | Step 9c–9d |
| `docker inspect` | Step 9e |
| Data persistence test | Step 9f |
| `docker container run --rm` (backup) | Step 10b |
| tar backup & restore | Step 10b–10d |
| `docker network ls`, `create`, `rm` | Step 11a–11c |
| `docker network connect`, `disconnect` | Step 11f–11g |
| Container-to-container DNS | Step 11e |
| `docker inspect` (network) | Step 11h |
| FROM, RUN, CMD, COPY, WORKDIR | Step 12e |
| LABEL, ENV, EXPOSE, USER | Step 12e |
| HEALTHCHECK | Step 12e |
| addgroup, adduser (non-root) | Step 12e |
| `docker build`, `--progress=plain`, `--no-cache` | Step 12f–12h |
| .dockerignore | Step 12d |
| ARG, COPY --from (multi-stage) | Step 13a |
| Multi-stage image size comparison | Step 13c |
| `docker login`, `push`, `pull`, `tag` | Step 14 |
| `docker compose up -d`, `down`, `ps` | Step 15e–15k |
| `docker compose logs -f` | Step 15g |
| `env_file`, `.env` | Step 15c |
| `depends_on` with `condition: service_healthy` | Step 15d |
| `healthcheck` in Compose | Step 15d |
| `restart` policy | Step 15d |
| `deploy.resources` limits | Step 15d |
| Multiple Compose files | Step 15m |
| `docker events` | Step 15n |
| `docker system prune` | Step 16f |
| `image prune`, `container prune`, `volume prune` | Step 16b–16e |
