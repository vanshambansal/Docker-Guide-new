# Docker Command Cheatsheet

Quick reference for essential Docker commands organized by category. Bookmark this page for instant access to the commands you use most.

## Container Management

### Basic Container Operations

```bash
# Run a new container
docker run [OPTIONS] IMAGE [COMMAND]
docker run -it ubuntu bash                    # Interactive container
docker run -d nginx                          # Background (detached)
docker run -p 8080:80 nginx                  # Port mapping
docker run --name myapp nginx                # Named container
docker run --rm alpine echo "hello"          # Auto-remove after exit

# List containers
docker ps                                     # Running containers only
docker ps -a                                 # All containers (including stopped)
docker ps -q                                 # Container IDs only

# Container lifecycle
docker start CONTAINER                        # Start stopped container
docker stop CONTAINER                         # Stop running container
docker restart CONTAINER                     # Restart container
docker pause CONTAINER                       # Pause container processes
docker unpause CONTAINER                     # Unpause container
docker kill CONTAINER                        # Force stop container

# Remove containers
docker rm CONTAINER                          # Remove stopped container
docker rm -f CONTAINER                       # Force remove (stop + remove)
docker container prune                       # Remove all stopped containers
```

### Container Inspection and Debugging

```bash
# Get container information
docker inspect CONTAINER                     # Full container details (JSON)
docker logs CONTAINER                        # View container logs
docker logs -f CONTAINER                     # Follow logs in real-time
docker logs --tail 50 CONTAINER              # Last 50 log entries
docker stats CONTAINER                       # Real-time resource usage
docker top CONTAINER                         # Running processes in container

# Execute commands in running container
docker exec -it CONTAINER bash               # Interactive shell
docker exec CONTAINER ls -la                 # Single command
docker exec -u 0 CONTAINER bash              # Run as root user

# Copy files between container and host
docker cp file.txt CONTAINER:/path/          # Host to container
docker cp CONTAINER:/path/file.txt .         # Container to host

# Container resource usage
docker stats                                 # All containers stats
docker stats --no-stream                     # One-time stats snapshot
```

## Image Management

### Basic Image Operations

```bash
# List and search images
docker images                                # List local images
docker images -q                             # Image IDs only
docker search IMAGE_NAME                     # Search Docker Hub

# Pull and push images
docker pull IMAGE                            # Download image
docker pull IMAGE:TAG                        # Specific version
docker push IMAGE                            # Upload to registry
docker push USERNAME/IMAGE:TAG               # Push with tag

# Remove images
docker rmi IMAGE                             # Remove image
docker rmi -f IMAGE                          # Force remove
docker image prune                           # Remove unused images
docker image prune -a                        # Remove all unused images

# Image inspection
docker inspect IMAGE                         # Image details
docker history IMAGE                         # Image layers
```

### Building Images

```bash
# Build from Dockerfile
docker build .                              # Build from current directory
docker build -t NAME:TAG .                  # Build with name and tag
docker build -f Dockerfile.prod .           # Use specific Dockerfile
docker build --no-cache .                   # Build without cache
docker build --target stage-name .          # Multi-stage build target

# Build with BuildKit (faster builds)
DOCKER_BUILDKIT=1 docker build .
docker buildx build --platform linux/amd64,linux/arm64 .

# Tag images
docker tag IMAGE NEW_NAME:TAG               # Create new tag
docker tag IMAGE:latest IMAGE:v1.0          # Tag with version
```

## Volume Management

```bash
# Create and list volumes
docker volume create VOLUME_NAME             # Create named volume
docker volume ls                             # List all volumes
docker volume inspect VOLUME                 # Volume details

# Use volumes with containers
docker run -v VOLUME:/path/in/container IMAGE
docker run -v /host/path:/container/path IMAGE    # Bind mount
docker run --mount type=volume,source=vol,target=/data IMAGE

# Remove volumes
docker volume rm VOLUME                      # Remove volume
docker volume prune                          # Remove unused volumes

# Backup and restore volumes
docker run --rm -v VOLUME:/data -v $(pwd):/backup alpine \
  tar czf /backup/backup.tar.gz -C /data .   # Backup volume

docker run --rm -v VOLUME:/data -v $(pwd):/backup alpine \
  tar xzf /backup/backup.tar.gz -C /data     # Restore volume
```

## Network Management

```bash
# List and create networks
docker network ls                            # List networks
docker network create NETWORK               # Create network
docker network create --driver bridge my-net # Specify driver

# Connect containers to networks
docker run --network NETWORK IMAGE          # Run with network
docker network connect NETWORK CONTAINER    # Connect existing container
docker network disconnect NETWORK CONTAINER # Disconnect container

# Network inspection
docker network inspect NETWORK              # Network details
docker port CONTAINER                       # Port mappings

# Remove networks
docker network rm NETWORK                   # Remove network
docker network prune                        # Remove unused networks
```

## Docker Compose Commands

