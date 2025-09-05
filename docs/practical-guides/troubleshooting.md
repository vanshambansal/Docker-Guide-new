# Troubleshooting

This is your go-to guide for solving any Docker problem! Instead of getting frustrated, you'll have systematic approaches to diagnose and fix issues quickly.

## The Docker Detective Method

When something goes wrong, follow this systematic approach:

### 1. 🔍 **Gather Information**
```bash
# Check what's running
docker ps -a

# See recent events
docker system events --since 5m

# Check Docker daemon status
docker version
```

### 2. 🕵️ **Investigate Logs**
```bash
# Container logs (most important!)
docker logs container_name

# Follow logs in real-time
docker logs -f container_name

# Get last 50 lines
docker logs --tail 50 container_name
```

### 3. 🩺 **Health Check**
```bash
# Check resource usage
docker stats container_name

# Inspect configuration
docker inspect container_name

# Check network connectivity
docker exec container_name ping google.com
```

---

## Common Problems & Solutions

### 🚨 Container Won't Start

**Symptoms:** Container exits immediately or fails to start

**Investigation commands:**
```bash
# Check container status
docker ps -a

# Look at logs
docker logs container_name

# Try running interactively
docker run -it image_name bash
```

**Common causes and fixes:**

**❌ Missing environment variables:**
```bash
# Error: database connection failed
# Solution: Add required environment variables
docker run -e DB_HOST=localhost -e DB_PORT=5432 myapp
```

**❌ Port already in use:**
```bash
# Error: port 8080 already allocated
# Check what's using the port
netstat -tulpn | grep 8080
# or
lsof -i :8080

# Solution: Use different port
docker run -p 8081:80 nginx
```

**❌ Wrong command or entrypoint:**
```bash
# Check what command container is trying to run
docker inspect image_name --format='{{.Config.Cmd}}'

# Override command to debug
docker run -it image_name bash
```

### 🔗 Container Connectivity Issues

**Symptoms:** Services can't reach each other, API calls fail

**Investigation commands:**
```bash
# Test connectivity between containers
docker exec container1 ping container2

# Check if service is listening on correct port
docker exec container netstat -tulpn

# Test from outside
curl http://localhost:8080/health
```

**Common solutions:**

**❌ Containers on different networks:**
```bash
# Check container networks
docker inspect container_name | grep NetworkMode

# Put containers on same network
docker network create mynetwork
docker run --network mynetwork container1
docker run --network mynetwork container2
```

**❌ Service binding to localhost only:**
```bash
# Bad: service only accepts local connections
server.listen(3000, 'localhost')  # Only accepts 127.0.0.1

# Good: accept connections from any interface  
server.listen(3000, '0.0.0.0')    # Accepts external connections
```

**❌ Firewall or port mapping issues:**
```bash
# Check port mapping
docker port container_name

# Make sure container port is exposed
docker run -p 8080:3000 myapp  # host:container
```

### 💾 Data Not Persisting

**Symptoms:** Data disappears when container restarts

**Investigation commands:**
```bash
# Check if volumes are mounted
docker inspect container_name | grep Mounts

# List all volumes
docker volume ls

# Check volume contents
docker run --rm -v volume_name:/data alpine ls -la /data
```

**Common solutions:**

**❌ No volume mounted:**
```bash
# Bad: data stored in container filesystem
docker run postgres

# Good: data stored in volume
docker run -v postgres_data:/var/lib/postgresql/data postgres
```

**❌ Wrong mount path:**
```bash
# Check application's data directory
# PostgreSQL uses: /var/lib/postgresql/data
# MySQL uses: /var/lib/mysql
# MongoDB uses: /data/db

# Mount volume to correct path
docker run -v mydata:/var/lib/postgresql/data postgres
```

### 🏗️ Image Build Failures

**Symptoms:** `docker build` command fails

**Investigation commands:**
```bash
# Build with verbose output
docker build --progress=plain --no-cache -t myapp .

# Check build context size
docker build . 2>&1 | grep "Sending build context"

# Verify Dockerfile syntax
docker build --dry-run -t myapp .
```

**Common solutions:**

**❌ Missing files in build context:**
```bash
# Error: COPY failed: file not found
# Solution: Make sure file exists and isn't in .dockerignore

# Check what's being sent to Docker
ls -la

# Check .dockerignore
cat .dockerignore
```

**❌ Wrong base image or version:**
```bash
# Error: unable to resolve image name
# Solution: Use correct image name and tag

# Bad
FROM nodejs:latest  # Wrong name

# Good  
FROM node:18-alpine  # Correct name
```

**❌ Dependency installation fails:**
```bash
# Error during npm install or pip install
# Solution: Clear cache and try again

# For Node.js
RUN npm ci --cache /tmp/empty-cache

# For Python
RUN pip install --no-cache-dir -r requirements.txt
```

### 🐌 Performance Issues

**Symptoms:** Containers running slowly, high resource usage

**Investigation commands:**
```bash
# Check resource usage
docker stats

# See detailed container info
docker system df

# Check host system resources
htop
df -h
```

**Common solutions:**

**❌ No resource limits set:**
```bash
# Set memory and CPU limits
docker run --memory=512m --cpus="1.0" myapp

# In Docker Compose
services:
  app:
    image: myapp
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: '1.0'
```

**❌ Large images:**
```bash
# Check image sizes
docker images

# Use smaller base images
FROM python:3.9-slim  # Instead of python:3.9
FROM node:18-alpine   # Instead of node:18
```

