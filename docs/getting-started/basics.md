# Docker Basics

Now that you've [installed Docker](installation.md) and [run your first containers](first-steps.md), let's build a solid understanding of how Docker actually works. This foundation will make everything else make sense!

## Docker's Architecture (The Big Picture)

Think of Docker like a **shipping company** for your applications:

```
Your Code → Docker → Standardized Container → Runs Anywhere
```

### The Key Players

1. **Docker Client** - The command-line tool you type commands into
2. **Docker Daemon** - The background service that does the actual work
3. **Docker Registry** - The warehouse where container images are stored (like Docker Hub)

!!! info "Simple Analogy"
    - **Docker Client** = Your smartphone
    - **Docker Daemon** = The cell tower that processes your requests
    - **Docker Registry** = The app store where applications live

## Core Concepts You Must Understand

### Images vs Containers (The Most Important Distinction)

This confuses **every** Docker beginner, so let's make it crystal clear:

| Concept | What It Is | Real-World Analogy | Command to See |
|---------|------------|-------------------|----------------|
| **Image** | Blueprint/Template | Recipe in a cookbook | `docker images` |
| **Container** | Running instance | Cake baked from recipe | `docker ps` |

**Key Points:**
- One image can create **multiple containers**
- Images are **read-only** (can't be changed)
- Containers are **temporary** (disappear when stopped, unless you save data)

### Volumes (Persistent Storage)

Containers are designed to be **ephemeral** (temporary). When you stop a container, everything inside disappears.

**Problem:** What if you need to save data?  
**Solution:** Volumes - external storage that persists beyond container lifecycles.

!!! tip "Think of volumes like"
    External hard drives you can plug into different computers. The computer (container) can be replaced, but your data (volume) stays safe.

### Networks (Container Communication)

By default, containers are isolated from each other. Networks let them communicate when needed.

**Common Use Case:** A web application container needs to talk to a database container.

## Essential Docker Commands

### Container Management

```bash
# Create and run a new container
docker run [OPTIONS] IMAGE [COMMAND]

# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Stop a running container
docker stop CONTAINER_NAME_OR_ID

# Start a stopped container  
docker start CONTAINER_NAME_OR_ID

# Remove a container
docker rm CONTAINER_NAME_OR_ID
```

### Image Management

```bash
# List downloaded images
docker images

# Download an image without running it
docker pull IMAGE_NAME

# Remove an image
docker rmi IMAGE_NAME

# Search for images on Docker Hub
docker search TERM
```

### Information and Debugging

```bash
# View container logs
docker logs CONTAINER_NAME_OR_ID

# Execute a command inside a running container
docker exec -it CONTAINER_NAME_OR_ID COMMAND

# Show detailed information about a container
docker inspect CONTAINER_NAME_OR_ID
```

## Docker Run Options (The Powerful Ones)

Understanding `docker run` options is crucial:

### Background vs Foreground

```bash
# Foreground (blocks terminal)
docker run nginx

# Background (detached)
docker run -d nginx
```

### Port Mapping

```bash
# Map host port 8080 to container port 80
docker run -p 8080:80 nginx

# Access multiple ports
docker run -p 8080:80 -p 8443:443 nginx
```

### Naming Containers

```bash
# Auto-generated name (like "quirky_einstein")
docker run nginx

# Custom name
docker run --name my-web-server nginx
```

### Environment Variables

```bash
# Set environment variables
docker run -e DB_HOST=localhost -e DB_PORT=5432 my-app
```

## Understanding the Docker Workflow

### Typical Development Workflow

1. **Find or Create** an image that has what you need
2. **Run a container** from that image
3. **Configure** the container (ports, environment, volumes)
4. **Test** your application
5. **Stop and remove** when done (or keep running)

### Image Lifecycle

1. **Pull** image from registry (automatic when you `docker run`)
2. **Create** containers from image
3. **Update** image when new version available
4. **Clean up** old images to save space

## Common Beginner Mistakes

### Mistake #1: Confusing Images and Containers

❌ **Wrong thinking:** "I deleted the container, now the image is gone"  
✅ **Correct:** Images and containers are separate. Deleting a container doesn't affect the image.

### Mistake #2: Not Using Volume for Important Data

❌ **Wrong:** Storing database data inside container  
✅ **Correct:** Using volumes for data that must persist

### Mistake #3: Running Everything in Foreground

❌ **Wrong:** `docker run nginx` (blocks terminal)  
✅ **Correct:** `docker run -d nginx` (runs in background)

### Mistake #4: Not Cleaning Up

❌ **Wrong:** Leaving hundreds of stopped containers  
✅ **Correct:** Regular cleanup with `docker rm` and `docker rmi`

## Practical Examples

### Example 1: Web Development Environment

```bash
# Run a web server for development
docker run -d -p 3000:80 --name dev-server nginx

# Check it's running
docker ps

# View logs if something goes wrong
docker logs dev-server

# Clean up when done
docker stop dev-server && docker rm dev-server
```

### Example 2: Database for Testing

```bash
# Run PostgreSQL with persistent data
docker run -d \
  --name test-db \
  -e POSTGRES_PASSWORD=mypassword \
  -p 5432:5432 \
  -v postgres-data:/var/lib/postgresql/data \
  postgres

# Connect from host
psql -h localhost -U postgres
```

## Docker vs Traditional Deployment

| Aspect | Traditional | Docker |
|--------|------------|---------|
| **Setup** | Install dependencies on host | Everything in container |
| **Conflicts** | Version conflicts common | Each container isolated |
| **Portability** | "Works on my machine" | Works everywhere |
| **Cleanup** | Manual uninstall | Delete container |
| **Scaling** | Complex | Start more containers |

## Quick Reference: When to Use What

### Use `docker run` when:
- Starting a new container from an image
- Testing something quickly
- One-time tasks

### Use `docker start/stop` when:
- Managing existing containers
- Containers you want to reuse
- Preserving container configuration

### Use volumes when:
- Database data
- Log files
- Any data that must survive container restarts

### Use port mapping when:
- Accessing container services from host
- Web applications
- APIs that need external access

## What's Next?

Now that you understand Docker's fundamentals, you're ready to dive deeper:

- **[Images](../core-concepts/images.md)** - Master Docker images and registries
- **[Containers](../core-concepts/containers.md)** - Advanced container management techniques  
- **[Dockerfile](../core-concepts/dockerfile.md)** - Create your own custom images

!!! success "You've Got the Foundation!"
    Understanding images, containers, and basic commands puts you ahead of many Docker users. Everything else builds on these concepts - you're ready for the next level!