# 🐳 Dockerfile Basics

A practical reference guide for building Docker images — covering `docker build`, every core Dockerfile instruction, non-root users, multi-stage builds, `.dockerignore`, and publishing to Docker Hub.

---

## 📋 Table of Contents

- [Introduction](#-introduction)
  - [What Is a Dockerfile?](#what-is-a-dockerfile)
  - [The Build Toolchain at a Glance](#the-build-toolchain-at-a-glance)
- [Building Docker Images](#-building-docker-images)
  - [Basic Build](#basic-build)
  - [Detailed Build Output](#detailed-build-output)
  - [Build Without Cache](#build-without-cache)
- [Dockerfile Instructions](#-dockerfile-instructions)
  - [FROM](#from)
  - [RUN](#run)
  - [CMD](#cmd)
  - [ENTRYPOINT](#entrypoint)
  - [CMD vs ENTRYPOINT](#cmd-vs-entrypoint)
  - [LABEL](#label)
  - [ADD](#add)
  - [COPY](#copy)
  - [EXPOSE](#expose)
  - [ENV](#env)
  - [ARG](#arg)
  - [VOLUME](#volume)
  - [WORKDIR](#workdir)
  - [HEALTHCHECK](#healthcheck)
- [User Management](#-user-management)
  - [Step-by-Step](#step-by-step)
  - [Complete Example](#complete-example)
- [Multi-Stage Builds](#-multi-stage-builds)
- [Dockerignore](#-dockerignore)
- [Docker Hub Registry](#-docker-hub-registry)
  - [Login](#login)
  - [Tag an Image](#tag-an-image)
  - [Push an Image](#push-an-image)
  - [Pull an Image](#pull-an-image)
- [Build Flow](#-build-flow)
- [Quick Reference](#-quick-reference)
- [Best Practices](#-best-practices)

---

## 🎯 Introduction

### What Is a Dockerfile?

- A **Dockerfile** is a plain-text file of instructions that describes, step by step, how to build a Docker image
- `docker build` reads the Dockerfile top to bottom and turns each instruction into a **layer** of the final image
- Because the whole environment is written down as code, anyone can rebuild the exact same image — on a laptop, in CI, or in production

> Reference: [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)

### The Build Toolchain at a Glance

| Piece | Role |
| --- | --- |
| `Dockerfile` | The recipe — base image, files to copy, commands to run, how the container starts |
| Build context | The folder sent to the builder; `COPY` and `ADD` can only read files from inside it |
| `.dockerignore` | Excludes files from the build context (like `.gitignore` for builds) |
| `docker build` | Executes the Dockerfile and produces an image |
| Build cache | Reuses layers whose instruction and inputs haven't changed |
| Docker Hub | The default registry where images are pushed and pulled |

> **Key Insight:** Instructions run at two different moments. `RUN`, `COPY`, `ADD`, and `ARG` act at **build time** and are baked into the image. `CMD`, `ENTRYPOINT`, and `HEALTHCHECK` only describe what happens at **run time**, when a container starts. Most Dockerfile surprises come from mixing the two up — see [Build Flow](#-build-flow).

---

## 🔨 Building Docker Images

### Basic Build

```bash
docker build -t dockerhub-username/imagename:tag path/to/build-context
```

Example:

```bash
docker build -t johndoe/myapp:1.0 .
docker build -t johndoe/webapp:latest ./app
```

| Part | Meaning |
| --- | --- |
| `-t` | Name and tag for the image (`username/imagename:tag`) |
| `.` / `./app` | Build context — the folder whose files the Dockerfile can access |

> **Note:** By default `docker build` looks for a file named `Dockerfile` inside the build context. Use `-f path/to/Dockerfile` when it lives elsewhere or has a different name.

### Detailed Build Output

`--progress=plain` prints the full output of every step instead of the collapsed view — handy for debugging a failing `RUN`.

```bash
docker build -t dockerhub-username/imagename:tag path/to/build-context --progress=plain
```

Example:

```bash
docker build -t johndoe/myapp:1.0 . --progress=plain
```

### Build Without Cache

`--no-cache` rebuilds every layer from scratch, ignoring the build cache.

```bash
docker build -t dockerhub-username/imagename:tag path/to/build-context --no-cache
```

Example:

```bash
docker build -t johndoe/myapp:1.0 . --no-cache
```

> **Tip:** Combine both flags (`--progress=plain --no-cache`) to see the output of `RUN` steps that would otherwise be skipped as `CACHED`.

---

## 📝 Dockerfile Instructions

### FROM

Specifies the base image the build starts from. Every Dockerfile begins with `FROM`.

```dockerfile
FROM imagename:tag
```

Example:

```dockerfile
FROM node:18-alpine
FROM ubuntu:22.04
FROM python:3.11-slim
```

> **Note:** A Dockerfile normally has one `FROM`. Several `FROM` lines mean a [multi-stage build](#-multi-stage-builds).

### RUN

Executes a command **during the image build** and saves the result as a new layer.

```dockerfile
RUN instruction
```

Example:

```dockerfile
RUN apt-get update && apt-get install -y curl
RUN npm install
RUN pip install -r requirements.txt
```

> **Tip:** Chain related commands with `&&` in a single `RUN`. Each `RUN` creates a layer, and `apt-get update` in a separate layer can be cached and go stale.

### CMD

Specifies the default command to execute **when the container starts**.

```dockerfile
CMD ["executable", "param1", "param2"]
```

Example:

```dockerfile
CMD ["npm", "start"]
CMD ["python", "app.py"]
CMD ["nginx", "-g", "daemon off;"]
```

> **Gotcha:** Only the **last** `CMD` in a Dockerfile takes effect, and it is replaced entirely by any command passed to `docker container run imagename <command>`.

### ENTRYPOINT

Defines the executable that always runs when the container starts.

```dockerfile
ENTRYPOINT ["executable", "param1"]
```

Example:

```dockerfile
ENTRYPOINT ["python", "app.py"]
ENTRYPOINT ["node", "server.js"]
ENTRYPOINT ["/usr/bin/nginx"]
```

### CMD vs ENTRYPOINT

| | `CMD` | `ENTRYPOINT` |
| --- | --- | --- |
| Purpose | Default command (or default arguments) | Fixed executable |
| `docker container run image args` | `args` **replaces** `CMD` | `args` are **appended** to `ENTRYPOINT` |
| Override | Pass a command after the image name | `--entrypoint` flag |

When both are present, `CMD` becomes the default arguments of `ENTRYPOINT`:

```dockerfile
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "8000"]
# Runs: python app.py --port 8000
```

> **Tip:** Prefer the JSON array (exec) form `["a", "b"]` over the shell form `a b` — the process then receives stop signals directly, so `docker container stop` shuts it down cleanly.

### LABEL

Adds metadata to the image as key-value pairs.

```dockerfile
LABEL key=value
```

Example:

```dockerfile
LABEL version="1.0"
LABEL maintainer="john@example.com"
LABEL description="My web application"
```

> **Tip:** Read labels back with `docker image inspect imagename:tag`.

### ADD

Adds files to the image, with two extra features over `COPY`: it can download from a URL, and it auto-extracts **local** tar archives.

```dockerfile
ADD path/to/source-file path/to/destination-file
```

Example:

```dockerfile
ADD app.tar.gz /app/
ADD https://example.com/file.zip /tmp/
ADD config.json /etc/app/
```

> **Gotcha:** Auto-extraction only applies to local tar archives (`.tar`, `.tar.gz`, `.tar.bz2`, `.tar.xz`). A `.zip` file — or any file downloaded from a URL — is copied as-is, **not** extracted.

### COPY

Copies files from the build context into the image.

```dockerfile
COPY path/to/source-file path/to/destination-file
```

Example:

```dockerfile
COPY app.js /app/
COPY package*.json /app/
COPY ./src /app/src
```

> **Key Insight:** `COPY` does exactly one predictable thing, which is why it is preferred over `ADD`. Reach for `ADD` only when you specifically need tar auto-extraction or a URL source.

### EXPOSE

Documents the port the application inside the container listens on.

```dockerfile
EXPOSE PORT
```

Example:

```dockerfile
EXPOSE 80
EXPOSE 3000
EXPOSE 8080
```

> **Gotcha:** `EXPOSE` does **not** publish the port to the host. You still need `--publish 8080:80` when creating the container.

### ENV

Sets environment variables that exist both during the build and inside every container started from the image.

```dockerfile
ENV key=value
```

Example:

```dockerfile
ENV NODE_ENV=production
ENV PORT=3000
ENV DATABASE_URL=postgresql://localhost/mydb
```

Reference the value later in the Dockerfile with `$KEY` or `${KEY}`:

```dockerfile
ENV APP_DIR=/app
WORKDIR ${APP_DIR}
```

> **Note:** Values can be overridden at run time with `docker container create --env KEY=VALUE`.

### ARG

Defines a build-time variable — available **only while the image is being built**, not inside the running container.

```dockerfile
ARG key=value
```

Example:

```dockerfile
ARG VERSION=1.0
ARG BUILD_DATE
ARG NODE_VERSION=18
```

Pass a value at build time with `--build-arg`:

```bash
docker build --build-arg VERSION=2.0 -t myapp:2.0 .
```

| | `ARG` | `ENV` |
| --- | --- | --- |
| Available during build | ✅ | ✅ |
| Available in the running container | ❌ | ✅ |
| Set from the command line | `--build-arg` | `--env` (at run time) |

> **Gotcha:** Don't pass secrets through `ARG` — build-arg values can be read back from the image history with `docker image history`.

### VOLUME

Declares a directory inside the container as a mount point for persistent data.

```dockerfile
VOLUME /path/in/container
```

Example:

```dockerfile
VOLUME /data
VOLUME /var/log/nginx
VOLUME /app/uploads
```

> **Note:** `VOLUME` cannot choose a host folder. If no volume is mounted there at run time, Docker automatically creates an **anonymous volume** for that path. To control where data lives, mount a named volume or bind mount when creating the container.

### WORKDIR

Sets the working directory for every following `RUN`, `CMD`, `ENTRYPOINT`, `COPY`, and `ADD`. The directory is created if it doesn't exist.

```dockerfile
WORKDIR /path
```

Example:

```dockerfile
WORKDIR /app
WORKDIR /usr/src/app
WORKDIR /var/www/html
```

> **Tip:** Use `WORKDIR` instead of `RUN cd /app` — a `cd` only lasts for that single `RUN`.

### HEALTHCHECK

Tells Docker how to test whether the container is still working. The container's status then shows `starting`, `healthy`, or `unhealthy`.

```dockerfile
HEALTHCHECK [OPTIONS] CMD command
```

| Option | Default | Meaning |
| --- | --- | --- |
| `--interval=DURATION` | `30s` | Time between checks |
| `--timeout=DURATION` | `30s` | Time before a single check counts as failed |
| `--start-period=DURATION` | `0s` | Grace period while the app starts up |
| `--retries=N` | `3` | Consecutive failures before `unhealthy` |

Example:

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost/ || exit 1

HEALTHCHECK --interval=5m --timeout=3s \
  CMD curl -f http://localhost:8080/health || exit 1
```

> **Gotcha:** The check runs **inside** the container, so the tool it uses (`curl` here) must be installed in the image. Many slim and Alpine images don't include `curl` by default.

---

## 👤 User Management

> **Key Insight:** By default, processes inside a container run as `root`. If the app is compromised, the attacker gets root inside the container. Creating a dedicated non-root user limits the damage.

### Step-by-Step

The commands below use Alpine's (BusyBox) `addgroup`/`adduser` syntax.

1. **Create a new group**:

   ```dockerfile
   RUN addgroup -S groupname
   ```

   Example:

   ```dockerfile
   RUN addgroup -S devteam
   ```

2. **Create a new user and assign it to the group**:

   ```dockerfile
   RUN adduser -S -D -h /app -G groupname username
   ```

   | Flag | Meaning |
   | --- | --- |
   | `-S` | Create a system user |
   | `-D` | Don't assign a password |
   | `-h` | Home directory |
   | `-G` | Group to add the user to |

   Example:

   ```dockerfile
   RUN adduser -S -D -h /app -G devteam johndoe
   ```

3. **Change ownership of the working directory**:

   ```dockerfile
   RUN chown -R username:groupname /app
   ```

   Example:

   ```dockerfile
   RUN chown -R johndoe:devteam /app
   ```

4. **Switch to the new user** — every instruction after this, and the container itself, runs as that user:

   ```dockerfile
   USER username
   ```

   Example:

   ```dockerfile
   USER johndoe
   ```

> **Note:** Debian/Ubuntu-based images use different flags: `groupadd -r devteam && useradd -r -g devteam -d /app johndoe`.

### Complete Example

```dockerfile
FROM node:18-alpine

WORKDIR /app

# Create group and user
RUN addgroup -S devteam && \
    adduser -S -D -h /app -G devteam johndoe

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

> **Tip:** `COPY --chown=johndoe:devteam . .` sets ownership while copying, so you can skip the separate `chown -R` layer, which can be slow and adds size.

---

## 🧱 Multi-Stage Builds

A multi-stage build uses several `FROM` instructions in one Dockerfile. Early stages hold the heavy build tools; the final stage copies **only the finished output**, so compilers and source code never reach the image you ship.

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

| Part | Meaning |
| --- | --- |
| `AS builder` | Names a stage so later stages can refer to it |
| `COPY --from=builder` | Copies files out of the `builder` stage instead of the build context |
| Last `FROM` | Becomes the final image — everything in earlier stages is discarded |

> **Key Insight:** The `golang` image is hundreds of MB, while `alpine` is only a few MB. The final image here holds just the compiled binary on top of Alpine, which makes it smaller and gives attackers less to work with.

---

## 📄 Dockerignore

A `.dockerignore` file in the root of the build context excludes files and folders from being sent to the builder. That makes builds faster and keeps secrets and junk out of the image.

**Syntax:**

```text
folder/*.matcher
folder/subfolder
```

**Example `.dockerignore`:**

```text
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

> **Gotcha:** Without a `.dockerignore`, `COPY . .` copies everything, including `.env` files with secrets and a local `node_modules` that can conflict with the one installed inside the image.

---

## 🐳 Docker Hub Registry

### Login

```bash
docker login -u username
```

- Run the login command with your Docker Hub username
- When prompted for a password, enter a **Personal Access Token**, not your account password

Example:

```bash
docker login -u johndoe
# Enter Personal Access Token when prompted
```

> **Tip:** Create a Personal Access Token under Docker Hub → Account Settings → Personal access tokens. A token can be scoped and revoked without changing your password.

### Tag an Image

Docker Hub only accepts images named `username/repository:tag`. If an image was built without your username, tag it before pushing.

```bash
docker tag local-image:tag username/repository:tag
```

Example:

```bash
docker tag myapp:1.0 johndoe/myapp:1.0
```

### Push an Image

```bash
docker push username/imagename:tag
```

Example:

```bash
docker push johndoe/myapp:1.0
docker push johndoe/myapp:latest
```

> **Gotcha:** Pushing `myapp:1.0` without the `username/` prefix fails, because Docker tries to push to the official `library/` namespace, which you don't have access to.

### Pull an Image

```bash
docker pull imagename:tag
```

Example:

```bash
docker pull johndoe/myapp:1.0
docker pull nginx:alpine
```

---

## 🔄 Build Flow

What happens between writing a Dockerfile and running a container from it:

```text
Project folder (build context)
  ↓
── .dockerignore ────────────────   excluded files never leave your machine
  ↓
docker build -t user/app:1.0 .
  ↓
FROM                              pull the base image
  ↓
ARG / ENV / WORKDIR / LABEL       configure the build
  ↓
COPY / ADD / RUN                  each instruction = one cached layer
  ↓                               a changed layer invalidates every layer after it
CMD / ENTRYPOINT / EXPOSE /       recorded as metadata — nothing runs yet
HEALTHCHECK / USER / VOLUME
  ↓
Image  user/app:1.0
  ↓  docker push                  → Docker Hub
  ↓  docker container run
Container starts                  ENTRYPOINT + CMD execute here, as USER
```

| Question | Answer |
| --- | --- |
| Why does changing one source file re-run `npm install`? | `COPY . .` came before `RUN npm install` — copy `package*.json` first, install, then copy the rest |
| Why can't my container reach port 3000 from the host? | `EXPOSE` is documentation only — publish it with `--publish` |
| Why is my `ARG` value empty inside the running container? | `ARG` only exists at build time — use `ENV` for runtime values |
| Why didn't my `.zip` get extracted by `ADD`? | `ADD` only auto-extracts local tar archives |
| Why does `COPY ../file .` fail? | `COPY` can only read files inside the build context |
| Why is my image so large? | Build tools are still in it — use a [multi-stage build](#-multi-stage-builds) |

---

## 🎯 Quick Reference

| Concept | Purpose | Key Syntax |
| --- | --- | --- |
| **Build** | Build an image from a Dockerfile | `docker build -t user/app:1.0 .` |
| **Build Output** | Show full step output | `--progress=plain` |
| **No Cache** | Rebuild every layer | `--no-cache` |
| **Build Arg** | Pass a build-time value | `--build-arg VERSION=2.0` |
| **FROM** | Base image | `FROM node:18-alpine` |
| **RUN** | Run a command at build time | `RUN npm install` |
| **CMD** | Default startup command | `CMD ["npm", "start"]` |
| **ENTRYPOINT** | Fixed startup executable | `ENTRYPOINT ["node", "server.js"]` |
| **LABEL** | Image metadata | `LABEL version="1.0"` |
| **ADD** | Copy + tar extract / URL | `ADD app.tar.gz /app/` |
| **COPY** | Copy from build context | `COPY . /app` |
| **EXPOSE** | Document a port | `EXPOSE 3000` |
| **ENV** | Build + runtime variable | `ENV NODE_ENV=production` |
| **ARG** | Build-time variable | `ARG VERSION=1.0` |
| **VOLUME** | Declare a mount point | `VOLUME /data` |
| **WORKDIR** | Set the working directory | `WORKDIR /app` |
| **HEALTHCHECK** | Container health test | `HEALTHCHECK CMD curl -f http://localhost/ \|\| exit 1` |
| **USER** | Run as a non-root user | `USER johndoe` |
| **Multi-Stage** | Copy output between stages | `COPY --from=builder /app/main ./` |
| **Login** | Authenticate to Docker Hub | `docker login -u johndoe` |
| **Tag** | Rename for a registry | `docker tag myapp:1.0 johndoe/myapp:1.0` |
| **Push / Pull** | Upload / download an image | `docker push johndoe/myapp:1.0` |

---

## 💡 Best Practices

**✅ Do This**

- **Pin base image tags** — `node:18-alpine` instead of `node:latest`, so builds are reproducible
- **Order instructions from least to most frequently changing** — dependencies first, source code last, so the build cache stays useful
- **Combine related `RUN` commands** with `&&` to reduce layers and avoid stale caches
- **Use `.dockerignore`** to keep `.git`, `node_modules`, and `.env` out of the build context
- **Prefer `COPY` over `ADD`** unless you need tar extraction or a URL source
- **Run as a non-root user** with `USER`
- **Use multi-stage builds** to separate build tools from the runtime image
- **Use small base images** (`alpine`, `slim`) when possible
- **Use the exec form** (`["a", "b"]`) for `CMD` and `ENTRYPOINT`
- **Keep base images updated** and scan images for vulnerabilities regularly
- **Log in with a Personal Access Token**, not your account password

**❌ Avoid This**

- **Relying on `latest`** — it changes silently and breaks reproducible builds
- **Putting secrets in `ENV`, `ARG`, or copied files** — they stay readable in the image layers and history
- **Expecting `EXPOSE` to publish a port** — it's documentation only
- **Using `RUN cd ...`** to change directories — use `WORKDIR`
- **Expecting `ADD` to unzip `.zip` files** — only local tar archives are extracted
- **Shipping compilers and source code** in the final image
- **Copying the whole project before installing dependencies** — every code change then invalidates the dependency cache

> Reference: [Dockerfile reference](https://docs.docker.com/reference/dockerfile/) · [Build best practices](https://docs.docker.com/build/building/best-practices/) · [Multi-stage builds](https://docs.docker.com/build/building/multi-stage/) · [Docker Hub](https://hub.docker.com/)