**❌ Too many layers:**
```bash
# Combine RUN commands to reduce layers
RUN apt-get update && \
    apt-get install -y curl git && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

---

## Docker Compose Issues

### Service Dependencies

**Problem:** Services start in wrong order

**Solution:** Use `depends_on` and health checks
```yaml
services:
  web:
    image: nginx
    depends_on:
      db:
        condition: service_healthy
  
  db:
    image: postgres
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 30s
      timeout: 10s
      retries: 3
```

### Environment Variable Issues

**Problem:** Variables not being passed correctly

**Investigation:**
```bash
# Check if .env file exists
ls -la .env

# Check what variables are set in container
docker-compose exec service_name env
```

**Solutions:**
```yaml
# Method 1: Direct in compose file
services:
  app:
    environment:
      - API_KEY=secret123
      - DEBUG=true

# Method 2: From .env file
services:
  app:
    env_file:
      - .env

# Method 3: Use environment substitution
services:
  app:
    environment:
      - API_KEY=${API_KEY}
```

---

## Debug Tools and Commands

### Essential Debug Commands

```bash
# Get shell inside running container
docker exec -it container_name bash

# Run one-off command
docker exec container_name ls -la /app

# Copy files from container
docker cp container_name:/app/logs.txt ./logs.txt

# Copy files to container
docker cp ./config.json container_name:/app/config.json

# Check container processes
docker exec container_name ps aux

# Check container filesystem
docker exec container_name df -h
```

### Network Debugging

```bash
# List all networks
docker network ls

# Inspect network configuration
docker network inspect network_name

# Test DNS resolution
docker exec container_name nslookup service_name

# Test connectivity with netcat
docker exec container_name nc -zv hostname port

# Check listening ports
docker exec container_name netstat -tulpn
```

### Volume Debugging

```bash
# Inspect volume details
docker volume inspect volume_name

# Browse volume contents
docker run --rm -v volume_name:/data alpine ls -la /data

# Backup volume
docker run --rm -v volume_name:/data -v $(pwd):/backup alpine \
  tar czf /backup/backup.tar.gz -C /data .

# Restore volume
docker run --rm -v volume_name:/data -v $(pwd):/backup alpine \
  tar xzf /backup/backup.tar.gz -C /data
```

### Resource Monitoring

```bash
# Real-time resource usage
docker stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}"

# Container resource limits
docker inspect container_name | grep -A 10 "Memory\|Cpu"

# System-wide resource usage
docker system df

# Cleanup unused resources
docker system prune
```

---

## Performance Optimization

### Image Optimization

```bash
# Analyze image layers
docker history image_name

# Use multi-stage builds
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:18-alpine
COPY --from=builder /app/node_modules ./node_modules
COPY . .
CMD ["npm", "start"]

# Use .dockerignore
echo "node_modules
.git
*.log" > .dockerignore
```

### Container Performance

```bash
# Set appropriate resource limits
docker run --memory=512m --cpus="1.0" --restart=unless-stopped myapp

# Use health checks
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost/ || exit 1

# Monitor and adjust
docker stats myapp
```

---

## Security Troubleshooting

### Permission Issues

```bash
# Check file permissions
docker exec container_name ls -la /app

# Run as specific user
docker run -u 1000:1000 myapp

# Fix ownership in running container
docker exec -u root container_name chown -R 1000:1000 /app
```

### Security Scanning

```bash
# Scan image for vulnerabilities
docker scout cves image_name

# Check running containers for security issues
docker scout recommendations image_name
```

---

## Emergency Recovery

### Container Recovery

```bash
# Container won't start - get emergency shell
docker run --rm -it --entrypoint bash image_name

# Recover data from stopped container
docker cp stopped_container:/data ./recovered_data

# Start container with different command
docker run -it image_name bash
```

### System Recovery

```bash
# Free up disk space
docker system prune -a

# Stop all containers
docker stop $(docker ps -q)

# Remove all containers
docker rm $(docker ps -aq)

# Clean everything (DANGEROUS!)
docker system prune -a --volumes
```

---

## Troubleshooting Checklist

When facing any Docker issue:

### ✅ **Quick Checks**
- [ ] Is Docker daemon running? (`docker version`)
- [ ] Check container status (`docker ps -a`)
- [ ] Look at logs (`docker logs container_name`)
- [ ] Check resource usage (`docker stats`)

### ✅ **Network Issues**
- [ ] Are containers on the same network?
- [ ] Is the service binding to 0.0.0.0 (not localhost)?
- [ ] Are ports properly mapped?
- [ ] Can containers ping each other?

### ✅ **Data Issues**  
- [ ] Are volumes properly mounted?
- [ ] Is the mount path correct for the application?
- [ ] Do file permissions allow access?
- [ ] Is there enough disk space?

### ✅ **Performance Issues**
- [ ] Are resource limits set appropriately?
- [ ] Is the base image optimized?
- [ ] Are there too many layers in the image?
- [ ] Is the host system healthy?

## Key Takeaways

- **Always check logs first** - `docker logs` solves 80% of issues
- **Use systematic debugging** - Don't guess, investigate step by step
- **Test connectivity** - `docker exec` and `ping` are your friends
- **Monitor resources** - `docker stats` shows what's really happening
- **Set proper limits** - Prevent containers from consuming all resources
- **Use health checks** - Know when services are actually ready
- **Keep images small** - Faster builds, deploys, and debugging

**Remember:** Every Docker expert started as a beginner who learned to debug systematically. These tools and techniques will make you unstoppable!

**Next:** You've completed the Practical Guides! Ready for [Advanced Topics →](../advanced/index.md)?