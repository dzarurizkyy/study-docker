# 🐳 Docker Basics

A practical reference guide for learning Docker from scratch — covering images, containers, port forwarding, environment variables, storage, networking, and housekeeping.

---

## 📋 Table of Contents

- [Introduction](#-introduction)
  - [What Is Docker?](#what-is-docker)
  - [Core Concepts at a Glance](#core-concepts-at-a-glance)
- [Version & Info](#-version--info)
- [Image Management](#-image-management)
  - [List Images](#list-images)
  - [Pull an Image](#pull-an-image)
  - [Remove an Image](#remove-an-image)
- [Container Management](#-container-management)
  - [List Containers](#list-containers)
  - [Create a Container](#create-a-container)
  - [Start and Stop a Container](#start-and-stop-a-container)
  - [Remove a Container](#remove-a-container)
- [Container Operations](#-container-operations)
  - [Container Logs](#container-logs)
  - [Interactive Terminal](#interactive-terminal)
  - [Container Stats](#container-stats)
- [Port Forwarding](#-port-forwarding)
- [Environment Variables](#-environment-variables)
- [Resource Management](#-resource-management)
- [Bind Mounts](#-bind-mounts)
- [Volume Management](#-volume-management)
  - [List Volumes](#list-volumes)
  - [Create and Remove a Volume](#create-and-remove-a-volume)
  - [Mount a Volume to a Container](#mount-a-volume-to-a-container)
- [Backup & Restore](#-backup--restore)
  - [Manual Backup](#manual-backup)
  - [Automatic Backup](#automatic-backup)
  - [Restore a Backup](#restore-a-backup)
- [Network Management](#-network-management)
  - [List Networks](#list-networks)
  - [Create and Remove a Network](#create-and-remove-a-network)
  - [Network Drivers](#network-drivers)
  - [Attach Containers to a Network](#attach-containers-to-a-network)
- [Utilities](#-utilities)
  - [Inspect](#inspect)
  - [Prune](#prune)
- [Container Lifecycle Flow](#-container-lifecycle-flow)
- [Quick Reference](#-quick-reference)
- [Best Practices](#-best-practices)

---

## 🎯 Introduction

### What Is Docker?

- Docker is a platform for packaging an application together with everything it needs to run — runtime, libraries, and configuration — into a single portable unit called an **image**
- A running instance of an image is a **container**: an isolated process that shares the host's kernel instead of booting a full operating system
- Because the environment travels with the image, an app behaves the same on a laptop, a CI server, and production

> Reference: [docs.docker.com](https://docs.docker.com/)

### Core Concepts at a Glance

| Piece | Role |
| --- | --- |
| **Image** | Read-only template (e.g. `nginx:latest`) — the blueprint a container is created from |
| **Container** | A running (or stopped) instance of an image, with its own writable layer |
| **Registry** | Where images are stored and pulled from — [Docker Hub](https://hub.docker.com/) by default |
| **Volume** | Storage managed by Docker that outlives any single container |
| **Bind Mount** | A folder on the host mapped directly into a container |
| **Network** | A virtual network that lets containers talk to each other |

> **Key Insight:** A container's own filesystem is thrown away when the container is removed. Anything you want to keep — database files, uploads, logs — must live in a [volume](#-volume-management) or a [bind mount](#-bind-mounts).

---

## 🔍 Version & Info

Verify that Docker is installed and that the client can reach the Docker daemon:

```bash
docker version
```

> **Note:** The output has two sections — **Client** and **Server**. If the Server section shows an error, the Docker daemon (or Docker Desktop) isn't running.

---

## 📦 Image Management

### List Images

```bash
docker image ls
```

### Pull an Image

```bash
docker image pull imagename:tag
```

Example:

```bash
docker image pull nginx:latest
docker image pull ubuntu:22.04
```

### Remove an Image

```bash
docker image rm imagename:tag
```

Example:

```bash
docker image rm nginx:latest
```

> **Gotcha:** An image can't be removed while a container — even a stopped one — still uses it. Remove the container first.

---

## 🚀 Container Management

### List Containers

- Display running containers:

  ```bash
  docker container ls
  ```

- Display all containers (including stopped):

  ```bash
  docker container ls -a
  ```

### Create a Container

```bash
docker container create --name containername imagename:tag
```

Example:

```bash
docker container create --name webserver nginx:latest
```

> **Note:** `create` only prepares the container — it doesn't start it. `docker container run` does both in one step (create + start).

### Start and Stop a Container

```bash
docker container start containerid/containername
docker container stop containerid/containername
```

Example:

```bash
docker container start webserver
docker container stop webserver
```

### Remove a Container

```bash
docker container rm containerid/containername
```

Example:

```bash
docker container rm webserver
```

> **Gotcha:** A running container can't be removed — stop it first.

---

## 📊 Container Operations

### Container Logs

- Display container logs:

  ```bash
  docker container logs containerid/containername
  ```

- Follow logs in real time (`-f`):

  ```bash
  docker container logs -f webserver
  ```

- Follow logs with timestamps:

  ```bash
  docker container logs -f --timestamps containername
  ```

### Interactive Terminal

Open a shell inside a running container:

```bash
docker container exec -i -t containername /bin/bash
```

Example:

```bash
docker container exec -i -t webserver /bin/bash
```

| Flag | Meaning |
| --- | --- |
| `-i` | Interactive — keep STDIN open |
| `-t` | Allocate a pseudo-terminal (TTY) |

> **Tip:** Some lightweight images (e.g. Alpine-based) don't ship `bash` — use `/bin/sh` instead.

### Container Stats

- Monitor resource usage of all running containers:

  ```bash
  docker container stats
  ```

- Monitor a specific container:

  ```bash
  docker container stats containername
  ```

---

## 🌐 Port Forwarding

A container runs on its own isolated network, so its ports aren't reachable from the host by default. `--publish` forwards a host port to a container port.

```bash
docker container create \
  --name containername \
  --publish porthost:portcontainer \
  imagename:tag
```

Example:

```bash
docker container create \
  --name webserver \
  --publish 8080:80 \
  nginx:latest
```

> **Key Insight:** The format is always `--publish HOST_PORT:CONTAINER_PORT`. In the example above, opening `http://localhost:8080` on the host reaches Nginx listening on port `80` inside the container.
>
> **Gotcha:** Ports are set when the container is **created**. To change them later, you must remove and recreate the container.

---

## 🔐 Environment Variables

Environment variables configure an app without rebuilding its image — a common pattern for passwords, database names, and feature flags.

```bash
docker container create \
  --name containername \
  --publish porthost:portcontainer \
  --env KEY=VALUE \
  imagename:tag
```

Single variable example:

```bash
docker container create \
  --name mysqldb \
  --env MYSQL_ROOT_PASSWORD=secret \
  mysql:latest
```

Multiple variables example — repeat `--env` once per variable:

```bash
docker container create \
  --name mysqldb \
  --env MYSQL_ROOT_PASSWORD=secret \
  --env MYSQL_DATABASE=myapp \
  --env MYSQL_USER=admin \
  mysql:latest
```

> **Tip:** Each image documents which variables it reads — check its page on Docker Hub (e.g. `mysql`, `postgres`).

---

## ⚡ Resource Management

By default a container can use as much memory and CPU as the host allows. Set limits so one container can't starve the others.

```bash
docker container create \
  --name containername \
  --memory="memorysize" \
  --cpus="cpusize" \
  --publish porthost:portcontainer \
  imagename:tag
```

Example:

```bash
docker container create \
  --name limited-nginx \
  --memory="512m" \
  --cpus="1.0" \
  --publish 8080:80 \
  nginx:latest
```

| Option | Units | Example |
| --- | --- | --- |
| `--memory` | `b`, `k`, `m`, `g` (bytes, kilobytes, megabytes, gigabytes) | `512m`, `2g` |
| `--cpus` | Number of CPUs | `0.5` = 50%, `1.0` = 100%, `2.0` = 200% (two cores) |

---

## 📁 Bind Mounts

A bind mount maps a folder on the host directly into the container. Changes on either side are visible on the other immediately.

```bash
docker container create \
  --name containername \
  --publish porthost:portcontainer \
  --mount "type=bind,source=folder,destination=folder,readonly" \
  imagename:tag
```

| Parameter | Meaning |
| --- | --- |
| `type` | `bind` for a bind mount |
| `source` | Folder on the host (absolute path) |
| `destination` | Folder inside the container |
| `readonly` | Optional — the container can read but not modify the files |

Read-write example:

```bash
docker container create \
  --name webserver \
  --publish 8080:80 \
  --mount "type=bind,source=$(pwd)/html,destination=/usr/share/nginx/html" \
  nginx:latest
```

Read-only example:

```bash
docker container create \
  --name webserver \
  --publish 8080:80 \
  --mount "type=bind,source=$(pwd)/html,destination=/usr/share/nginx/html,readonly" \
  nginx:latest
```

> **Gotcha:** `source` must be an **absolute** path — that's why the examples use `$(pwd)/html` rather than `./html`.

---

## 💾 Volume Management

A volume is storage that Docker creates and manages itself. Unlike a bind mount, you don't pick a host folder — Docker does — which makes volumes the preferred choice for persistent app data.

### List Volumes

```bash
docker volume ls
```

### Create and Remove a Volume

```bash
docker volume create volumename
docker volume rm volumename
```

Example:

```bash
docker volume create mydata
docker volume rm mydata
```

> **Gotcha:** A volume can't be removed while a container still uses it.

### Mount a Volume to a Container

```bash
docker container create \
  --name containername \
  --publish porthost:portcontainer \
  --mount "type=volume,source=volumename,destination=folder" \
  imagename:tag
```

Example:

```bash
docker container create \
  --name webserver \
  --publish 8080:80 \
  --mount "type=volume,source=webdata,destination=/usr/share/nginx/html" \
  nginx:latest
```

> **Note:** The syntax is identical to a bind mount — only `type=volume` and `source` (a volume name instead of a host path) change.

---

## 💿 Backup & Restore

Docker has no built-in "back up this volume" command. The trick is to start a temporary container that mounts **both** the volume and a host folder, then archive one into the other.

```text
Volume (webdata)  →  /data     ┐
                               ├─ temporary container runs tar
Host folder       →  /backup   ┘   /data  →  /backup/backup.tar.gz
```

### Manual Backup

1. Stop the container that uses the volume you want to back up
2. Create a temporary container with two mounts (volume + bind mount):

   ```bash
   docker container run -it \
     --name backup-container \
     --mount "type=bind,source=/path/on/host/backup,destination=/backup" \
     --mount "type=volume,source=my-docker-volume,destination=/data" \
     ubuntu \
     bash
   ```

3. Inside the container, archive the volume contents:

   ```bash
   tar czvf /backup/my-volume-backup.tar.gz /data
   ```

4. Exit the container — the backup file is now in the host folder
5. Remove the temporary container (optional):

   ```bash
   docker container rm backup-container
   ```

### Automatic Backup

**(Recommended)** Do the whole thing in one command. `--rm` deletes the temporary container as soon as `tar` finishes.

```bash
docker container run --rm \
  --name backup-container \
  --mount "type=bind,source=/path/to/backup,destination=/backup" \
  --mount "type=volume,source=volumename,destination=/data" \
  ubuntu \
  tar czvf /backup/backup-data.tar.gz /data
```

Example:

```bash
docker container run --rm \
  --name backup-webdata \
  --mount "type=bind,source=$(pwd)/backup,destination=/backup" \
  --mount "type=volume,source=webdata,destination=/data" \
  ubuntu \
  tar czvf /backup/webdata-backup.tar.gz /data
```

### Restore a Backup

Same idea in reverse — mount the backup folder and the target volume, then extract.

```bash
docker container run --rm \
  --name restore-container \
  --mount "type=bind,source=/path/to/backup,destination=/backup" \
  --mount "type=volume,source=volumename,destination=/data" \
  ubuntu \
  bash -c "tar xzvf /backup/backup-data.tar.gz -C /"
```

Example:

```bash
docker container run --rm \
  --name restore-webdata \
  --mount "type=bind,source=$(pwd)/backup,destination=/backup" \
  --mount "type=volume,source=webdata,destination=/data" \
  ubuntu \
  bash -c "tar xzvf /backup/webdata-backup.tar.gz -C /"
```

> **Note:** `tar` stores the archived path as `data/...` (without the leading `/`), so extracting with `-C /` puts the files back into `/data` — which is the mounted volume.

---

## 🔗 Network Management

Containers on the same user-defined network can reach each other **by container name** — e.g. an app container connects to `mysqldb:3306` instead of an IP address.

### List Networks

```bash
docker network ls
```

### Create and Remove a Network

```bash
docker network create --driver drivername networkname
docker network rm networkname
```

Example:

```bash
docker network create --driver bridge appnet
docker network rm appnet
```

### Network Drivers

| Driver | Description |
| --- | --- |
| `bridge` | Default driver — an isolated network on a single host |
| `host` | Removes network isolation — the container uses the host's network directly |
| `none` | Disables networking entirely |

### Attach Containers to a Network

- Create a container inside a network:

  ```bash
  docker container create \
    --name containername \
    --network networkname \
    imagename:tag
  ```

  Example:

  ```bash
  docker container create \
    --name webserver \
    --network appnet \
    nginx:latest
  ```

- Disconnect a container from a network:

  ```bash
  docker network disconnect networkname containername
  ```

  Example:

  ```bash
  docker network disconnect appnet webserver
  ```

- Connect an existing container to a network:

  ```bash
  docker network connect networkname containername
  ```

  Example:

  ```bash
  docker network connect appnet webserver
  ```

> **Gotcha:** A network can't be removed while containers are still connected to it — disconnect or remove them first.

---

## 🛠️ Utilities

### Inspect

Show detailed information (as JSON) about any Docker object:

```bash
docker inspect name
```

Examples:

```bash
docker inspect webserver          # Container info
docker inspect nginx:latest       # Image info
docker inspect webdata            # Volume info
docker inspect appnet             # Network info
```

> **Tip:** If a container and a volume share the same name, be explicit: `docker container inspect`, `docker volume inspect`, and so on.

### Prune

- Delete unused data from a specific feature:

  ```bash
  docker dockerfeature prune
  ```

  Examples:

  ```bash
  docker image prune               # Remove unused (dangling) images
  docker container prune           # Remove stopped containers
  docker volume prune              # Remove unused volumes
  docker network prune             # Remove unused networks
  ```

- Delete all unused Docker data:

  ```bash
  docker system prune
  ```

- Remove everything, including volumes:

  ```bash
  docker system prune -a --volumes
  ```

| Flag | Meaning |
| --- | --- |
| `-a` / `--all` | Remove all unused images, not just dangling ones |
| `--volumes` | Also remove unused volumes |
| `-f` / `--force` | Skip the confirmation prompt |

> **Gotcha:** `--volumes` permanently deletes data in every volume not attached to a container. [Back up](#-backup--restore) anything important first.

Weekly cleanup routine:

```bash
docker system prune -f
docker image prune -a -f
docker volume prune -f
```

---

## 🔄 Container Lifecycle Flow

How the commands in this guide fit together, from pulling an image to cleaning up:

```text
Registry (Docker Hub)
  ↓  docker image pull
Image                             read-only template
  ↓  docker container create      ports, env, limits, mounts, network are fixed here
Container (created)
  ↓  docker container start
Container (running)               logs · exec · stats
  ↓  docker container stop
Container (stopped)
  ↓  docker container rm
Removed                           container's own filesystem is gone —
                                  data survives only in volumes / bind mounts
```

| Question | Answer |
| --- | --- |
| Why did my data disappear after removing a container? | It lived in the container's own layer — store it in a [volume](#-volume-management) or [bind mount](#-bind-mounts) |
| Why can't I reach my app on `localhost`? | The port isn't published — add `--publish HOST:CONTAINER` at create time |
| Why can't I change ports or env on an existing container? | They're fixed at `create` — remove and recreate the container |
| Why does `docker image rm` fail? | A container (even a stopped one) still uses the image |
| How do containers find each other? | Put them on the same user-defined [network](#-network-management) and use the container name |

---

## 🎯 Quick Reference

| Concept | Purpose | Key Syntax |
| --- | --- | --- |
| **Version** | Show Docker client/server version | `docker version` |
| **List Images** | Show downloaded images | `docker image ls` |
| **Pull Image** | Download an image | `docker image pull nginx:latest` |
| **Remove Image** | Delete an image | `docker image rm nginx:latest` |
| **List Containers** | Running / all containers | `docker container ls` / `ls -a` |
| **Create Container** | Prepare a container | `docker container create --name web nginx` |
| **Start / Stop** | Control a container | `docker container start web` |
| **Remove Container** | Delete a container | `docker container rm web` |
| **Logs** | View (or follow) output | `docker container logs -f web` |
| **Exec** | Run a command inside | `docker container exec -it web /bin/bash` |
| **Stats** | Live resource usage | `docker container stats` |
| **Port Forwarding** | Expose a container port | `--publish 8080:80` |
| **Environment Variable** | Configure the app | `--env KEY=VALUE` |
| **Resource Limit** | Cap memory / CPU | `--memory="512m" --cpus="1.0"` |
| **Bind Mount** | Map a host folder | `--mount "type=bind,source=...,destination=..."` |
| **Volume** | Docker-managed storage | `docker volume create mydata` |
| **Volume Mount** | Attach a volume | `--mount "type=volume,source=...,destination=..."` |
| **Network** | Connect containers | `docker network create --driver bridge appnet` |
| **Connect / Disconnect** | Change a container's network | `docker network connect appnet web` |
| **Inspect** | Detailed JSON info | `docker inspect web` |
| **Prune** | Clean up unused data | `docker system prune` |

---

## 💡 Best Practices

**✅ Do This**

- **Use descriptive container names** — `webserver`, `database`, `cache`; use hyphens for multi-word names (`my-web-server`) and avoid special characters
- **Always pin image tags** — `nginx:1.21` instead of `nginx:latest`, especially in production
- **Store persistent data in volumes** — the container's own filesystem disappears with it
- **Use read-only mounts** when the container doesn't need to modify the data
- **Limit container resources** with `--memory` and `--cpus` so one container can't starve the host
- **Put related containers on a user-defined network** and connect them by name
- **Back up volumes** before pruning or upgrading
- **Keep images updated** to pick up security fixes
- **Clean up regularly** with the prune commands

**❌ Avoid This**

- **Relying on `latest`** — it doesn't mean "newest", it's just the default tag
- **Running containers as root** when it isn't required
- **Keeping important data only inside a container** — it's lost on `docker container rm`
- **Running `docker system prune -a --volumes` casually** — it permanently deletes unused volumes
- **Hard-coding container IP addresses** — they change; use container names on a shared network

> Reference: [Docker Docs](https://docs.docker.com/) · [Docker CLI reference](https://docs.docker.com/reference/cli/docker/) · [Docker Hub](https://hub.docker.com/)