```bash
# Basic Compose operations
docker-compose up                           # Start services
docker-compose up -d                        # Start in background
docker-compose down                         # Stop and remove services
docker-compose restart                      # Restart all services
docker-compose stop                         # Stop services (keep containers)

# Service management
docker-compose start SERVICE                # Start specific service
docker-compose stop SERVICE                 # Stop specific service
docker-compose restart SERVICE              # Restart specific service
docker-compose scale SERVICE=3              # Scale service to 3 instances

# Logs and debugging
docker-compose logs                         # View all service logs
docker-compose logs -f SERVICE              # Follow specific service logs
docker-compose exec SERVICE bash            # Shell into service
docker-compose ps                           # List running services

# Build and pull
docker-compose build                        # Build all services
docker-compose build SERVICE                # Build specific service
docker-compose pull                         # Pull all images
docker-compose up --build                   # Build and start

# Configuration
docker-compose config                       # Validate and view config
docker-compose -f custom.yml up             # Use specific compose file
```

## System Management

```bash
# System information
docker version                              # Docker version info
docker info                                 # System-wide information
docker system df                            # Disk usage
docker system events                        # Real-time system events

# Cleanup commands
docker system prune                         # Remove unused data
docker system prune -a                      # Remove all unused data
docker system prune --volumes               # Include unused volumes

# Resource monitoring
docker stats                                # Container resource usage
docker events                               # System events stream
```

## Registry and Authentication

```bash
# Login and logout
docker login                                # Login to Docker Hub
docker login registry.company.com           # Login to private registry
docker logout                               # Logout

# Work with registries
docker tag IMAGE registry.com/USER/IMAGE    # Tag for registry
docker push registry.com/USER/IMAGE         # Push to registry
docker pull registry.com/USER/IMAGE         # Pull from registry
```

## Advanced Operations

### Multi-platform Builds

```bash
# Setup buildx for multi-platform
docker buildx create --name multibuilder --use
docker buildx inspect --bootstrap

# Build for multiple platforms
docker buildx build --platform linux/amd64,linux/arm64 -t IMAGE .
docker buildx build --platform linux/amd64,linux/arm64 --push -t IMAGE .
```

### Swarm Mode (Docker Swarm)

```bash
# Initialize swarm
docker swarm init                           # Initialize swarm mode
docker swarm join --token TOKEN HOST:PORT   # Join existing swarm

# Service management
docker service create --name SERVICE IMAGE  # Create service
docker service ls                           # List services
docker service ps SERVICE                   # Service tasks
docker service scale SERVICE=5              # Scale service
docker service rm SERVICE                   # Remove service

# Stack deployment
docker stack deploy -c compose.yml STACK    # Deploy stack
docker stack ls                             # List stacks
docker stack rm STACK                       # Remove stack
```

### Security and Scanning

```bash
# Image scanning
docker scout cves IMAGE                     # Scan for vulnerabilities
docker scout recommendations IMAGE          # Security recommendations

# Run with security options
docker run --security-opt=no-new-privileges:true IMAGE
docker run --read-only --tmpfs /tmp IMAGE
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE IMAGE
```

## Troubleshooting Commands

```bash
# Debug failing containers
docker run --rm -it IMAGE /bin/bash         # Debug image interactively
docker logs --details CONTAINER             # Detailed logs
docker exec -it CONTAINER /bin/bash         # Debug running container

# System diagnostics
docker system df -v                         # Detailed disk usage
docker info                                 # System information
docker version                              # Version details

# Network debugging
docker network ls                           # List networks
docker port CONTAINER                       # Port mappings
docker exec CONTAINER netstat -tulpn        # Network connections

# Performance monitoring
docker stats --no-stream                    # Resource usage snapshot
docker system events --since="1h"           # Recent system events
```

## Common Flag Combinations

### Run Container Options

```bash
# Development container
docker run -it --rm -v $(pwd):/app -w /app node:18 bash

# Production web server
docker run -d --name web --restart unless-stopped -p 80:80 nginx

# Database with persistent data
docker run -d --name db -e POSTGRES_PASSWORD=pass -v db_data:/var/lib/postgresql/data postgres

# Temporary debugging container
docker run --rm -it --network container:other_container alpine sh
```

### Build Options

```bash
# Production build
docker build -t app:latest --target production --no-cache .

# Development build with cache
docker build -t app:dev --target development .

# Multi-platform build
docker buildx build --platform linux/amd64,linux/arm64 -t app:latest --push .
```

## Quick Tips

**Use image IDs for exact references:**
```bash
docker run $(docker images -q IMAGE)        # Use latest image ID
```

**Combine commands for cleanup:**
```bash
docker stop $(docker ps -q) && docker rm $(docker ps -aq)  # Stop and remove all
```

**One-liner for complete cleanup:**
```bash
docker system prune -a --volumes -f         # Remove everything unused
```

**Format output for scripts:**
```bash
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

This cheatsheet covers the most frequently used Docker commands. Keep it bookmarked for quick reference during development and troubleshooting.