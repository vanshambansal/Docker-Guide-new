# Networks

Docker networks control how containers communicate with each other and the outside world. By default, containers can talk to each other, but networks let you create secure, isolated groups and control exactly which containers can communicate.

## Why Docker Networks Matter

**Without custom networks:**
```bash
# All containers can reach each other
docker run -d --name web nginx
docker run -d --name db postgres  
docker run -d --name cache redis

# Problem: Web can access DB, but so can any other container
# Security risk: No isolation between services
```

**With custom networks:**
```bash
# Create isolated networks
docker network create frontend
docker network create backend

# Web server only on frontend network
docker run -d --name web --network frontend nginx

# Database only on backend network  
docker run -d --name db --network backend postgres

# API on both networks (can access DB, serve web)
docker run -d --name api --network frontend --network backend myapi
```

Now the web server cannot directly access the database - only the API can!

## Docker's Default Networks

Docker creates three networks automatically:

```bash
# List all networks
docker network ls
```

**Expected output:**
```
NETWORK ID     NAME      DRIVER    SCOPE
b1234567890a   bridge    bridge    local
c2345678901b   host      host      local  
d3456789012c   none      null      local
```

### Bridge Network (Default)
- **What it does:** All containers connect here by default
- **IP range:** Usually 172.17.0.0/16
- **Container communication:** Containers can reach each other by IP address
- **Internet access:** Yes, containers can reach outside world

```bash
# These containers can communicate with each other
docker run -d --name app1 nginx
docker run -d --name app2 nginx

# Test connectivity
docker exec app1 ping 172.17.0.3  # app2's IP
```

### Host Network
- **What it does:** Container uses host's network directly
- **Use case:** High-performance applications that need host networking
- **Security:** Less isolated, container shares all host ports

```bash
# Container uses host's network stack
docker run -d --network host nginx
# nginx now directly uses host's port 80
```

### None Network  
- **What it does:** No network access at all
- **Use case:** Highly secure containers that don't need network access

```bash
# Container has no network access
docker run -it --network none alpine ping google.com
# ping: bad address 'google.com'
```

## Creating Custom Networks

### Basic Custom Network

```bash
# Create a custom bridge network
docker network create mynetwork
```

The `create` command makes a new isolated network where containers can communicate by name.

```bash
# Run containers on custom network
docker run -d --name web --network mynetwork nginx
docker run -d --name api --network mynetwork myapi
```

**Key benefit:** Containers can now reach each other using container names instead of IP addresses:

```bash
# Inside web container, can reach api by name
docker exec web curl http://api:3000/health
```

### Network with Custom Settings

```bash
# Create network with specific IP range
docker network create \
  --driver bridge \
  --subnet 192.168.1.0/24 \
  --gateway 192.168.1.1 \
  mynetwork
```

**Command breakdown:**
- `--driver bridge` specifies network type (bridge is most common)
- `--subnet 192.168.1.0/24` sets IP address range for containers
- `--gateway 192.168.1.1` sets the gateway IP address

## Network Management Commands

### List and Inspect Networks

```bash
# List all networks
docker network ls

# Get detailed info about a network
docker network inspect mynetwork
```

The `inspect` command shows connected containers, IP addresses, and network configuration.

### Connect/Disconnect Containers

```bash
# Connect running container to network
docker network connect mynetwork existing_container

# Disconnect container from network
docker network disconnect mynetwork existing_container
```

**Use case:** Add a container to multiple networks for controlled access:

```bash
# Container can communicate with both networks
docker network connect frontend api_container
docker network connect backend api_container
```

### Remove Networks

```bash
# Remove specific network (must disconnect containers first)
docker network rm mynetwork

# Remove all unused networks
docker network prune
```

## Practical Examples

### Example 1: Web Application with Database

Create a secure 3-tier architecture:

```bash
# Create networks for different tiers
docker network create frontend    # User-facing services
docker network create backend     # Database and internal services

# Database on backend network only (isolated from users)
docker run -d \
  --name db \
  --network backend \
  -e POSTGRES_PASSWORD=secret123 \
  postgres:13

# API server on both networks (can access DB and serve users)
docker run -d \
  --name api \
  --network frontend \
  myapi

# Connect API to backend network so it can reach database
docker network connect backend api

# Web server on frontend only (can reach API, not DB)
docker run -d \
  --name web \
  --network frontend \
  -p 8080:80 \
  nginx
```

**Security benefit:** Web server cannot directly access database, even if compromised.

**Test connectivity:**
```bash
# API can reach database
docker exec api ping db
# Works! ✅

# Web cannot reach database  
docker exec web ping db
# Fails! ✅ (This is what we want)

# Web can reach API
docker exec web ping api  
# Works! ✅
```

### Example 2: Development Environment

```bash
# Create development network
docker network create dev

# Start all services on same network
docker run -d --name redis --network dev redis:alpine
docker run -d --name postgres --network dev -e POSTGRES_PASSWORD=dev postgres:13
docker run -d --name app --network dev -p 3000:3000 myapp
```

**Benefits for development:**
- All services can reach each other by name
- Easy to add/remove services
- Isolated from other projects

## Networks in Docker Compose

Compose makes network management much easier:

```yaml
version: '3.8'

services:
  web:
    image: nginx
    networks:
      - frontend
    ports:
      - "8080:80"
  
  api:
    image: myapi
    networks:
      - frontend      # Can receive requests from web
      - backend       # Can access database
  
  db:
    image: postgres:13
    networks:
      - backend       # Only accessible by API
    environment:
      POSTGRES_PASSWORD: secret123

networks:
  frontend:           # Declare custom networks
  backend:
```

