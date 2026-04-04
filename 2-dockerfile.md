# 🏗️ Dockerfile & Docker Build Guide

A comprehensive reference for building Docker images and working with Dockerfiles.

---

## 📋 Table of Contents

- [Building Docker Images](#-building-docker-images)
- [Dockerfile Instructions](#-dockerfile-instructions)
- [User Management](#-user-management)
- [Multi-Stage Builds](#-multi-stage-builds)
- [Docker Hub Registry](#-docker-hub-registry)

---

## 🔨 Building Docker Images

- **Build docker image from Dockerfile**
  
  ```bash
  docker build -t dockerhub-username/imagename:tag path/to/dockerfile-folder
  ```
  
  Example:
  
  ```bash
  docker build -t johndoe/myapp:1.0 .
  docker build -t johndoe/webapp:latest ./app
  ```

- **Build docker image with detailed output**
  
  ```bash
  docker build -t dockerhub-username/imagename:tag path/to/dockerfile-folder --progress=plain
  ```
  
  Example:
  
  ```bash
  docker build -t johndoe/myapp:1.0 . --progress=plain
  ```

- **Build docker image without cache**
  
  ```bash
  docker build -t dockerhub-username/imagename:tag path/to/dockerfile-folder --no-cache
  ```
  
  Example:
  
  ```bash
  docker build -t johndoe/myapp:1.0 . --no-cache
  ```

---

## 📝 Dockerfile Instructions

- **FROM - Base image**
  
  Specifies the base image to use for the build.
  
  ```dockerfile
  FROM imagename:tag
  ```
  
  Example:
  
  ```dockerfile
  FROM node:18-alpine
  FROM ubuntu:22.04
  FROM python:3.11-slim
  ```

- **RUN - Execute commands during build**
  
  Executes commands during the image build stage.
  
  ```dockerfile
  RUN instruction
  ```
  
  Example:
  
  ```dockerfile
  RUN apt-get update && apt-get install -y curl
  RUN npm install
  RUN pip install -r requirements.txt
  ```

- **CMD - Default container command**
  
  Specifies the default command to execute when the container starts.
  
  ```dockerfile
  CMD instruction
  ```
  
  Example:
  
  ```dockerfile
  CMD ["npm", "start"]
  CMD ["python", "app.py"]
  CMD ["nginx", "-g", "daemon off;"]
  ```

- **LABEL - Add metadata**
  
  Adds metadata to the Docker image.
  
  ```dockerfile
  LABEL key=value
  ```
  
  Example:
  
  ```dockerfile
  LABEL version="1.0"
  LABEL maintainer="john@example.com"
  LABEL description="My web application"
  ```

- **ADD - Add files with advanced features**
  
  Adds files to the image (supports URL sources and auto-extracts archives like .zip).
  
  ```dockerfile
  ADD path/to/source-file path/to/destination-file
  ```
  
  Example:
  
  ```dockerfile
  ADD app.tar.gz /app/
  ADD https://example.com/file.zip /tmp/
  ADD config.json /etc/app/
  ```

- **COPY - Copy files**
  
  Copies files to the Docker image.
  
  ```dockerfile
  COPY path/to/source-file path/to/destination-file
  ```
  
  Example:
  
  ```dockerfile
  COPY app.js /app/
  COPY package*.json /app/
  COPY ./src /app/src
  ```

- **EXPOSE - Specify container port**
  
  Specifies the port that will be used to access the Docker container.
  
  ```dockerfile
  EXPOSE PORT
  ```
  
  Example:
  
  ```dockerfile
  EXPOSE 80
  EXPOSE 3000
  EXPOSE 8080
  ```

- **ENV - Environment variables**
  
  Adds environment variables to the image.
  
  ```dockerfile
  ENV key=value
  ```
  
  Example:
  
  ```dockerfile
  ENV NODE_ENV=production
  ENV PORT=3000
  ENV DATABASE_URL=postgresql://localhost/mydb
  ```

- **VOLUME - Mount point**
  
  Binds host directory to Docker volume.
  
  ```dockerfile
  VOLUME /path/to/source-file
  ```
  
  Example:
  
  ```dockerfile
  VOLUME /data
  VOLUME /var/log/nginx
  VOLUME /app/uploads
  ```

- **WORKDIR - Set working directory**
  
  Sets the working directory inside the Docker container.
  
  ```dockerfile
  WORKDIR /path
  ```
  
  Example:
  
  ```dockerfile
  WORKDIR /app
  WORKDIR /usr/src/app
  WORKDIR /var/www/html
  ```

- **ARG - Build-time variables**
  
  Defines build-time variables (only available during docker image build).
  
  ```dockerfile
  ARG key=value
  ```
  
  Example:
  
  ```dockerfile
  ARG VERSION=1.0
  ARG BUILD_DATE
  ARG NODE_VERSION=18
  ```
  
  Usage in build:
  
  ```bash
  docker build --build-arg VERSION=2.0 -t myapp:2.0 .
  ```

- **ENTRYPOINT - Container startup command**
  
  Defines the container startup command.
  
  ```dockerfile
  ENTRYPOINT ["executable", "param1"]
  ```
  
  Example:
  
  ```dockerfile
  ENTRYPOINT ["python", "app.py"]
  ENTRYPOINT ["node", "server.js"]
  ENTRYPOINT ["/usr/bin/nginx"]
  ```

- **HEALTHCHECK - Health check configuration**
  
  Defines health check for Docker container.
  
  ```dockerfile
  HEALTHCHECK [OPTIONS] CMD command
  ```
  
  Options:
  - `--interval=DURATION` (default: 30s)
  - `--timeout=DURATION` (default: 30s)
  - `--start-period=DURATION` (default: 0s)
  - `--retries=N` (default: 3)
  
  Example:
  
  ```dockerfile
  HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
    CMD curl -f http://localhost/ || exit 1
  
  HEALTHCHECK --interval=5m --timeout=3s \
    CMD curl -f http://localhost:8080/health || exit 1
  ```

---

## 👤 User Management

Creating and using non-root users in Docker containers for better security.

- **Step 1: Create new group**
  
  ```dockerfile
  RUN addgroup -S groupname
  ```
  
  Example:
  
  ```dockerfile
  RUN addgroup -S devteam
  ```

- **Step 2: Create new user and assign to group**
  
  ```dockerfile
  RUN adduser -S -D -h /app username groupname
  ```
  
  Flags:
  - `-S` - Create as system user
  - `-D` - Don't assign a password
  - `-h` - Specify home directory
  
  Example:
  
  ```dockerfile
  RUN adduser -S -D -h /app johndoe devteam
  ```

- **Step 3: Change ownership of working directory**
  
  ```dockerfile
  RUN chown -R username:groupname /app
  ```
  
  Example:
  
  ```dockerfile
  RUN chown -R johndoe:devteam /app
  ```

- **Step 4: Switch to new user**
  
  ```dockerfile
  USER username
  ```
  
  Example:
  
  ```dockerfile
  USER johndoe
  ```

- **Complete example:**
  
  ```dockerfile
  FROM node:18-alpine
  
  WORKDIR /app
  
  # Create group and user
  RUN addgroup -S devteam && \
      adduser -S -D -h /app johndoe devteam
  
  # Copy application files
  COPY package*.json ./
  RUN npm install
  
  COPY . .
  
  # Change ownership
  RUN chown -R johndoe:devteam /app
  
  # Switch to non-root user
  USER johndoe
  
  EXPOSE 3000
  CMD ["npm", "start"]
  ```

---

## 🏗️ Multi-Stage Builds

Multi-stage builds help create smaller, more secure images by separating build and runtime environments.

**Basic structure:**

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

---

## 🐳 Docker Hub Registry

- **Login to Docker Hub**
  
  ```bash
  docker login -u username
  ```
  
  Steps:
  - Run the login command with your Docker Hub username
  - When prompted for password, enter your Personal Access Token (not your account password)
  - For better security, use Personal Access Tokens instead of passwords
  
  Example:
  
  ```bash
  docker login -u johndoe
  # Enter Personal Access Token when prompted
  ```

- **Push image to Docker Hub**
  
  ```bash
  docker push imagename:tag
  ```
  
  Example:
  
  ```bash
  docker push johndoe/myapp:1.0
  docker push johndoe/myapp:latest
  ```

- **Pull image from Docker Hub**
  
  ```bash
  docker pull imagename:tag
  ```
  
  Example:
  
  ```bash
  docker pull johndoe/myapp:1.0
  docker pull nginx:alpine
  ```

- **Tag image before pushing**
  
  ```bash
  docker tag local-image:tag username/repository:tag
  ```
  
  Example:
  
  ```bash
  docker tag myapp:1.0 johndoe/myapp:1.0
  docker push johndoe/myapp:1.0
  ```

---

## 📄 .dockerignore

Create a `.dockerignore` file to exclude files and directories from the build context.

**Syntax:**

```
folder/*.matcher
folder/subfolder
```

**Example .dockerignore file:**

```
# Git files
.git
.gitignore

# Node modules
node_modules
npm-debug.log

# Environment files
.env
.env.local
.env.*.local

# Documentation
README.md
docs/

# Test files
test/
*.test.js
__tests__/

```

---

## 💡 Best Practices

**Image Building**
- Use specific base image tags instead of `latest`
- Minimize the number of layers by combining RUN commands
- Order instructions from least to most frequently changing
- Use `.dockerignore` to exclude unnecessary files

**Security**
- Always run containers as non-root users
- Use multi-stage builds to reduce final image size
- Scan images for vulnerabilities regularly
- Keep base images updated

**Optimization**
- Leverage build cache by ordering instructions properly
- Use multi-stage builds to separate build and runtime dependencies
- Minimize image size by removing unnecessary files
- Use alpine-based images when possible
