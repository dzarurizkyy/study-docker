# 🐙 Docker Compose Basics

A practical reference guide for running multi-container applications with Docker Compose — covering the Compose file, service lifecycle commands, ports, environment variables, storage, networking, startup order, and multi-environment setups.

---

## 📋 Table of Contents

- [Introduction](#-introduction)
  - [What Is Docker Compose?](#what-is-docker-compose)
  - [Compose at a Glance](#compose-at-a-glance)
- [Version & Info](#-version--info)
- [YAML vs JSON Format](#-yaml-vs-json-format)
  - [YAML Format](#yaml-format)
  - [JSON Format](#json-format)
  - [Why YAML?](#why-yaml)
- [Service Management](#-service-management)
  - [The Compose File](#the-compose-file)
  - [Create and Start](#create-and-start)
  - [Up: Create and Start in One Command](#up-create-and-start-in-one-command)
  - [List Containers and Projects](#list-containers-and-projects)
  - [Stop and Remove](#stop-and-remove)
  - [Other Useful Commands](#other-useful-commands)
- [Service Configuration](#-service-configuration)
  - [Multiple Services](#multiple-services)
  - [Port Forwarding](#port-forwarding)
  - [Environment Variables](#environment-variables)
  - [Mount Binding](#mount-binding)
  - [Create Volume](#create-volume)
  - [Create Network](#create-network)
  - [Depends On](#depends-on)
  - [Restart Policy](#restart-policy)
- [Advanced Features](#-advanced-features)
  - [Resource Limits](#resource-limits)
  - [Build from Dockerfile](#build-from-dockerfile)
  - [Health Check](#health-check)
  - [Monitor Docker Events](#monitor-docker-events)
  - [Multiple Compose Files](#multiple-compose-files)
- [Complete Example](#-complete-example)
- [Compose Flow](#-compose-flow)
- [Quick Reference](#-quick-reference)
- [Best Practices](#-best-practices)

---

## 🎯 Introduction

### What Is Docker Compose?

- Docker Compose is a tool for defining and running **multi-container** applications from a single YAML file
- Instead of typing a long `docker container create ...` command for every container, you describe all services — images, ports, environment variables, volumes, networks — in one file
- One command (`docker compose up`) then creates and starts the whole stack, and another (`docker compose down`) tears it all down

> Reference: [docs.docker.com/compose](https://docs.docker.com/compose/)

### Compose at a Glance

| Piece | Role |
| --- | --- |
| `docker-compose.yml` / `compose.yaml` | The Compose file — describes every service, volume, and network |
| **Project** | One running stack; named after the folder containing the Compose file by default |
| **Service** | One entry under `services:` — becomes one (or more) containers |
| `volumes:` (top level) | Named volumes the project creates and manages |
| `networks:` (top level) | Networks the project creates and manages |
| `.env` | Variables loaded automatically for `${VAR}` substitution in the Compose file |

> **Key Insight:** Every Compose key maps to a `docker` CLI flag you already know — `ports` is `--publish`, `environment` is `--env`, `volumes` is `--mount`, `networks` is `--network`, `deploy.resources` is `--memory`/`--cpus`. Compose doesn't add new container features; it makes the configuration declarative and repeatable.

---

## 🔍 Version & Info

```bash
docker compose version
```

> **Note:** Modern Docker ships Compose V2 as a plugin, invoked as `docker compose` (with a space). The old standalone `docker-compose` (with a hyphen) is Compose V1 and is no longer maintained.

---

## 📄 YAML vs JSON Format

Compose files are written in YAML. YAML describes the same data structures as JSON — objects, arrays, strings, numbers — just with indentation instead of brackets.

### YAML Format

```yaml
firstName: "Dzaru"
lastName: "Rizky Fathan Fortuna"
hobbies:
  - "Coding"
  - "Reading"
address:
  city: "Surabaya"
  country: "Indonesia"
education:
  - type: Bachelor degree
    name: Universitas Pembangunan Nasional Veteran Jawa Timur
  - type: Master degree
    name: Monash University
```

### JSON Format

```json
{
  "firstName": "Dzaru",
  "lastName": "Rizky Fathan Fortuna",
  "hobbies": [
    "Coding",
    "Reading"
  ],
  "address": {
    "city": "Surabaya",
    "country": "Indonesia"
  },
  "education": [
    {
      "type": "Bachelor degree",
      "name": "Universitas Pembangunan Nasional Veteran Jawa Timur"
    },
    {
      "type": "Master degree",
      "name": "Monash University"
    }
  ]
}
```

### Why YAML?

| YAML | JSON equivalent |
| --- | --- |
| `key: value` | `"key": "value"` |
| Indentation | `{ }` nesting |
| `- item` | `[ "item" ]` |
| `# comment` | Not supported |

- **More human-readable** — less visual noise
- **Less verbose** — no brackets and quotes everywhere
- **Supports comments** — useful for documenting configuration
- **Better for configuration files** — which is why Compose uses it

> **Gotcha:** YAML indentation is meaningful and must use **spaces, not tabs**. A single misaligned line moves a key under the wrong parent — or makes the whole file invalid.

---

## 🚀 Service Management

### The Compose File

All `docker compose` commands read the Compose file in the current directory.

**`docker-compose.yml`**

```yaml
services:
  nginx-example:
    container_name: nginx-example
    image: nginx:latest
```

| Key | Meaning |
| --- | --- |
| `services` | List of services in the project |
| `nginx-example` | Service name — used by other services and by `docker compose` commands |
| `container_name` | Exact name of the created container (optional) |
| `image` | Image to create the container from |

> **Note:** Without `container_name`, Compose names containers `<project>-<service>-<number>`, e.g. `myapp-nginx-example-1`.

### Create and Start

- Create the containers (without starting them):

  ```bash
  docker compose create
  ```

- Start the created containers:

  ```bash
  docker compose start
  ```

### Up: Create and Start in One Command

- Create and start containers, attached to the logs (stop with `Ctrl+C`):

  ```bash
  docker compose up
  ```

- Create and start containers in the background (detached):

  ```bash
  docker compose up -d
  ```

> **Tip:** `docker compose up -d` is the command you'll use most. It also applies changes — after editing the Compose file, run it again and Compose recreates only the services whose configuration changed.

### List Containers and Projects

- Display the containers of the current project:

  ```bash
  docker compose ps
  ```

- Display running Compose projects (add `-a` to include stopped ones):

  ```bash
  docker compose ls
  ```

### Stop and Remove

- Stop running containers (they can be started again):

  ```bash
  docker compose stop
  ```

- Stop and remove containers and networks:

  ```bash
  docker compose down
  ```

- Stop and remove containers, networks, **and volumes**:

  ```bash
  docker compose down -v
  ```

| Command | Containers | Networks | Volumes |
| --- | --- | --- | --- |
| `stop` | Stopped | Kept | Kept |
| `down` | Removed | Removed | Kept |
| `down -v` | Removed | Removed | **Removed** |

> **Gotcha:** `down -v` permanently deletes the project's named volumes — including database data. Use plain `down` unless you really want a clean slate.

### Other Useful Commands

```bash
docker compose restart           # Restart containers
docker compose logs              # View logs of all services
docker compose logs -f webserver # Follow logs of one service
docker compose exec webserver sh # Run a command in a running service
docker compose build             # Build or rebuild services
docker compose pull              # Pull service images
docker compose push              # Push service images
```

---

## 🧩 Service Configuration

### Multiple Services

Each key under `services` defines one service.

```yaml
services:
  container-name1:
    container_name: container-name1
    image: image-name1:tag-name1

  container-name2:
    container_name: container-name2
    image: image-name2:tag-name2
```

Example:

```yaml
services:
  webserver:
    container_name: nginx-web
    image: nginx:alpine

  database:
    container_name: mysql-db
    image: mysql:8.0
```

```bash
docker compose create
docker compose start
```

### Port Forwarding

- **Short syntax**:

  ```yaml
  ports:
    - "HOSTPORT:CONTAINERPORT"
  ```

- **Long syntax**:

  ```yaml
  ports:
    - protocol: tcp
      published: HOSTPORT
      target: CONTAINERPORT
  ```

Example (short syntax):

```yaml
services:
  webserver:
    container_name: nginx-web
    image: nginx:latest
    ports:
      - "8080:80"
      - "8443:443"
```

Example (long syntax, with protocol):

```yaml
services:
  webserver:
    container_name: nginx-web
    image: nginx:latest
    ports:
      - protocol: tcp
        published: 8080
        target: 80
```

| Long syntax key | Meaning |
| --- | --- |
| `published` | Port on the host |
| `target` | Port inside the container |
| `protocol` | `tcp` (default) or `udp` |

> **Tip:** Always quote short-syntax ports (`"8080:80"`). Unquoted values like `22:22` can be misread by YAML as a number in base 60.

### Environment Variables

- **Syntax**:

  ```yaml
  environment:
    KEY: value
  ```

  Example:

  ```yaml
  services:
    database:
      container_name: mysql-db
      image: mysql:8.0
      environment:
        MYSQL_ROOT_PASSWORD: secret
        MYSQL_DATABASE: myapp
        MYSQL_USER: admin
        MYSQL_PASSWORD: admin123
  ```

- **Load from a file** with `env_file`:

  ```yaml
  services:
    database:
      container_name: mysql-db
      image: mysql:8.0
      env_file:
        - .env
  ```

- **Substitute values** from `.env` with `${VAR}`, keeping secrets out of the Compose file:

  ```yaml
  services:
    database:
      image: mysql:8.0
      environment:
        MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
  ```

> **Note:** These two features are different. `env_file` passes every variable in the file **into the container**. `${VAR}` substitution happens **in the Compose file itself**, using the `.env` file found in the project directory.

### Mount Binding

- **Short syntax**:

  ```yaml
  volumes:
    - "SOURCE:TARGET:MODE"
  ```

- **Long syntax**:

  ```yaml
  volumes:
    - type: bind
      source: SOURCE
      target: TARGET
  ```

| Part | Meaning |
| --- | --- |
| `SOURCE` | Host path (bind mount) or volume name (volume mount) |
| `TARGET` | Path inside the container |
| `MODE` | `rw` (read-write, default) or `ro` (read-only) |

Example (bind mount, short syntax):

```yaml
services:
  webserver:
    container_name: nginx-web
    image: nginx:latest
    volumes:
      - "./html:/usr/share/nginx/html:ro"
```

Example (bind mount, long syntax):

```yaml
services:
  webserver:
    container_name: nginx-web
    image: nginx:latest
    volumes:
      - type: bind
        source: ./html
        target: /usr/share/nginx/html
        read_only: true
```

Example (volume mount):

```yaml
services:
  database:
    container_name: mysql-db
    image: mysql:8.0
    volumes:
      - dbdata:/var/lib/mysql

volumes:
  dbdata:
    name: mysql-data
```

> **Key Insight:** Unlike the `docker` CLI, Compose accepts **relative** host paths like `./html` — they're resolved relative to the Compose file. A `SOURCE` that starts with `./` or `/` is a bind mount; a plain name like `dbdata` is a volume.

### Create Volume

Named volumes are declared in the top-level `volumes` section and referenced by their key inside a service.

```yaml
volumes:
  unique-key:
    name: volume-name
```

Example:

```yaml
services:
  database:
    container_name: mysql-db
    image: mysql:8.0
    volumes:
      - mysqldata:/var/lib/mysql

volumes:
  mysqldata:
    name: mysql-data
```

> **Note:** `mysqldata` is the key used inside the Compose file; `name: mysql-data` is the real volume name shown by `docker volume ls`. Without `name`, Docker names it `<project>_mysqldata`.

### Create Network

Networks are declared in the top-level `networks` section, and each service lists the networks it joins.

```yaml
networks:
  unique-key:
    name: network-name
    driver: bridge
```

Example:

```yaml
services:
  webserver:
    container_name: nginx-web
    image: nginx:latest
    networks:
      - appnet

  database:
    container_name: mysql-db
    image: mysql:8.0
    networks:
      - appnet

networks:
  appnet:
    name: app-network
    driver: bridge
```

| Driver | Description |
| --- | --- |
| `bridge` | Default network driver — an isolated network on a single host |
| `host` | Removes network isolation — uses the host's network directly |
| `none` | Disables networking |

> **Key Insight:** Services on the same network reach each other by **service name** — `webserver` connects to the database at `database:3306`. Even without a `networks` section, Compose creates a default network for the project, so every service can already talk to the others.

### Depends On

`depends_on` controls startup order: a service is started only after the services it depends on.

```yaml
depends_on:
  - service-name
```

Example:

```yaml
services:
  database:
    container_name: mysql-db
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: secret

  webserver:
    container_name: nginx-web
    image: nginx:latest
    depends_on:
      - database
```

With a health check condition:

```yaml
services:
  database:
    container_name: mysql-db
    image: mysql:8.0
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 5s
      timeout: 3s
      retries: 3

  webserver:
    container_name: nginx-web
    image: nginx:latest
    depends_on:
      database:
        condition: service_healthy
```

| Condition | Waits until the dependency… |
| --- | --- |
| `service_started` | Has started (default) |
| `service_healthy` | Reports `healthy` via its health check |
| `service_completed_successfully` | Has run and exited with code `0` |

> **Gotcha:** `depends_on` refers to the **service name** (`database`), not the `container_name` (`mysql-db`). And the plain list form only waits for the container to *start*, not for the app inside to be *ready* — use `condition: service_healthy` when it matters, as with databases.

### Restart Policy

```yaml
restart: policy
```

| Policy | Behavior |
| --- | --- |
| `"no"` | Never restart (default) |
| `always` | Always restart if the container stops |
| `on-failure` | Restart only if the container exits with an error |
| `unless-stopped` | Always restart, except when stopped manually |

Example:

```yaml
services:
  webserver:
    container_name: nginx-web
    image: nginx:latest
    restart: always

  database:
    container_name: mysql-db
    image: mysql:8.0
    restart: unless-stopped
```

> **Gotcha:** Write `restart: "no"` **with quotes** — unquoted, YAML can read `no` as the boolean `false`.

---

## 🔧 Advanced Features

### Resource Limits

```yaml
deploy:
  resources:
    reservations:
      cpus: "0.50"
      memory: "256M"
    limits:
      cpus: "1.00"
      memory: "512M"
```

| Key | Meaning |
| --- | --- |
| `reservations` | Minimum resources guaranteed to the container |
| `limits` | Maximum resources the container may use |
| `cpus` | Number of CPUs (`"0.50"` = 50% of one core) |
| `memory` | Memory size with unit (`M`, `G`) |

Example:

```yaml
services:
  webserver:
    container_name: nginx-web
    image: nginx:latest
    deploy:
      resources:
        reservations:
          cpus: "0.25"
          memory: "128M"
        limits:
          cpus: "0.50"
          memory: "256M"

  database:
    container_name: mysql-db
    image: mysql:8.0
    deploy:
      resources:
        reservations:
          cpus: "0.50"
          memory: "512M"
        limits:
          cpus: "2.00"
          memory: "2G"
```

### Build from Dockerfile

Instead of pulling an image, Compose can build one from a Dockerfile.

```yaml
build:
  context: "path/to/app"
  dockerfile: dockerfile-name
image: "imagename:tagname"
```

| Key | Meaning |
| --- | --- |
| `context` | Build context folder |
| `dockerfile` | Dockerfile name, relative to `context` |
| `image` | Name and tag given to the built image |
| `args` | Build arguments (Dockerfile `ARG`) |

Example:

```yaml
services:
  webapp:
    container_name: my-webapp
    build:
      context: ./app
      dockerfile: Dockerfile
    image: myapp:1.0
    ports:
      - "3000:3000"
```

With build arguments:

```yaml
services:
  webapp:
    container_name: my-webapp
    build:
      context: ./app
      dockerfile: Dockerfile
      args:
        NODE_VERSION: 18
        BUILD_ENV: production
    image: myapp:1.0
```

> **Gotcha:** `docker compose up` only builds the image if it doesn't exist yet. After changing the code or Dockerfile, run `docker compose up -d --build` (or `docker compose build`) to rebuild.

### Health Check

```yaml
healthcheck:
  test: ["CMD", "COMMAND"]
  interval: DURATION
  timeout: DURATION
  start_period: DURATION
  retries: N
```

| Key | Meaning |
| --- | --- |
| `test` | Command run inside the container — exit code `0` means healthy |
| `interval` | Time between checks |
| `timeout` | Time before a single check counts as failed |
| `start_period` | Grace period while the app starts up |
| `retries` | Consecutive failures before `unhealthy` |

Example:

```yaml
services:
  webserver:
    container_name: nginx-web
    image: nginx:latest
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 3s
      start_period: 5s
      retries: 3

  database:
    container_name: mysql-db
    image: mysql:8.0
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5
```

> **Gotcha:** The `test` command runs **inside** the container, so the tool it uses (`curl`, `mysqladmin`) must exist in that image.

### Monitor Docker Events

- View real-time Docker events for a specific container:

  ```bash
  docker events --filter 'container=container-name'
  ```

  Example:

  ```bash
  docker events --filter 'container=nginx-web'
  ```

- Filter by multiple containers:

  ```bash
  docker events --filter 'container=nginx-web' --filter 'container=mysql-db'
  ```

> **Tip:** Events show lifecycle changes — `create`, `start`, `die`, `health_status` — which makes them handy for watching restart policies and health checks in action.

### Multiple Compose Files

Several Compose files can be merged, with later files overriding or extending earlier ones.

```bash
docker compose -f docker-compose-1.yaml -f docker-compose-2.yaml up
```

**`docker-compose.yml`** (base configuration)

```yaml
services:
  webserver:
    container_name: nginx-web
    image: nginx:latest
    ports:
      - "8080:80"
```

**`docker-compose.override.yml`** (development overrides)

```yaml
services:
  webserver:
    volumes:
      - ./html:/usr/share/nginx/html
    environment:
      DEBUG: "true"
```

**`docker-compose.prod.yml`** (production overrides)

```yaml
services:
  webserver:
    restart: always
    deploy:
      resources:
        limits:
          cpus: "1.0"
          memory: "512M"
```

```bash
# Development — base + override, merged automatically
docker compose up -d

# Production — base + prod only
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

> **Key Insight:** `docker-compose.override.yml` is picked up **automatically** by a plain `docker compose up`. As soon as you pass `-f`, only the files you list are used — that's how the production command above skips the development overrides.

---

## 📝 Complete Example

A complete `docker-compose.yml` with a web server, a database, and a database GUI:

```yaml
services:
  # Web Server
  webserver:
    container_name: nginx-web
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html:ro
    networks:
      - app-network
    depends_on:
      database:
        condition: service_healthy
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 3s
      retries: 3
    deploy:
      resources:
        limits:
          cpus: "0.50"
          memory: "256M"

  # Database
  database:
    container_name: mysql-db
    image: mysql:8.0
    env_file:
      - .env
    volumes:
      - dbdata:/var/lib/mysql
    networks:
      - app-network
    restart: always
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5
    deploy:
      resources:
        limits:
          cpus: "1.0"
          memory: "1G"

  # Adminer (Database GUI)
  adminer:
    container_name: adminer
    image: adminer:latest
    ports:
      - "8081:8080"
    networks:
      - app-network
    depends_on:
      - database
    restart: unless-stopped

# Define volumes
volumes:
  dbdata:
    name: mysql-data

# Define networks
networks:
  app-network:
    name: app-network
    driver: bridge
```

**`.env`**

```env
MYSQL_ROOT_PASSWORD=secret
MYSQL_DATABASE=myapp
MYSQL_USER=admin
MYSQL_PASSWORD=admin123
```

```bash
docker compose up -d
```

| Service | URL |
| --- | --- |
| Web | <http://localhost:8080> |
| Adminer | <http://localhost:8081> (server: `database`) |

> **Note:** Older guides start the file with `version: '3.8'`. That key is obsolete in Compose V2 — it's ignored and only triggers a warning, so leave it out.

---

## 🔄 Compose Flow

What happens when you run `docker compose up -d`:

```text
docker-compose.yml (+ docker-compose.override.yml)
  ↓
── .env ──────────────────────────   ${VAR} values substituted
  ↓
Parse + merge                     multiple -f files combined, later files win
  ↓
Project name                      folder name (or -p / name:)
  ↓
Networks + volumes                created if they don't exist yet
  ↓
Images                            pulled (image:) or built (build:)
  ↓
Containers created                ports, env, mounts, limits applied
  ↓
Started in depends_on order       waits for service_healthy when configured
  ↓
Stack running                     ps · logs · exec · events

docker compose down               containers + networks removed
docker compose down -v            … and volumes too
```

| Question | Answer |
| --- | --- |
| Why can't my app connect to `localhost:3306`? | Inside a container, `localhost` is the container itself — use the service name, e.g. `database:3306` |
| Why does the app fail even with `depends_on`? | The plain form only waits for the container to start — use `condition: service_healthy` |
| Why wasn't my code change picked up? | The image was already built — run `docker compose up -d --build` |
| Where did my database data go? | `docker compose down -v` deleted the volume — use plain `down` to keep it |
| Why do my containers have names like `myapp-web-1`? | No `container_name` was set, so Compose used `<project>-<service>-<number>` |
| Why are my dev overrides missing in production? | Passing `-f` disables automatic loading of `docker-compose.override.yml` |

---

## 🎯 Quick Reference

| Concept | Purpose | Key Syntax |
| --- | --- | --- |
| **Version** | Show Compose version | `docker compose version` |
| **Up** | Create + start the stack | `docker compose up -d` |
| **Down** | Stop + remove containers and networks | `docker compose down` |
| **Down with Volumes** | Also remove volumes | `docker compose down -v` |
| **Create / Start / Stop** | Step-by-step lifecycle | `docker compose create` |
| **Restart** | Restart containers | `docker compose restart` |
| **Status** | List project containers | `docker compose ps` |
| **Projects** | List running projects | `docker compose ls` |
| **Logs** | View / follow logs | `docker compose logs -f` |
| **Exec** | Run a command in a service | `docker compose exec webserver sh` |
| **Build / Pull / Push** | Manage service images | `docker compose build` |
| **Service** | Define a container | `services: { web: { image: nginx } }` |
| **Ports** | Publish a port | `ports: ["8080:80"]` |
| **Environment** | Set variables | `environment: { KEY: value }` |
| **Env File** | Load variables from a file | `env_file: [.env]` |
| **Bind Mount** | Map a host folder | `- "./html:/usr/share/nginx/html:ro"` |
| **Volume** | Persistent named storage | `volumes: { dbdata: { name: mysql-data } }` |
| **Network** | Connect services | `networks: { appnet: { driver: bridge } }` |
| **Depends On** | Startup order | `depends_on: { db: { condition: service_healthy } }` |
| **Restart Policy** | Auto-restart behavior | `restart: unless-stopped` |
| **Resource Limits** | Cap CPU / memory | `deploy.resources.limits` |
| **Build** | Build from a Dockerfile | `build: { context: ./app }` |
| **Health Check** | Container health test | `healthcheck: { test: [...] }` |
| **Multiple Files** | Merge configurations | `docker compose -f a.yml -f b.yml up` |

---

## 💡 Best Practices

**✅ Do This**

- **Use `docker-compose.yml` for the base configuration**, `docker-compose.override.yml` for local development, and separate files per environment (dev, staging, prod)
- **Pin image tags** — `mysql:8.0` instead of `mysql:latest`
- **Add health checks to critical services**, and combine them with `depends_on: condition: service_healthy`
- **Set resource limits** to prevent one service from exhausting the host
- **Choose a restart policy** that fits each service
- **Use custom networks** for service isolation, and connect services by **service name**
- **Use named volumes for persistent data**, and back them up regularly
- **Keep secrets in `.env`** (added to `.gitignore`) and reference them with `${VAR}` or `env_file`
- **Run containers as non-root users** when possible, and keep images updated

**❌ Avoid This**

- **Relying on `latest`** — builds stop being reproducible
- **Hardcoding passwords in the Compose file** — it usually ends up in Git
- **Using bind mounts for production data** — they're best for development source code
- **Running `docker compose down -v` casually** — it permanently deletes volume data
- **Assuming plain `depends_on` means "ready"** — it only means "started"
- **Connecting to `localhost` between services** — use the service name
- **Keeping `version:` at the top of the file** — it's obsolete in Compose V2

> Reference: [Docker Compose docs](https://docs.docker.com/compose/) · [Compose file reference](https://docs.docker.com/reference/compose-file/) · [docker compose CLI](https://docs.docker.com/reference/cli/docker/compose/)
