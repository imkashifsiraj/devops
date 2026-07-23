# Month 2 - Part 1: Docker & Containerization

> **DevOps Upskilling Program — GCC Job Market Focus**
> Comprehensive study material covering containers, Docker architecture, images, networking, volumes, Compose, security, and best practices.

---

## Table of Contents

1. [What Are Containers?](#1-what-are-containers)
2. [Docker Architecture](#2-docker-architecture)
3. [Docker Installation](#3-docker-installation)
4. [Docker Images](#4-docker-images)
5. [Docker Commands Reference](#5-docker-commands-reference)
6. [Docker Networking Deep Dive](#6-docker-networking-deep-dive)
7. [Docker Volumes Deep Dive](#7-docker-volumes-deep-dive)
8. [Docker Compose](#8-docker-compose)
9. [Docker Security Best Practices](#9-docker-security-best-practices)
10. [Docker Registry](#10-docker-registry)
11. [Troubleshooting Docker](#11-troubleshooting-docker)
12. [Lessons Learned / Pro Tips](#12-lessons-learned--pro-tips)
13. [Interview Questions](#13-interview-questions)
14. [Free Resources](#14-free-resources)

---

## 1. What Are Containers?

### Definition

A **container** is a lightweight, standalone, executable package that includes everything needed to run a piece of software: code, runtime, system tools, libraries, and settings. Containers use OS-level virtualization to isolate processes.

### Containers vs Virtual Machines

```
┌─────────────────────────────────────────────────────────────────┐
│              CONTAINERS vs VIRTUAL MACHINES                       │
├────────────────────────────┬────────────────────────────────────┤
│        CONTAINERS          │        VIRTUAL MACHINES            │
├────────────────────────────┼────────────────────────────────────┤
│  ┌─────┐ ┌─────┐ ┌─────┐  │  ┌─────────┐ ┌─────────┐         │
│  │App A│ │App B│ │App C│  │  │  App A  │ │  App B  │         │
│  ├─────┤ ├─────┤ ├─────┤  │  ├─────────┤ ├─────────┤         │
│  │Bins/│ │Bins/│ │Bins/│  │  │ Bins/   │ │ Bins/   │         │
│  │Libs │ │Libs │ │Libs │  │  │ Libs    │ │ Libs    │         │
│  └──┬──┘ └──┬──┘ └──┬──┘  │  ├─────────┤ ├─────────┤         │
│     │       │       │      │  │Guest OS │ │Guest OS │         │
│  ┌──┴───────┴───────┴──┐  │  └────┬────┘ └────┬────┘         │
│  │   Container Runtime  │  │       │           │              │
│  ├──────────────────────┤  │  ┌────┴───────────┴────┐         │
│  │      Host OS         │  │  │     Hypervisor      │         │
│  ├──────────────────────┤  │  ├─────────────────────┤         │
│  │    Infrastructure    │  │  │      Host OS        │         │
│  └──────────────────────┘  │  ├─────────────────────┤         │
│                            │  │   Infrastructure    │         │
│                            │  └─────────────────────┘         │
└────────────────────────────┴────────────────────────────────────┘
```

| Feature | Containers | Virtual Machines |
|---------|-----------|-----------------|
| **Boot time** | Seconds | Minutes |
| **Size** | MBs (10-500 MB) | GBs (1-50 GB) |
| **Performance** | Near-native | ~5-20% overhead |
| **Isolation** | Process-level (namespaces) | Hardware-level (hypervisor) |
| **OS** | Shares host kernel | Full guest OS |
| **Density** | 100s per host | 10s per host |
| **Portability** | Very high | Medium |
| **Use case** | Microservices, CI/CD | Legacy apps, different OS kernels |

### Key Container Technologies

- **Namespaces** — Isolate processes (PID, NET, MNT, UTS, IPC, USER)
- **cgroups** — Limit and account for resource usage (CPU, memory, I/O)
- **Union filesystems** — Layer filesystem for efficient storage (OverlayFS)

---

## 2. Docker Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Docker Architecture                    │
├──────────┐          ┌──────────┐         ┌─────────────┤
│  Docker  │  REST    │  Docker  │         │  Registry   │
│  Client  │◄────────►│  Daemon  │◄───────►│ (Docker Hub)│
│  (CLI)   │  API     │ (dockerd)│  Pull/  │             │
└──────────┘          └─────┬────┘  Push   └─────────────┘
                            │
                      ┌─────┴─────┐
                      │containerd │
                      └─────┬─────┘
                            │
                      ┌─────┴─────┐
                      │   runc    │
                      └─────┬─────┘
                            │
                   ┌────────┴────────┐
                   │   Container(s)  │
                   └─────────────────┘
```

### Components Explained

| Component | Role |
|-----------|------|
| **Docker Client** | CLI tool (`docker` command) that sends commands to the daemon |
| **Docker Daemon (dockerd)** | Background service managing images, containers, networks, volumes |
| **containerd** | Industry-standard container runtime managing container lifecycle |
| **runc** | Low-level OCI runtime that creates and runs containers |
| **Images** | Read-only templates with instructions for creating containers |
| **Containers** | Running instances of images (writable layer on top) |
| **Registry** | Storage and distribution system for Docker images |

---

## 3. Docker Installation

### Ubuntu/Debian

```bash
# Remove old versions
sudo apt-get remove docker docker-engine docker.io containerd runc

# Install prerequisites
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg lsb-release

# Add Docker's official GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Set up repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Post-install: run without sudo
sudo usermod -aG docker $USER
newgrp docker

# Verify installation
docker --version
docker run hello-world
```

### CentOS/RHEL

```bash
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl start docker
sudo systemctl enable docker
```

### Verify Installation

```bash
docker version          # Client and server version info
docker info             # System-wide information
docker system info      # Same as above
```

---

## 4. Docker Images

### 4.1 What Are Layers?

Every instruction in a Dockerfile creates a **layer**. Layers are cached and reused, making builds faster and images smaller.

```
┌─────────────────────────┐
│   Writable Container    │  ← Container layer (read-write)
├─────────────────────────┤
│   CMD ["node", "app"]   │  ← Layer 5
├─────────────────────────┤
│   COPY . /app           │  ← Layer 4
├─────────────────────────┤
│   RUN npm install       │  ← Layer 3
├─────────────────────────┤
│   WORKDIR /app          │  ← Layer 2
├─────────────────────────┤
│   FROM node:18-alpine   │  ← Base layer (Layer 1)
└─────────────────────────┘
```

**Key points:**
- Layers are read-only once created
- Only the top container layer is writable
- Layers are shared between images (saves disk space)
- Changing a layer invalidates all layers above it (cache busting)

### 4.2 Dockerfile Instructions (Complete Reference)

#### FROM — Base image

```dockerfile
# Basic usage
FROM ubuntu:22.04

# Multi-stage naming
FROM node:18-alpine AS builder

# Scratch (empty) image for minimal containers
FROM scratch
```

#### WORKDIR — Set working directory

```dockerfile
WORKDIR /app
# All subsequent commands run from /app
# Creates directory if it doesn't exist
```

#### COPY — Copy files from build context

```dockerfile
# Copy single file
COPY package.json .

# Copy directory contents
COPY src/ ./src/

# Copy with ownership (avoids extra RUN chown)
COPY --chown=node:node . .

# Copy from a build stage
COPY --from=builder /app/dist ./dist
```

#### ADD — Copy with extra features

```dockerfile
# Auto-extract tar archives
ADD archive.tar.gz /app/

# Download from URL (not recommended — use curl in RUN instead)
ADD https://example.com/file.txt /app/

# Same as COPY for regular files
ADD package.json .
```

> **Best Practice:** Use `COPY` unless you need tar extraction. `ADD` is less transparent.

#### RUN — Execute commands during build

```dockerfile
# Shell form (runs in /bin/sh -c)
RUN apt-get update && apt-get install -y curl

# Exec form
RUN ["apt-get", "install", "-y", "curl"]

# Multi-line with cleanup (reduces layer size)
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      curl \
      wget \
      git && \
    rm -rf /var/lib/apt/lists/*
```

#### ENV — Set environment variables

```dockerfile
# Single variable
ENV NODE_ENV=production

# Multiple (legacy syntax)
ENV APP_HOME=/app \
    APP_PORT=3000

# Available at build time AND runtime
```

#### ARG — Build-time variables

```dockerfile
# Define with default
ARG NODE_VERSION=18

# Use in FROM
ARG NODE_VERSION=18
FROM node:${NODE_VERSION}-alpine

# Override at build time:
# docker build --build-arg NODE_VERSION=20 .

# ARG is NOT available at runtime (unlike ENV)
ARG BUILD_DATE
ENV BUILD_DATE=${BUILD_DATE}  # Promote to runtime
```

#### EXPOSE — Document ports

```dockerfile
# Document which ports the container listens on
EXPOSE 3000
EXPOSE 8080/tcp
EXPOSE 8443/udp

# This is DOCUMENTATION only — doesn't actually publish ports
# You still need -p flag at runtime: docker run -p 3000:3000
```

#### VOLUME — Create mount point

```dockerfile
# Create anonymous volume mount point
VOLUME /data
VOLUME ["/data", "/logs"]

# Data written here survives container removal
# Better to use named volumes at runtime:
# docker run -v mydata:/data
```

#### USER — Set runtime user

```dockerfile
# Create non-root user and switch
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# All subsequent RUN, CMD, ENTRYPOINT run as this user
```

#### HEALTHCHECK — Container health monitoring

```dockerfile
# HTTP health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# Simple process check
HEALTHCHECK CMD pgrep nginx || exit 1

# Disable inherited healthcheck
HEALTHCHECK NONE
```

#### ENTRYPOINT — Container executable

```dockerfile
# Exec form (preferred)
ENTRYPOINT ["node", "server.js"]

# Shell form
ENTRYPOINT node server.js

# Combined with CMD for default arguments
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "8000"]
# Result: python app.py --port 8000
# Override args: docker run myimage --port 9000
# Result: python app.py --port 9000
```

#### CMD — Default command/arguments

```dockerfile
# Exec form (preferred)
CMD ["node", "app.js"]

# Shell form
CMD node app.js

# As default parameters to ENTRYPOINT
CMD ["--help"]
```

### 4.3 ENTRYPOINT vs CMD (Detailed)

| Aspect | ENTRYPOINT | CMD |
|--------|-----------|-----|
| **Purpose** | Define the executable | Default arguments |
| **Override** | `--entrypoint` flag | Any args after image name |
| **Combination** | Fixed part of command | Variable/default part |
| **Use when** | Container IS a command | Container has a default |

```dockerfile
# Example 1: CMD only (easy to override entirely)
FROM python:3.11-slim
CMD ["python", "--version"]
# docker run myimage              → python --version
# docker run myimage python app.py → python app.py (CMD replaced)

# Example 2: ENTRYPOINT only (fixed command)
FROM python:3.11-slim
ENTRYPOINT ["python"]
# docker run myimage              → python (no args)
# docker run myimage app.py       → python app.py

# Example 3: Both (best pattern for CLI tools)
FROM python:3.11-slim
ENTRYPOINT ["python", "app.py"]
CMD ["--host", "0.0.0.0", "--port", "8000"]
# docker run myimage              → python app.py --host 0.0.0.0 --port 8000
# docker run myimage --port 9000  → python app.py --port 9000
```

### 4.4 Multi-Stage Builds

#### Node.js Example

```dockerfile
# ============ Stage 1: Build ============
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && \
    cp -R node_modules prod_node_modules && \
    npm ci
COPY . .
RUN npm run build

# ============ Stage 2: Production ============
FROM node:18-alpine AS production
WORKDIR /app
ENV NODE_ENV=production

# Copy only production dependencies and built files
COPY --from=builder /app/prod_node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package.json .

RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s CMD wget -q --spider http://localhost:3000/health || exit 1
CMD ["node", "dist/server.js"]
```

#### Python Example

```dockerfile
# ============ Stage 1: Build ============
FROM python:3.11-slim AS builder
WORKDIR /app
RUN pip install --no-cache-dir poetry
COPY pyproject.toml poetry.lock ./
RUN poetry export -f requirements.txt -o requirements.txt --without-hashes
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt
COPY . .

# ============ Stage 2: Production ============
FROM python:3.11-slim AS production
WORKDIR /app

COPY --from=builder /install /usr/local
COPY --from=builder /app .

RUN adduser --disabled-password --gecos "" appuser
USER appuser

EXPOSE 8000
HEALTHCHECK --interval=30s --timeout=5s CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')" || exit 1
CMD ["gunicorn", "app:app", "--bind", "0.0.0.0:8000", "--workers", "4"]
```

### 4.5 Image Optimization Best Practices

| Technique | Before | After | Savings |
|-----------|--------|-------|---------|
| Use Alpine base | `node:18` (950MB) | `node:18-alpine` (175MB) | ~82% |
| Multi-stage build | 1.2GB (with devDeps) | 180MB (prod only) | ~85% |
| Combine RUN layers | 450MB (3 layers) | 380MB (1 layer) | ~16% |
| Remove cache files | 250MB | 190MB | ~24% |
| Use .dockerignore | 500MB context | 50MB context | ~90% |

**Optimization checklist:**
1. Use minimal base images (`alpine`, `slim`, `distroless`)
2. Multi-stage builds to exclude build tools
3. Combine and clean up in the same `RUN` layer
4. Order Dockerfile for maximum cache hits (least-changing first)
5. Use `.dockerignore` to reduce build context
6. Pin versions for reproducibility
7. Don't install unnecessary packages

### 4.6 .dockerignore

```dockerignore
# Version control
.git
.gitignore

# Dependencies (will be installed in container)
node_modules
__pycache__
*.pyc
.venv

# Build outputs
dist
build
*.egg-info

# IDE and OS files
.vscode
.idea
*.swp
.DS_Store
Thumbs.db

# Docker files
Dockerfile*
docker-compose*
.dockerignore

# Environment and secrets
.env
.env.*
*.pem
*.key

# Documentation
README.md
docs/
*.md

# Tests
tests/
coverage/
.pytest_cache
```

---

## 5. Docker Commands Reference

### 5.1 Image Commands

```bash
# Build an image from Dockerfile
docker build -t myapp:1.0 .
docker build -t myapp:1.0 -f Dockerfile.prod .
docker build --no-cache -t myapp:1.0 .
docker build --build-arg VERSION=2.0 -t myapp:2.0 .
docker build --platform linux/amd64,linux/arm64 -t myapp:1.0 .

# Pull image from registry
docker pull nginx:latest
docker pull nginx:1.25-alpine
docker pull myregistry.com/myapp:1.0

# Push image to registry
docker push myregistry.com/myapp:1.0
docker push myregistry.com/myapp:latest

# Tag an image
docker tag myapp:1.0 myregistry.com/myapp:1.0
docker tag myapp:1.0 myapp:latest

# Remove image(s)
docker rmi nginx:latest
docker rmi -f $(docker images -q)  # Remove ALL images (force)

# List images
docker images
docker images --filter "dangling=true"
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"

# Inspect image metadata
docker inspect nginx:latest
docker inspect --format '{{.Config.Env}}' myapp:1.0

# View image layer history
docker history nginx:latest
docker history --no-trunc myapp:1.0

# Remove unused images
docker image prune          # Remove dangling images
docker image prune -a       # Remove ALL unused images
docker image prune -a --filter "until=24h"
```

### 5.2 Container Commands

```bash
# Run a container (comprehensive flags)
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

# Common run flags:
docker run -d nginx                          # Detached mode (background)
docker run -it ubuntu bash                   # Interactive + TTY
docker run --name my-nginx nginx             # Custom name
docker run -p 8080:80 nginx                  # Port mapping host:container
docker run -p 127.0.0.1:8080:80 nginx       # Bind to specific interface
docker run -P nginx                          # Publish all exposed ports
docker run -v mydata:/data nginx             # Named volume
docker run -v $(pwd):/app node               # Bind mount
docker run --tmpfs /tmp nginx                # tmpfs mount
docker run -e NODE_ENV=production node       # Environment variable
docker run --env-file .env node              # Env file
docker run -w /app node                      # Working directory
docker run --network mynet nginx             # Attach to network
docker run --restart=always nginx            # Restart policy (no|on-failure|always|unless-stopped)
docker run --rm nginx echo "hello"           # Remove after exit
docker run --memory=512m nginx               # Memory limit
docker run --cpus=1.5 nginx                  # CPU limit
docker run --user 1000:1000 nginx            # Run as specific user
docker run --read-only nginx                 # Read-only filesystem
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE nginx  # Linux capabilities
docker run --health-cmd="curl -f http://localhost/" nginx    # Healthcheck
docker run --log-driver=json-file --log-opt max-size=10m nginx  # Log options
docker run --pid=host nginx                  # Share host PID namespace
docker run --privileged nginx                # Full host privileges (DANGEROUS)
docker run --init nginx                      # Use tini as PID 1

# List containers
docker ps                    # Running containers
docker ps -a                 # All containers (including stopped)
docker ps -q                 # Only IDs
docker ps --filter status=exited
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# Stop containers
docker stop my-nginx         # Graceful stop (SIGTERM, then SIGKILL after 10s)
docker stop -t 30 my-nginx   # Custom timeout before SIGKILL

# Kill containers
docker kill my-nginx         # Immediate SIGKILL
docker kill -s SIGUSR1 my-nginx  # Send specific signal

# Remove containers
docker rm my-nginx           # Remove stopped container
docker rm -f my-nginx        # Force remove (even if running)
docker rm $(docker ps -aq)   # Remove all stopped containers
docker container prune       # Remove all stopped containers

# Execute command in running container
docker exec -it my-nginx bash           # Interactive shell
docker exec my-nginx cat /etc/nginx/nginx.conf  # Run command
docker exec -u root my-nginx whoami     # Execute as specific user
docker exec -w /app my-nginx ls         # Specify working directory

# View container logs
docker logs my-nginx                    # All logs
docker logs -f my-nginx                 # Follow (tail -f)
docker logs --tail 100 my-nginx         # Last 100 lines
docker logs --since 2024-01-01 my-nginx # Since timestamp
docker logs --timestamps my-nginx       # Show timestamps

# Inspect container
docker inspect my-nginx
docker inspect --format '{{.NetworkSettings.IPAddress}}' my-nginx
docker inspect --format '{{json .Config.Env}}' my-nginx

# Copy files between host and container
docker cp myfile.txt my-nginx:/tmp/     # Host → Container
docker cp my-nginx:/var/log/nginx/ ./logs/  # Container → Host

# Container resource statistics
docker stats                            # Live stats (all containers)
docker stats my-nginx                   # Specific container
docker stats --no-stream                # Single snapshot

# Container processes
docker top my-nginx                     # Running processes in container
```

### 5.3 Volume Commands

```bash
# Create a named volume
docker volume create mydata
docker volume create --driver local --opt type=nfs --opt o=addr=192.168.1.1,rw --opt device=:/path/to/dir nfs-data

# List volumes
docker volume ls
docker volume ls --filter dangling=true

# Remove volume
docker volume rm mydata

# Inspect volume
docker volume inspect mydata

# Remove all unused volumes
docker volume prune
docker volume prune -f   # Skip confirmation
```

### 5.4 Network Commands

```bash
# Create network
docker network create mynet
docker network create --driver bridge --subnet 172.20.0.0/16 --gateway 172.20.0.1 mynet
docker network create --driver overlay --attachable my-overlay

# List networks
docker network ls
docker network ls --filter driver=bridge

# Remove network
docker network rm mynet

# Inspect network
docker network inspect mynet
docker network inspect --format '{{range .Containers}}{{.Name}} {{end}}' mynet

# Connect/disconnect container to network
docker network connect mynet my-nginx
docker network connect --ip 172.20.0.10 mynet my-nginx
docker network disconnect mynet my-nginx
```

---

## 6. Docker Networking Deep Dive

### Network Drivers

| Driver | Use Case | Scope | DNS | Container-to-Container |
|--------|----------|-------|-----|----------------------|
| **bridge** | Default; containers on same host | Local | Yes (custom) | Via IP or DNS (custom networks) |
| **host** | Performance-critical; no isolation | Local | N/A | Shares host network |
| **none** | Complete network isolation | Local | No | None |
| **overlay** | Multi-host (Swarm/K8s) | Swarm | Yes | Across hosts |
| **macvlan** | Containers need real IPs on LAN | Local | No | Via LAN |

### Bridge Network (Default)

```bash
# Default bridge — containers communicate by IP only
docker run -d --name web1 nginx
docker run -d --name web2 nginx
# web1 cannot reach web2 by name on default bridge

# Custom bridge — enables DNS resolution between containers
docker network create app-net
docker run -d --name web1 --network app-net nginx
docker run -d --name web2 --network app-net nginx
# web2 CAN reach web1 by name: curl http://web1
```

### Host Network

```bash
# Container shares host's network stack directly
docker run -d --network host nginx
# nginx is available on host's port 80 directly — no -p needed
# Best for: high-throughput apps where NAT overhead matters
```

### None Network

```bash
# Completely isolated — no network interface (except loopback)
docker run -d --network none alpine sleep 3600
# Use case: security-sensitive batch processing
```

### Overlay Network (Docker Swarm)

```bash
# Create overlay for multi-host communication
docker network create --driver overlay --attachable my-overlay
# Containers on different hosts can communicate via this network
```

### Macvlan Network

```bash
# Containers get IPs directly on physical network
docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 my-macvlan

docker run -d --network my-macvlan --ip 192.168.1.100 nginx
# Container appears as physical device on LAN
```

### Container DNS Resolution

```bash
# On custom networks, Docker provides built-in DNS (127.0.0.11)
docker network create backend
docker run -d --name postgres --network backend postgres:15
docker run -d --name api --network backend myapi:1.0
# Inside 'api' container: postgres resolves to the postgres container's IP

# Add DNS aliases
docker run -d --name postgres --network backend --network-alias db postgres:15
# Now both 'postgres' and 'db' resolve to the same container
```

### Port Mapping (-p)

```bash
# Syntax: -p [host_ip:]host_port:container_port[/protocol]
docker run -p 8080:80 nginx            # All interfaces, TCP
docker run -p 127.0.0.1:8080:80 nginx  # Localhost only
docker run -p 8080:80/udp nginx        # UDP
docker run -p 8080-8090:80-90 nginx    # Port range
docker run -P nginx                     # Map all EXPOSE'd ports to random host ports
```

---

## 7. Docker Volumes Deep Dive

### Volume Types Comparison

| Type | Syntax | Managed by Docker | Performance | Use Case |
|------|--------|-------------------|-------------|----------|
| **Named Volume** | `-v mydata:/data` | Yes | Best | Databases, persistent data |
| **Bind Mount** | `-v /host/path:/container/path` | No | Good | Development, config files |
| **tmpfs** | `--tmpfs /tmp` | Yes (memory) | Fastest | Sensitive data, caches |

### Named Volumes

```bash
# Create and use
docker volume create pgdata
docker run -d --name db -v pgdata:/var/lib/postgresql/data postgres:15

# Data persists even when container is removed
docker rm -f db
docker run -d --name db2 -v pgdata:/var/lib/postgresql/data postgres:15
# db2 has all the data from db

# Inspect volume location on host
docker volume inspect pgdata
# Mountpoint: /var/lib/docker/volumes/pgdata/_data
```

### Bind Mounts

```bash
# Development: live code reload
docker run -d -v $(pwd)/src:/app/src -p 3000:3000 node-dev

# Configuration files
docker run -d -v /host/nginx.conf:/etc/nginx/nginx.conf:ro nginx

# Read-only mount (:ro)
docker run -d -v $(pwd)/config:/app/config:ro myapp
```

### tmpfs Mounts

```bash
# Store sensitive data in memory (never written to disk)
docker run -d --tmpfs /run/secrets:rw,noexec,nosuid,size=64m myapp

# Temporary cache that doesn't need persistence
docker run -d --tmpfs /tmp:size=256m myapp
```

### When to Use Which

| Scenario | Best Choice | Reason |
|----------|-------------|--------|
| Database data (Postgres, MySQL) | Named volume | Docker manages, survives container lifecycle |
| Development hot-reload | Bind mount | See changes immediately |
| Config files in production | Bind mount (read-only) | Host-managed, auditable |
| Secrets at runtime | tmpfs | Never persisted to disk |
| Build caches | Named volume | Shared between builds |
| Log files | Named volume or bind mount | Depends on log aggregation strategy |

### Data Persistence Patterns

```bash
# Pattern 1: Database with backup
docker volume create db-data
docker run -d --name db -v db-data:/var/lib/postgresql/data postgres:15

# Backup volume data
docker run --rm -v db-data:/data -v $(pwd)/backup:/backup alpine \
  tar czf /backup/db-backup-$(date +%Y%m%d).tar.gz -C /data .

# Pattern 2: Shared volume between containers
docker volume create shared-assets
docker run -d --name builder -v shared-assets:/output mybuilder
docker run -d --name web -v shared-assets:/usr/share/nginx/html:ro nginx
```

---

## 8. Docker Compose

### 8.1 Full Syntax Reference

```yaml
# docker-compose.yml (Compose file version 3.8+)
version: "3.8"  # Optional in modern Docker Compose

services:
  service-name:
    image: nginx:latest              # Use pre-built image
    build:                           # OR build from Dockerfile
      context: ./app
      dockerfile: Dockerfile.prod
      args:
        NODE_ENV: production
      target: production             # Multi-stage target
    container_name: my-service       # Custom container name
    restart: unless-stopped          # no | always | on-failure | unless-stopped
    ports:
      - "8080:80"                    # host:container
      - "127.0.0.1:3000:3000"       # bind to localhost
    expose:
      - "3000"                       # Expose to other services only
    environment:
      - NODE_ENV=production
      - DB_HOST=postgres
    env_file:
      - .env
      - .env.production
    volumes:
      - ./src:/app/src               # Bind mount
      - pgdata:/var/lib/postgresql/data  # Named volume
      - /tmp:/tmp:ro                 # Read-only bind
    networks:
      - frontend
      - backend
    depends_on:
      postgres:
        condition: service_healthy   # Wait for healthcheck
      redis:
        condition: service_started
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 10s
    deploy:
      resources:
        limits:
          cpus: "1.0"
          memory: 512M
        reservations:
          cpus: "0.5"
          memory: 256M
      replicas: 3
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
    command: ["node", "server.js"]   # Override CMD
    entrypoint: ["/entrypoint.sh"]   # Override ENTRYPOINT
    working_dir: /app
    user: "1000:1000"
    read_only: true
    tmpfs:
      - /tmp
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    profiles:
      - debug                        # Only start with --profile debug

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true                   # No external access

volumes:
  pgdata:
    driver: local
```

### 8.2 Complete Multi-Service Example

```yaml
# docker-compose.yml — Full-stack application
version: "3.8"

services:
  # ===== NGINX Reverse Proxy =====
  nginx:
    image: nginx:1.25-alpine
    container_name: nginx-proxy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
    depends_on:
      app:
        condition: service_healthy
    networks:
      - frontend
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost/health"]
      interval: 30s
      timeout: 5s
      retries: 3

  # ===== Node.js Application =====
  app:
    build:
      context: ./app
      dockerfile: Dockerfile
      target: production
    container_name: node-app
    environment:
      - NODE_ENV=production
      - DB_HOST=postgres
      - DB_PORT=5432
      - DB_NAME=myapp
      - DB_USER=appuser
      - DB_PASS_FILE=/run/secrets/db_password
      - REDIS_URL=redis://redis:6379
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - frontend
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost:3000/health"]
      interval: 15s
      timeout: 5s
      retries: 3
      start_period: 10s
    deploy:
      resources:
        limits:
          cpus: "1.0"
          memory: 512M
    secrets:
      - db_password

  # ===== PostgreSQL Database =====
  postgres:
    image: postgres:15-alpine
    container_name: postgres-db
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=appuser
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./db/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    networks:
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser -d myapp"]
      interval: 10s
      timeout: 5s
      retries: 5
    deploy:
      resources:
        limits:
          cpus: "0.5"
          memory: 256M
    secrets:
      - db_password

  # ===== Redis Cache =====
  redis:
    image: redis:7-alpine
    container_name: redis-cache
    command: redis-server --appendonly yes --maxmemory 128mb --maxmemory-policy allkeys-lru
    volumes:
      - redis-data:/data
    networks:
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3

  # ===== Debug tools (only with --profile debug) =====
  adminer:
    image: adminer:latest
    ports:
      - "8080:8080"
    networks:
      - backend
    profiles:
      - debug
    depends_on:
      - postgres

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true  # No internet access for backend services

volumes:
  pgdata:
    driver: local
  redis-data:
    driver: local

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

### 8.3 Docker Compose Commands

```bash
# Start services
docker compose up                    # Foreground
docker compose up -d                 # Detached
docker compose up -d --build         # Rebuild images before starting
docker compose up -d --scale app=3   # Scale specific service

# Stop services
docker compose down                  # Stop and remove containers
docker compose down -v               # Also remove volumes
docker compose down --rmi all        # Also remove images

# Service management
docker compose start                 # Start existing containers
docker compose stop                  # Stop without removing
docker compose restart               # Restart services
docker compose pause                 # Pause services
docker compose unpause               # Unpause services

# View status
docker compose ps                    # List services
docker compose logs                  # View all logs
docker compose logs -f app           # Follow specific service
docker compose top                   # Running processes

# Execute commands
docker compose exec app bash         # Shell into running service
docker compose run app npm test      # Run one-off command

# Build
docker compose build                 # Build all services
docker compose build --no-cache app  # Rebuild without cache

# Configuration
docker compose config               # Validate and view resolved config
docker compose --profile debug up -d # Start with profile
```

### 8.4 Compose Profiles for Different Environments

```yaml
# docker-compose.yml
services:
  app:
    build: .
    profiles: []  # Always starts (no profile = always active)

  postgres:
    image: postgres:15
    profiles: []

  # Development tools
  mailhog:
    image: mailhog/mailhog
    ports:
      - "8025:8025"
    profiles:
      - dev

  # Monitoring
  prometheus:
    image: prom/prometheus
    profiles:
      - monitoring

  grafana:
    image: grafana/grafana
    profiles:
      - monitoring

  # Testing
  selenium:
    image: selenium/standalone-chrome
    profiles:
      - test
```

```bash
# Usage:
docker compose --profile dev up -d               # Dev tools included
docker compose --profile monitoring up -d        # With monitoring
docker compose --profile dev --profile monitoring up -d  # Both
```

---

## 9. Docker Security Best Practices

### 9.1 Run as Non-Root

```dockerfile
# Create non-root user in Dockerfile
FROM node:18-alpine
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Copy files with correct ownership
COPY --chown=appuser:appgroup . /app
WORKDIR /app

USER appuser
CMD ["node", "server.js"]
```

```bash
# Or at runtime
docker run --user 1000:1000 myapp
docker run --user nobody myapp
```

### 9.2 Minimal Base Images

| Base Image | Size | Security Surface |
|-----------|------|-----------------|
| `ubuntu:22.04` | ~77MB | Large (many packages) |
| `debian:slim` | ~80MB | Medium |
| `alpine:3.18` | ~7MB | Small (musl libc) |
| `distroless` | ~2-20MB | Minimal (no shell!) |
| `scratch` | 0MB | Nothing (static binaries only) |

```dockerfile
# Google Distroless — no shell, no package manager
FROM gcr.io/distroless/nodejs18-debian12
COPY --from=builder /app /app
WORKDIR /app
CMD ["server.js"]
```

### 9.3 Image Scanning with Trivy

```bash
# Install Trivy
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin

# Scan an image
trivy image myapp:1.0
trivy image --severity HIGH,CRITICAL myapp:1.0
trivy image --exit-code 1 --severity CRITICAL myapp:1.0  # Fail CI if critical vulns

# Scan Dockerfile
trivy config Dockerfile

# Scan in CI/CD pipeline
trivy image --format json --output results.json myapp:1.0
```

### 9.4 Read-Only Filesystem

```bash
# Make container filesystem read-only
docker run --read-only --tmpfs /tmp --tmpfs /run nginx

# In docker-compose
services:
  app:
    read_only: true
    tmpfs:
      - /tmp
      - /run
```

### 9.5 Drop Capabilities

```bash
# Drop ALL capabilities and add only what's needed
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE nginx

# Common capabilities to add back:
# NET_BIND_SERVICE — bind to ports < 1024
# CHOWN — change file ownership
# SETUID/SETGID — change user/group
```

### 9.6 Secrets Handling

```bash
# NEVER do this in Dockerfile:
# ENV DB_PASSWORD=mysecret        ← Visible in image history!
# COPY .env /app/.env             ← Baked into image!

# Use Docker secrets (Compose)
echo "supersecret" > ./secrets/db_pass.txt
```

```yaml
# docker-compose.yml
services:
  app:
    secrets:
      - db_password
    environment:
      - DB_PASS_FILE=/run/secrets/db_password

secrets:
  db_password:
    file: ./secrets/db_pass.txt
```

```bash
# At runtime — mount secret as file
docker run -v ./secrets/db_pass.txt:/run/secrets/db_password:ro myapp

# BuildKit secrets for build-time (never stored in layers)
docker build --secret id=npmrc,src=.npmrc .
# In Dockerfile:
RUN --mount=type=secret,id=npmrc,target=/root/.npmrc npm install
```

### 9.7 Content Trust and Signing

```bash
# Enable Docker Content Trust (image signing)
export DOCKER_CONTENT_TRUST=1
docker push myregistry.com/myapp:1.0  # Will sign the image
docker pull myregistry.com/myapp:1.0  # Verifies signature

# Verify specific image digest
docker pull myapp@sha256:abc123def456...
```

### Security Checklist Summary

- [x] Use non-root user
- [x] Use minimal base images (alpine/distroless)
- [x] Scan images for vulnerabilities regularly
- [x] Make filesystems read-only where possible
- [x] Drop all Linux capabilities, add only needed ones
- [x] Never bake secrets into images
- [x] Use specific image tags (not `:latest`)
- [x] Enable Docker Content Trust
- [x] Limit resources (memory, CPU)
- [x] Use `--no-new-privileges` flag
- [x] Keep Docker and base images updated

---

## 10. Docker Registry

### 10.1 Docker Hub

```bash
# Login
docker login
docker login -u username

# Push to Docker Hub
docker tag myapp:1.0 username/myapp:1.0
docker push username/myapp:1.0

# Pull from Docker Hub
docker pull username/myapp:1.0
```

### 10.2 Private Registries

| Registry | Provider | Use Case |
|----------|----------|----------|
| **ECR** | AWS | AWS-native workloads |
| **ACR** | Azure | Azure-native workloads |
| **GCR/Artifact Registry** | Google Cloud | GCP workloads |
| **Harbor** | Open Source | Self-hosted, enterprise |
| **GitLab Registry** | GitLab | CI/CD integrated |

```bash
# AWS ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 123456789.dkr.ecr.us-east-1.amazonaws.com
docker tag myapp:1.0 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:1.0
docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:1.0

# Azure ACR
az acr login --name myregistry
docker tag myapp:1.0 myregistry.azurecr.io/myapp:1.0
docker push myregistry.azurecr.io/myapp:1.0

# Google Artifact Registry
gcloud auth configure-docker us-central1-docker.pkg.dev
docker tag myapp:1.0 us-central1-docker.pkg.dev/my-project/my-repo/myapp:1.0
docker push us-central1-docker.pkg.dev/my-project/my-repo/myapp:1.0
```

### 10.3 Image Tagging Strategy

```bash
# Semantic versioning
myapp:1.0.0
myapp:1.0
myapp:1

# Git-based tags
myapp:abc1234          # Git commit SHA
myapp:main-abc1234     # Branch + SHA
myapp:v1.2.3           # Git tag

# Environment tags
myapp:staging
myapp:production

# Date-based
myapp:2024-01-15
myapp:20240115-abc1234

# Best practice: use immutable tags in production
# AVOID: myapp:latest (mutable, unpredictable)
# PREFER: myapp:1.2.3 or myapp:sha-abc1234
```

---

## 11. Troubleshooting Docker

### 11.1 Common Errors and Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `permission denied` on docker socket | User not in docker group | `sudo usermod -aG docker $USER && newgrp docker` |
| `port is already allocated` | Host port in use | Change port mapping or stop conflicting service |
| `no space left on device` | Docker disk full | `docker system prune -a --volumes` |
| `OOMKilled` | Container exceeded memory limit | Increase `--memory` or optimize app |
| `exec format error` | Wrong architecture | Build for correct platform: `--platform linux/amd64` |
| `network not found` | Network deleted or typo | `docker network create <name>` |
| `name already in use` | Container name exists | `docker rm <name>` or use different name |
| `image not found` | Wrong tag or not pulled | `docker pull <image>` or check spelling |
| `COPY failed: file not found` | File not in build context | Check `.dockerignore` and path |
| `DNS resolution failed` | DNS issue in container | Use custom network or `--dns` flag |

### 11.2 Debug Techniques

```bash
# 1. Shell into running container
docker exec -it mycontainer sh

# 2. Run image with shell (override entrypoint)
docker run -it --entrypoint sh myimage

# 3. Check container events
docker events --filter container=mycontainer

# 4. Inspect container details
docker inspect mycontainer | jq '.[0].State'
docker inspect mycontainer | jq '.[0].NetworkSettings'

# 5. View filesystem changes
docker diff mycontainer

# 6. Export container filesystem for inspection
docker export mycontainer | tar -tvf - | head -50

# 7. Check Docker daemon logs
sudo journalctl -u docker.service --since "1 hour ago"

# 8. Debug build issues (use intermediate containers)
docker build --progress=plain --no-cache .
# BuildKit debug: set target to failing stage

# 9. Network debugging
docker run --rm --network container:mycontainer nicolaka/netshoot tcpdump -i any
docker run --rm --network mynet nicolaka/netshoot dig myservice
```

### 11.3 Container Logs

```bash
# View logs
docker logs mycontainer
docker logs -f --tail 50 mycontainer           # Follow last 50 lines
docker logs --since "2024-01-01T00:00:00" mycontainer

# Configure logging driver
docker run --log-driver=json-file --log-opt max-size=10m --log-opt max-file=3 myapp

# Common log drivers: json-file, syslog, fluentd, awslogs, gelf
```

### 11.4 Resource Issues

```bash
# Check disk usage
docker system df              # Overview
docker system df -v           # Detailed per image/container/volume

# Clean up resources
docker system prune           # Remove stopped containers, unused networks, dangling images
docker system prune -a        # Also remove unused images
docker system prune --volumes # Also remove unused volumes

# Check container resource usage
docker stats --no-stream
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# Limit resources
docker run --memory=256m --cpus=0.5 myapp
docker update --memory=512m --cpus=1.0 mycontainer  # Update running container
```

---

## 12. Lessons Learned / Pro Tips

1. **Always use specific tags** — Never use `:latest` in production. Pin versions: `nginx:1.25.3-alpine`.

2. **Order Dockerfile for cache efficiency** — Put rarely changing instructions first (OS deps), frequently changing last (app code). `COPY package.json` before `COPY . .` so npm install is cached.

3. **One process per container** — Follow the single responsibility principle. Don't run nginx + app + db in one container.

4. **Use `.dockerignore`** — Reduce build context size. A 500MB context becomes 50MB = faster builds.

5. **Multi-stage builds are non-negotiable** — Your production image doesn't need gcc, make, or node_modules devDependencies.

6. **Health checks everywhere** — Docker (and orchestrators) need to know if your app is healthy. Always define HEALTHCHECK.

7. **Never store secrets in images** — Use Docker secrets, environment variables from secure sources, or mounted files. Secrets in ENV show in `docker inspect`.

8. **Use `docker compose up --build`** — Rebuilds images before starting. Prevents serving stale code.

9. **Named volumes over bind mounts in production** — Docker manages them, they perform better, and they're portable.

10. **Use `--init` flag** — Adds tini as PID 1 to properly handle signals and reap zombie processes: `docker run --init myapp`.

11. **Leverage BuildKit** — Enable with `DOCKER_BUILDKIT=1`. Parallel builds, better caching, secret mounts, SSH forwarding.

12. **Tag images with git SHA** — Always know exactly which code is in which image: `myapp:$(git rev-parse --short HEAD)`.

13. **Use `docker compose down -v` carefully** — The `-v` flag removes volumes (and your data!). Use only when you want a fresh start.

14. **Custom networks over default bridge** — Custom bridge networks provide DNS resolution between containers. Default bridge requires IP-based communication.

15. **Set resource limits** — Always set `--memory` and `--cpus` in production. One runaway container shouldn't kill the host.

16. **Use `docker system prune` regularly** — Docker accumulates garbage. Schedule cleanup in CI/CD agents and development machines.

17. **Test your Dockerfile locally before CI** — A broken Dockerfile discovered in CI wastes 10 minutes. Test locally first.

18. **Log to stdout/stderr** — Docker captures stdout/stderr automatically. Don't write to files inside containers.

19. **Use `COPY --chown`** — Avoids an extra `RUN chown` layer that duplicates file data.

20. **Keep images under 200MB** — If your image is bigger, you can probably optimize it. Smaller = faster deploys.

---

## 13. Interview Questions

### Q1: What is the difference between a Docker image and a container?

**Answer:** An **image** is a read-only template containing application code, runtime, libraries, and configuration. A **container** is a running instance of an image with a writable layer on top. You can create many containers from the same image. Think of an image as a class and a container as an object/instance.

### Q2: Explain Docker's layered filesystem. Why is it important?

**Answer:** Each Dockerfile instruction creates a read-only layer. Layers are stacked using a union filesystem (OverlayFS). This is important because: (1) layers are cached — unchanged layers aren't rebuilt, (2) layers are shared between images — if 10 images use `node:18-alpine`, it's stored once, (3) only the diff is stored per layer, saving disk space. The container adds a thin writable layer on top.

### Q3: What is the difference between CMD and ENTRYPOINT?

**Answer:** `CMD` provides default arguments that can be entirely overridden by passing arguments to `docker run`. `ENTRYPOINT` defines the main executable that always runs. When both are used, `CMD` provides default arguments to `ENTRYPOINT`. Use ENTRYPOINT when the container IS a specific command (like a CLI tool), and CMD when you want a default that users might override.

### Q4: How does multi-stage build work and why use it?

**Answer:** Multi-stage builds use multiple `FROM` statements in one Dockerfile. Each FROM starts a new build stage. You can copy artifacts from earlier stages using `COPY --from=<stage>`. Benefits: (1) final image doesn't include build tools (gcc, npm dev deps), (2) dramatically smaller images (often 80-90% reduction), (3) single Dockerfile for build and runtime, (4) better security — fewer packages = fewer vulnerabilities.

### Q5: Explain Docker networking. What's the difference between bridge, host, and overlay?

**Answer:** **Bridge** (default) creates an isolated network on the host; containers communicate via internal IPs or DNS (on custom bridges). **Host** removes network isolation — the container shares the host's network stack directly, eliminating NAT overhead. **Overlay** enables communication between containers across multiple Docker hosts (used in Swarm/Kubernetes). Choose bridge for single-host isolation, host for performance-critical apps, overlay for multi-host deployments.

### Q6: How do you persist data in Docker containers?

**Answer:** Three options: (1) **Named volumes** (`-v mydata:/data`) — Docker-managed, best for databases and persistent data. (2) **Bind mounts** (`-v /host/path:/container/path`) — map host directory into container, best for development. (3) **tmpfs** (`--tmpfs /tmp`) — in-memory only, best for sensitive temporary data. Containers are ephemeral by design; always use volumes for data that must survive container recreation.

### Q7: What is Docker Compose and when would you use it?

**Answer:** Docker Compose is a tool for defining and running multi-container applications using a YAML file. Use it when: (1) your app has multiple services (app + db + cache), (2) you need reproducible development environments, (3) you want to define networks, volumes, and dependencies declaratively, (4) you need to coordinate service startup order. It's ideal for local development and single-host deployments.

### Q8: How do you secure a Docker container?

**Answer:** Key practices: (1) Run as non-root user, (2) Use minimal base images (alpine/distroless), (3) Scan images with Trivy/Snyk, (4) Drop all Linux capabilities and add only needed ones, (5) Use read-only filesystems, (6) Never embed secrets in images, (7) Set resource limits, (8) Keep images updated, (9) Use specific tags (not :latest), (10) Enable Docker Content Trust for image signing.

### Q9: What happens when you run `docker run`?

**Answer:** Docker: (1) Pulls the image if not local, (2) Creates a new container (writable layer on image), (3) Allocates a network interface and IP, (4) Starts the container and runs the specified command (ENTRYPOINT/CMD). Under the hood: dockerd instructs containerd, which calls runc to set up namespaces, cgroups, and the root filesystem, then executes the process.

### Q10: How do you reduce Docker image size?

**Answer:** (1) Use minimal base images (alpine, slim, distroless), (2) Multi-stage builds, (3) Combine RUN commands and clean up in the same layer, (4) Remove package manager caches (`rm -rf /var/lib/apt/lists/*`), (5) Use `.dockerignore` to exclude unnecessary files, (6) Don't install dev dependencies in production, (7) Use `--no-install-recommends` with apt-get.

### Q11: Explain the difference between COPY and ADD in Dockerfile.

**Answer:** `COPY` simply copies files from the build context into the image. `ADD` does the same but has extra features: (1) auto-extracts local tar archives (tar, gzip, bzip2, xz), (2) can download files from URLs (deprecated practice). Best practice: always use `COPY` unless you specifically need tar extraction. `COPY` is more transparent and predictable.

### Q12: What is Docker Content Trust?

**Answer:** Docker Content Trust (DCT) provides image signing and verification. When enabled (`DOCKER_CONTENT_TRUST=1`), Docker only pulls/runs images that are signed and verified. Publishers sign images with private keys, and consumers verify signatures before running. This prevents man-in-the-middle attacks and ensures image integrity.

### Q13: How does container DNS work in Docker?

**Answer:** On custom bridge networks, Docker runs an embedded DNS server at 127.0.0.11. Containers can resolve other containers by their name or network alias. The default bridge network does NOT provide DNS (only `--link` which is deprecated). This is why custom networks are recommended — they enable service discovery without hardcoding IPs.

### Q14: What is the difference between `docker stop` and `docker kill`?

**Answer:** `docker stop` sends SIGTERM to the main process, allowing graceful shutdown (closing connections, flushing data). After a timeout (default 10s), it sends SIGKILL. `docker kill` sends SIGKILL immediately, forcing the process to terminate without cleanup. Always prefer `docker stop` in production to avoid data corruption.

### Q15: How would you debug a container that keeps crashing?

**Answer:** (1) Check logs: `docker logs --tail 100 <container>`, (2) Check exit code: `docker inspect <container> | jq '.[0].State'`, (3) Override entrypoint to get a shell: `docker run -it --entrypoint sh <image>`, (4) Check events: `docker events --filter container=<name>`, (5) Check resource limits: `docker stats`, (6) Look for OOMKilled in inspect output, (7) Check if health check is failing, (8) Review Dockerfile for proper signal handling.

### Q16: What are Docker namespaces and cgroups?

**Answer:** **Namespaces** provide isolation — each container gets its own view of: PID (process IDs), NET (network interfaces), MNT (filesystems), UTS (hostname), IPC (inter-process communication), USER (user/group IDs). **Cgroups** (Control Groups) limit and account for resources: CPU time, memory, disk I/O, network bandwidth. Together they create lightweight isolation without needing a full VM.

### Q17: Explain the Docker build cache. How do you optimize for it?

**Answer:** Docker caches each layer. If a layer's instruction and inputs haven't changed, Docker reuses the cached layer. Cache is invalidated from the first changed layer downward. To optimize: (1) Put stable layers first (FROM, RUN apt-get install), (2) Copy dependency files before source code (package.json before COPY . .), (3) Use `--mount=type=cache` for package manager caches, (4) Separate dev and prod dependencies.

---

## 14. Free Resources

| # | Resource | Link | Type |
|---|----------|------|------|
| 1 | **Docker Official Documentation** | https://docs.docker.com | Docs |
| 2 | **Docker Getting Started Tutorial** | https://docs.docker.com/get-started/ | Tutorial |
| 3 | **Play with Docker (browser lab)** | https://labs.play-with-docker.com | Hands-on |
| 4 | **Docker Curriculum (Prakhar Srivastav)** | https://docker-curriculum.com | Tutorial |
| 5 | **Dockerfile Best Practices** | https://docs.docker.com/develop/develop-images/dockerfile_best-practices/ | Guide |
| 6 | **Docker Security Cheat Sheet (OWASP)** | https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html | Security |
| 7 | **Awesome Docker (GitHub)** | https://github.com/veggiemonk/awesome-docker | Curated list |
| 8 | **Container Training** | https://container.training | Workshop |
| 9 | **Docker Deep Dive (Nigel Poulton)** | https://github.com/nigelpoulton/docker-deep-dive | Book/Repo |
| 10 | **KodeKloud Docker Free Course** | https://kodekloud.com/courses/docker-for-the-absolute-beginner/ | Video |
| 11 | **Trivy (Vulnerability Scanner)** | https://github.com/aquasecurity/trivy | Tool |
| 12 | **Docker Slim (Image Optimizer)** | https://github.com/slimtoolkit/slim | Tool |

---

## Quick Reference Card

```bash
# === Most Used Commands ===
docker build -t app:1.0 .                # Build image
docker run -d -p 8080:80 --name web app   # Run container
docker exec -it web sh                    # Shell access
docker logs -f web                        # Follow logs
docker stop web && docker rm web          # Cleanup
docker compose up -d --build              # Start stack
docker compose down                       # Stop stack
docker system prune -a                    # Free disk space
```

---

> **Next:** Month 2 - Part 2: Container Orchestration with Kubernetes
