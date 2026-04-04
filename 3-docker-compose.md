# 🐙 Docker Compose Guide

A comprehensive reference for Docker Compose commands and configurations.

---

## 📋 Table of Contents

- [Docker Compose Basics](#-docker-compose-basics)
- [YAML vs JSON Format](#-yaml-vs-json-format)
- [Service Management](#-service-management)
- [Service Configuration](#-service-configuration)
- [Advanced Features](#-advanced-features)

---

## 🔍 Docker Compose Basics

- **Display Docker Compose version**
  
  ```bash
  docker compose version
  ```

---

## 📄 YAML vs JSON Format

Docker Compose uses YAML format for configuration files. Here's a comparison:

- **YAML Format**
  
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

- **JSON Format**
  
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

- **Why YAML?**
  - More human-readable
  - Less verbose (no brackets and quotes everywhere)
  - Supports comments
  - Better for configuration files

---

## 🚀 Service Management

- **Create Docker containers**
  
  ```bash
  docker compose create
  ```
  
  Example `docker-compose.yml`:
  
  ```yaml
  services:
    nginx-example:
      container_name: nginx-example
      image: nginx:latest
  ```

- **Display list of Docker containers**
  
  ```bash
  docker compose ps
  ```

- **Start Docker containers**
  
  ```bash
  docker compose start
  ```

- **Stop Docker containers**
  
  ```bash
  docker compose stop
  ```

- **Delete Docker containers**
  
  ```bash
  docker compose down
  ```

- **Display list of running projects**
  
  ```bash
  docker compose ls
  ```

- **Create and start containers (one command)**
  
  ```bash
  docker compose up
  ```

- **Create and start containers in background**
  
  ```bash
  docker compose up -d
  ```

- **Stop and remove containers, networks, volumes**
  
  ```bash
  docker compose down -v
  ```

---

## ⚙️ Service Configuration

### Multiple Services

- **YAML Configuration**
  
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
  
  Commands:
  
  ```bash
  docker compose create
  docker compose start
  ```

### Port Forwarding

- **Short Syntax**
  
  ```yaml
  ports:
    - "HOSTPORT:CONTAINERPORT"
  ```

- **Long Syntax**
  
  ```yaml
  ports:
    - protocol: tcp
      published: HOSTPORT
      target: CONTAINERPORT
  ```
  
  Example:
  
  ```yaml
  services:
    webserver:
      container_name: nginx-web
      image: nginx:latest
      ports:
        - "8080:80"
        - "8443:443"
  ```
  
  With protocol:
  
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

### Environment Variables

- **Syntax**
  
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

- **Using .env file**
  
  ```yaml
  services:
    database:
      container_name: mysql-db
      image: mysql:8.0
      env_file:
        - .env
  ```

### Mount Binding

- **Short Syntax**
  
  ```yaml
  volumes:
    - "SOURCE:TARGET:MODE"
  ```

- **Long Syntax**
  
  ```yaml
  volumes:
    - type: bind
      source: SOURCE
      target: TARGET
  ```
  
  Example (Bind Mount):
  
  ```yaml
  services:
    webserver:
      container_name: nginx-web
      image: nginx:latest
      volumes:
        - "./html:/usr/share/nginx/html:ro"
  ```
  
  Long syntax:
  
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

- **Example (Volume Mount)**
  
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

### Create Volume

- **Syntax**
  
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

### Create Network

- **Syntax**
  
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
  
  Driver options:
  - `bridge` - Default network driver (isolated network)
  - `host` - Remove network isolation (use host network)
  - `none` - Disable networking

### Depends On

- **Syntax**
  
  ```yaml
  depends_on:
    - container_name
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

- **With health check condition**
  
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

### Restart Policy

- **Syntax**
  
  ```yaml
  restart: policy
  ```
  
  Options:
  - `no` - Never restart by default
  - `always` - Always restart if the container stops
  - `on-failure` - Restart if the container encounters an error upon exit
  - `unless-stopped` - Always restart the container, except when stopped manually
  
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

---

## 🔧 Advanced Features

### Resource Limits

- **Syntax**
  
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

- **Syntax**
  
  ```yaml
  build:
    context: "path/to/app"
    dockerfile: dockerfile-name
  image: "imagename:tagname"
  ```
  
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

- **With build arguments**
  
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

### Health Check

- **Syntax**
  
  ```yaml
  healthcheck:
    test: ["CMD", "COMMAND"]
    interval: DURATION
    timeout: DURATION
    start_period: DURATION
    retries: N
  ```
  
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

### Monitor Docker Events

- **View realtime logs of Docker events for specific container**
  
  ```bash
  docker events --filter 'container=container-name'
  ```
  
  Example:
  
  ```bash
  docker events --filter 'container=nginx-web'
  ```

- **Filter by multiple containers**
  
  ```bash
  docker events --filter 'container=nginx-web' --filter 'container=mysql-db'
  ```

### Multiple Compose Files

- **Extend service using multiple Compose files**
  
  ```bash
  docker compose -f docker-compose-1.yaml -f docker-compose-2.yaml up
  ```
  
  Example:
  
  `docker-compose.yml` (base configuration):
  
  ```yaml
  services:
    webserver:
      container_name: nginx-web
      image: nginx:latest
      ports:
        - "8080:80"
  ```
  
  `docker-compose.override.yml` (development overrides):
  
  ```yaml
  services:
    webserver:
      volumes:
        - ./html:/usr/share/nginx/html
      environment:
        DEBUG: "true"
  ```
  
  `docker-compose.prod.yml` (production overrides):
  
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
  
  Commands:
  
  ```bash
  # Development
  docker compose up -d
  
  # Production
  docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
  ```

---

## 📝 Complete Example

Here's a complete `docker-compose.yml` with multiple services:

```yaml
version: '3.8'

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
      - database
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
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: myapp
      MYSQL_USER: admin
      MYSQL_PASSWORD: admin123
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

**Start the stack:**

```bash
docker compose up -d
```

**Access:**
- Web: http://localhost:8080
- Adminer: http://localhost:8081

---

## 🎯 Quick Reference

| Command | Description |
|---------|-------------|
| `docker compose version` | Show Docker Compose version |
| `docker compose up` | Create and start containers |
| `docker compose up -d` | Create and start in background |
| `docker compose down` | Stop and remove containers |
| `docker compose start` | Start existing containers |
| `docker compose stop` | Stop running containers |
| `docker compose restart` | Restart containers |
| `docker compose ps` | List containers |
| `docker compose ls` | List running projects |
| `docker compose logs` | View logs |
| `docker compose logs -f` | Follow log output |
| `docker compose exec` | Execute command in container |
| `docker compose build` | Build or rebuild services |
| `docker compose pull` | Pull service images |
| `docker compose push` | Push service images |

---

## 💡 Best Practices

- **File Organization**
  - Use `docker-compose.yml` for base configuration
  - Use `docker-compose.override.yml` for local development
  - Use separate files for different environments (dev, staging, prod)

- **Service Configuration**
  - Always specify image tags (avoid `latest`)
  - Use health checks for critical services
  - Set resource limits to prevent resource exhaustion
  - Use restart policies appropriate for your service

- **Networking**
  - Create custom networks for service isolation
  - Use `depends_on` to control startup order
  - Use service names for inter-container communication

- **Volumes**
  - Use named volumes for persistent data
  - Use bind mounts for development only
  - Backup volumes regularly

- **Security**
  - Don't hardcode sensitive data in compose files
  - Use `.env` files for environment variables (add to `.gitignore`)
  - Run containers as non-root users when possible
  - Keep images updated

- **Example .env file**
  
  ```env
  MYSQL_ROOT_PASSWORD=secret
  MYSQL_DATABASE=myapp
  MYSQL_USER=admin
  MYSQL_PASSWORD=admin123
  ```