**Compose automatically:**
- Creates the networks when you run `docker-compose up`
- Connects services to specified networks
- Removes networks when you run `docker-compose down`

### Advanced Compose Networking

```yaml
version: '3.8'

services:
  web:
    image: nginx
    networks:
      frontend:
        aliases:          # Additional names for this service
          - webserver
          - www
  
  api:
    image: myapi  
    networks:
      frontend:
        ipv4_address: 192.168.1.10    # Fixed IP address

networks:
  frontend:
    driver: bridge
    ipam:
      config:
        - subnet: 192.168.1.0/24      # Custom IP range
```

**When to use fixed IPs:**
- Load balancers that expect specific IPs
- Services that need consistent addressing
- Advanced routing configurations

## Network Communication Examples

### Container-to-Container Communication

```bash
# Create network and containers
docker network create appnet
docker run -d --name server --network appnet nginx
docker run -it --name client --network appnet alpine sh
```

**Inside the client container:**
```bash
# Can reach server by name
ping server

# Can access server's web service
wget -O- http://server
# Returns nginx welcome page

# Check network configuration
ip addr show
# Shows container's IP in the appnet range
```

### External Access Control

```bash
# Create internal network (no external access by default)
docker network create --internal internal-net

# Container on internal network
docker run -d --name secret-service --network internal-net myapp

# This container cannot reach the internet
docker exec secret-service ping google.com
# Network is unreachable
```

**Use case:** Highly secure services that should never access external networks.

## Port Publishing vs Network Access

Understanding the difference between `-p` (port publishing) and network connectivity:

```bash
# Port publishing: Makes container accessible from host
docker run -d -p 8080:80 --name web nginx
# Can access via http://localhost:8080

# Network connectivity: Containers talk to each other
docker network create mynet
docker run -d --name web --network mynet nginx  
docker run -d --name app --network mynet myapp
# app can reach web via http://web:80 (no port publishing needed)
```

**Key insight:**
- **Port publishing (-p)** = External access from your computer
- **Networks** = Internal communication between containers

## Troubleshooting Network Issues

### Can't Connect Between Containers

```bash
# Check if containers are on same network
docker inspect container1 | grep NetworkMode
docker inspect container2 | grep NetworkMode

# List networks and connected containers  
docker network inspect network_name
```

**Common fix:**
```bash
# Connect both containers to same network
docker network connect mynetwork container1
docker network connect mynetwork container2
```

### Service Discovery Not Working

```bash
# Test if container can resolve service name
docker exec container1 nslookup container2

# If DNS fails, check network connection
docker network inspect network_name
# Look for both containers in "Containers" section
```

### Port Conflicts

```bash
# Error: Port 8080 already in use
# Check what's using the port
docker ps --filter "publish=8080"

# Use different host port
docker run -p 8081:80 nginx  # Instead of 8080:80
```

### Network Performance Issues

```bash
# Test network performance between containers
docker exec container1 ping -c 10 container2

# Check network statistics
docker exec container1 cat /proc/net/dev
```

## Security Best Practices

### Network Isolation

```bash
# Good: Separate networks for different functions
docker network create web-tier
docker network create app-tier  
docker network create db-tier

# Frontend servers only on web-tier
# Application servers on web-tier + app-tier
# Databases only on db-tier
```

### Least Privilege Access

```yaml
# Only connect services to networks they actually need
services:
  frontend:
    networks:
      - web           # Only needs web access
  
  api:  
    networks:
      - web           # Serves web requests
      - backend       # Accesses database
  
  database:
    networks:
      - backend       # Only backend access
```

### Internal Networks

```bash
# Create network with no internet access
docker network create --internal secure-net

# Containers can communicate with each other but not internet
docker run -d --name secure-app --network secure-net myapp
```

## Network Types and Use Cases

| Network Type | Use Case | External Access | Container Communication |
|--------------|----------|----------------|------------------------|
| **bridge** (default) | General applications | Yes | By IP address |
| **Custom bridge** | Isolated applications | Yes | By name |  
| **host** | High performance | Direct host access | Uses host networking |
| **none** | Security-critical | No network access | None |
| **Internal** | Secure services | No internet access | Internal only |

## Command Reference

```bash
# Network management
docker network create mynet               # Create network
docker network ls                        # List networks  
docker network inspect mynet             # Network details
docker network rm mynet                  # Remove network
docker network prune                     # Remove unused networks

# Container network operations
docker run --network mynet image         # Start container on network
docker network connect mynet container   # Add container to network
docker network disconnect mynet container # Remove from network

# Debugging
docker exec container ping other_container    # Test connectivity
docker exec container nslookup other_container # Test name resolution
docker network inspect mynet                   # See connected containers
```

## What's Next?

Fantastic! You now understand how to create secure, isolated networks and control container communication.

**You've completed the Working with Docker section!** 🎉

**Next up:** [Practical Guides →](../practical-guides/index.md) - Apply everything you've learned with real-world projects, best practices, and troubleshooting techniques.

## Key Takeaways

- **Custom networks** enable container-to-container communication by name instead of IP addresses
- **Network isolation** improves security by controlling which containers can communicate  
- **Multi-network containers** (like APIs) can bridge different network tiers securely
- **Docker Compose** automatically manages networks, making multi-service applications easier
- **Port publishing (-p)** is for external access; networks are for internal container communication
- **Internal networks** prevent internet access while allowing container communication
- Always use **least privilege networking** - only connect containers to networks they actually need