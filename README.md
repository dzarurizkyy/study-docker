# Study Docker 🐳

This repository contains a comprehensive reference guide for Docker — covering core CLI commands for images and containers, building custom images with a Dockerfile, and orchestrating multi-container applications with Docker Compose.

## Installation 🔧

1. **Install Docker Desktop**:

   Download and install [Docker Desktop](https://www.docker.com/get-started) for your OS.

   > **Linux**: Docker Engine can be installed directly without Docker Desktop — see [docs.docker.com/engine/install](https://docs.docker.com/engine/install/)

2. **Verify the Installation**:

   ```bash
   docker --version
   docker compose version
   ```

   > Reference: [docs.docker.com/get-started](https://docs.docker.com/get-started/)

## List of Material 📚

- 🐳 **[Docker Basics](001-docker-basics.md)**

  Core CLI commands for images and containers — port forwarding, environment variables, resource limits, bind mounts, volumes, backup & restore, and networking:

  ```bash
  docker container create \
    --name webserver \
    --publish 8080:80 \
    --mount "type=volume,source=webdata,destination=/usr/share/nginx/html" \
    nginx:latest
  ```

- 🏗️ **[Dockerfile Basics](002-dockerfile.md)**

  Building custom images with a Dockerfile — every core instruction, running as a non-root user, multi-stage builds, `.dockerignore`, and publishing to Docker Hub:

  ```dockerfile
  # Stage 1: Build
  FROM golang:1.18-alpine AS builder
  WORKDIR /app/
  COPY main.go .
  RUN go build -o /app/main main.go

  # Stage 2: Runtime
  FROM alpine:3
  WORKDIR /app/
  COPY --from=builder /app/main ./
  CMD ["/app/main"]
  ```

- 🐙 **[Docker Compose Basics](003-docker-compose.md)**

  Running multi-container applications with Docker Compose — services, ports, environment variables, volumes, networks, startup order, and multi-environment setups:

  ```yaml
  services:
    webserver:
      image: nginx:latest
      ports:
        - "8080:80"
      networks:
        - app-network
      depends_on:
        database:
          condition: service_healthy

    database:
      image: mysql:8.0
      env_file:
        - .env
      networks:
        - app-network
  ```

## 📍 References

- [Udemy](https://www.udemy.com/course/docker-pemula)

## 👨‍💻 Contributors

- [Dzaru Rizky Fathan Fortuna](https://www.linkedin.com/in/dzarurizky)
