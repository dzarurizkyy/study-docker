# 🐳 Docker Commands Cheatsheet

A comprehensive reference guide for Docker commands and operations.

---

## 📋 Table of Contents

- [Version & Info](#-version--info)
- [Image Management](#-image-management)
- [Container Management](#-container-management)
- [Container Operations](#-container-operations)
- [Port Forwarding](#-port-forwarding)
- [Environment Variables](#-environment-variables)
- [Resource Management](#-resource-management)
- [Bind Mounts](#-bind-mounts)
- [Volume Management](#-volume-management)
- [Backup & Restore](#-backup--restore)
- [Network Management](#-network-management)
- [Utilities](#-utilities)

---

## 🔍 Version & Info

- **Display Docker version**
  
  ```bash
  docker version
  ```

---

## 📦 Image Management

- **Display list of images**
  
  ```bash
  docker image ls
  ```

- **Download Docker image**
  
  ```bash
  docker image pull imagename:tag
  ```
  
  Example:
  
  ```bash
  docker image pull nginx:latest
  docker image pull ubuntu:22.04
  ```

- **Delete Docker image**
  
  ```bash
  docker image rm imagename:tag
  ```
  
  Example:
  
  ```bash
  docker image rm nginx:latest
  ```

---

## 🚀 Container Management

- **Display running containers**
  
  ```bash
  docker container ls
  ```

- **Display all containers (including stopped)**
  
  ```bash
  docker container ls -a
  ```

- **Create Docker container**
  
  ```bash
  docker container create --name containername imagename:tag
  ```
  
  Example:
  
  ```bash
  docker container create --name webserver nginx:latest
  ```

- **Start Docker container**
  
  ```bash
  docker container start containerid/containername
  ```
  
  Example:
  
  ```bash
  docker container start webserver
  ```

- **Stop Docker container**
  
  ```bash
  docker container stop containerid/containername
  ```
  
  Example:
  
  ```bash
  docker container stop webserver
  ```

- **Remove Docker container**
  
  ```bash
  docker container rm containerid/containername
  ```
  
  Example:
  
  ```bash
  docker container rm webserver
  ```

---

## 📊 Container Operations

- **Display container logs**
  
  ```bash
  docker container logs containerid/containername
  ```

- **Display container logs in real-time**
  
  ```bash
  docker container logs -f containerid/containername
  ```
  
  Example:
  
  ```bash
  docker container logs -f webserver
  ```

- **Execute interactive terminal in container**
  
  ```bash
  docker container exec -i -t containername /bin/bash
  ```
  
  Example:
  
  ```bash
  docker container exec -i -t webserver /bin/bash
  ```

- **Display statistics of running containers**
  
  ```bash
  docker container stats
  ```

---

## 🌐 Port Forwarding

- **Forward ports from host to container**
  
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
  
  Format: `--publish HOST_PORT:CONTAINER_PORT`

---

## 🔐 Environment Variables

- **Add environment variables to container**
  
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
  
  Multiple variables example:
  
  ```bash
  docker container create \
    --name mysqldb \
    --env MYSQL_ROOT_PASSWORD=secret \
    --env MYSQL_DATABASE=myapp \
    --env MYSQL_USER=admin \
    mysql:latest
  ```

---

## ⚡ Resource Management

- **Set container resource limits**
  
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
  
  Memory units: `b`, `k`, `m`, `g` (bytes, kilobytes, megabytes, gigabytes)
  
  CPU units: Number of CPUs (e.g., `0.5` = 50%, `1.0` = 100%, `2.0` = 200%)

---

## 📁 Bind Mounts

- **Mount host folder to container**
  
  ```bash
  docker container create \
    --name containername \
    --publish porthost:portcontainer \
    --mount "type=bind,source=folder,destination=folder,readonly" \
    imagename:tag
  ```
  
  Read-Write example:
  
  ```bash
  docker container create \
    --name webserver \
    --publish 8080:80 \
    --mount "type=bind,source=$(pwd)/html,destination=/usr/share/nginx/html" \
    nginx:latest
  ```
  
  Read-Only example:
  
  ```bash
  docker container create \
    --name webserver \
    --publish 8080:80 \
    --mount "type=bind,source=$(pwd)/html,destination=/usr/share/nginx/html,readonly" \
    nginx:latest
  ```

---

## 💾 Volume Management

- **Display list of volumes**
  
  ```bash
  docker volume ls
  ```

- **Create Docker volume**
  
  ```bash
  docker volume create volumename
  ```
  
  Example:
  
  ```bash
  docker volume create mydata
  ```

- **Delete Docker volume**
  
  ```bash
  docker volume rm volumename
  ```
  
  Example:
  
  ```bash
  docker volume rm mydata
  ```

- **Mount volume to container**
  
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

---

## 💿 Backup & Restore

- **Backup data from Docker volume**
  
  Process:
  - Stop container that is using the volume you want to back up
  - Create temporary container with two mounts (volume + bind mount)
  - Run backup command inside container by archiving the volume contents
  - Backup file is now available in host folder
  - Remove temporary container (optional)
  
  Manual backup:
  
  ```bash
  docker container create \
    --name backup-container \
    --mount "type=bind,source=/path/on/host/backup,destination=/backup" \
    --mount "type=volume,source=my-docker-volume,destination=/data" \
    imagename:tag
  ```
  
  ```bash
  tar cvf /backup/my-volume-backup.tar.gz /data
  ```
  
  Automatic backup (recommended):
  
  ```bash
  docker container run --rm \
    --name backup-container \
    --mount "type=bind,source=/path/to/backup,destination=/backup" \
    --mount "type=volume,source=volumename,destination=/data" \
    ubuntu \
    tar cvf /backup/backup-data.tar.gz /data
  ```
  
  Example:
  
  ```bash
  docker container run --rm \
    --name backup-webdata \
    --mount "type=bind,source=$(pwd)/backup,destination=/backup" \
    --mount "type=volume,source=webdata,destination=/data" \
    ubuntu \
    tar cvf /backup/webdata-backup.tar.gz /data
  ```

- **Restore backup data to Docker volume**
  
  ```bash
  docker container run --rm \
    --name restore-container \
    --mount "type=bind,source=/path/to/backup,destination=/backup" \
    --mount "type=volume,source=volumename,destination=/data" \
    imagename:tag \
    bash -c "tar xvf /backup/backup-data.tar.gz"
  ```
  
  Example:
  
  ```bash
  docker container run --rm \
    --name restore-webdata \
    --mount "type=bind,source=$(pwd)/backup,destination=/backup" \
    --mount "type=volume,source=webdata,destination=/data" \
    ubuntu \
    bash -c "tar xvf /backup/webdata-backup.tar.gz"
  ```

---

## 🌐 Network Management

- **Display list of networks**
  
  ```bash
  docker network ls
  ```

- **Create Docker network**
  
  ```bash
  docker network create --driver drivername networkname
  ```
  
  Example:
  
  ```bash
  docker network create --driver bridge appnet
  ```
  
  Common drivers:
  - `bridge` - Default network driver (isolated network)
  - `host` - Remove network isolation (use host network)
  - `none` - Disable networking

- **Delete Docker network**
  
  ```bash
  docker network rm networkname
  ```
  
  Example:
  
  ```bash
  docker network rm appnet
  ```

- **Create container with network**
  
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

- **Disconnect container from network**
  
  ```bash
  docker network disconnect networkname containername
  ```
  
  Example:
  
  ```bash
  docker network disconnect appnet webserver
  ```

- **Connect container to network**
  
  ```bash
  docker network connect networkname containername
  ```
  
  Example:
  
  ```bash
  docker network connect appnet webserver
  ```

---

## 🛠️ Utilities

- **Get detailed information from Docker feature**
  
  ```bash
  docker inspect dockerfeature name
  ```
  
  Examples:
  
  ```bash
  docker inspect webserver          # Container info
  docker inspect nginx:latest       # Image info
  docker inspect webdata            # Volume info
  docker inspect appnet             # Network info
  ```

- **Delete unused Docker data from specific feature**
  
  ```bash
  docker dockerfeature prune
  ```
  
  Examples:
  
  ```bash
  docker image prune               # Remove unused images
  docker container prune           # Remove stopped containers
  docker volume prune              # Remove unused volumes
  docker network prune             # Remove unused networks
  ```

- **Delete all unused Docker data**
  
  ```bash
  docker system prune
  ```
  
  Remove everything (including volumes):
  
  ```bash
  docker system prune -a --volumes
  ```
  
  Flags:
  - `-a` or `--all` - Remove all unused images, not just dangling ones
  - `--volumes` - Remove all unused volumes

---

## 🎯 Quick Reference Table

| Command | Description |
|---------|-------------|
| `docker version` | Show Docker version |
| `docker image ls` | List images |
| `docker image pull` | Download image |
| `docker image rm` | Delete image |
| `docker container ls` | List running containers |
| `docker container ls -a` | List all containers |
| `docker container create` | Create container |
| `docker container start` | Start container |
| `docker container stop` | Stop container |
| `docker container rm` | Remove container |
| `docker container logs` | View logs |
| `docker container logs -f` | View logs (real-time) |
| `docker container exec` | Execute command in container |
| `docker container stats` | View resource usage |
| `docker volume ls` | List volumes |
| `docker volume create` | Create volume |
| `docker volume rm` | Delete volume |
| `docker network ls` | List networks |
| `docker network create` | Create network |
| `docker network rm` | Delete network |
| `docker inspect` | View detailed info |
| `docker system prune` | Clean up unused data |

---

## 💡 Tips & Best Practices

- **Container Naming**
  - Use descriptive names: `webserver`, `database`, `cache`
  - Use hyphens for multi-word names: `my-web-server`
  - Avoid special characters

- **Image Tags**
  - Always specify tags: `nginx:1.21` instead of `nginx:latest`
  - `latest` doesn't mean "newest", it's just a default tag
  - Use specific versions for production

- **Regular Cleanup**
  
  Weekly cleanup routine:
  
  ```bash
  docker system prune -f
  docker image prune -a -f
  docker volume prune -f
  ```

- **Security**
  - Don't run containers as root when possible
  - Use read-only mounts when data doesn't need to be modified
  - Limit container resources to prevent DoS
  - Keep images updated

- **Monitoring**
  
  Monitor all containers:
  
  ```bash
  docker container stats
  ```
  
  Monitor specific container:
  
  ```bash
  docker container stats containername
  ```
  
  View logs with timestamps:
  
  ```bash
  docker container logs -f --timestamps containername
  ```
