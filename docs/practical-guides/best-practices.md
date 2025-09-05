# Best Practices

These are the professional standards that separate hobby Docker users from production-ready developers. Following these practices will make your applications more secure, performant, and maintainable.

## Dockerfile Best Practices

### Use Specific Base Images

**❌ Bad - Unpredictable:**
```dockerfile
FROM python
```

**✅ Good - Specific and reliable:**
```dockerfile
FROM python:3.11-alpine
```

**Why this matters:**
- `python` tag changes over time (could be 3.9 today, 3.12 tomorrow)
- `python:3.11-alpine` is locked to specific Python version on lightweight Alpine Linux
- `alpine` variants are smaller and more secure than full distributions

**Command to check image size:**
```bash
# Compare image sizes
docker images python:latest     # Usually 800MB+
docker images python:3.11-alpine  # Usually 50MB
```

The `docker images` command shows size in the SIZE column - always prefer smaller, more specific images.

### Optimize Layer Caching

**❌ Bad - Breaks caching:**
```dockerfile
FROM python:3.11-alpine
COPY . /app                    # Changes every time you edit code
RUN pip install -r requirements.txt  # Reinstalls every time
```

**✅ Good - Efficient caching:**
```dockerfile
FROM python:3.11-alpine

# Copy requirements first (changes less frequently)
COPY requirements.txt /app/requirements.txt
WORKDIR /app
RUN pip install -r requirements.txt

# Copy code last (changes most frequently)  
COPY . /app
```

**Why layer order matters:**
- Docker caches each layer (RUN, COPY, etc.)
- If a layer changes, all subsequent layers rebuild
- Dependencies change less often than application code
- By copying requirements first, Docker reuses the pip install layer when only code changes

**Test this caching:**
```bash
# First build (downloads and installs everything)
docker build -t myapp .

# Edit your application code (not requirements.txt)
# Second build (reuses pip install layer)
docker build -t myapp .
# Notice "CACHED" appears for the pip install step
```

### Combine RUN Commands

**❌ Bad - Many layers:**
```dockerfile
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y git
RUN apt-get clean
```

**✅ Good - Single layer:**
```dockerfile
RUN apt-get update && \
    apt-get install -y \
        curl \
        git \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*
```

**Command breakdown:**
- `&&` chains commands - if any fails, the whole RUN fails
- `\` continues the command on the next line for readability
- `apt-get clean` removes package cache files
- `rm -rf /var/lib/apt/lists/*` removes apt repository lists
- Each `RUN` creates a new layer, so fewer `RUN` commands = smaller images

**Check layer count:**
```bash
# See how many layers your image has
docker history myapp
```

### Create Non-Root Users

**❌ Bad - Security risk:**
```dockerfile
FROM python:3.11-alpine
# Runs as root by default (dangerous!)
```

**✅ Good - Secure:**
```dockerfile
FROM python:3.11-alpine

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001 -G nodejs

# Switch to non-root user
USER nextjs
```

**Command explanation:**
- `addgroup -g 1001 -S nodejs` creates group with ID 1001 (system group)
- `adduser -S nextjs -u 1001 -G nodejs` creates user with ID 1001 in the nodejs group
- `USER nextjs` switches to non-root user for all subsequent commands

**Test user security:**
```bash
# Check what user the container runs as
docker run myapp whoami
# Should show: nextjs (not root)

# Verify limited permissions
docker run myapp id
# Shows user and group IDs
```

### Use Multi-Stage Builds

**Problem:** Build tools make production images huge and insecure.

**✅ Solution - Multi-stage build:**
```dockerfile
# Stage 1: Build stage (includes dev tools)
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Stage 2: Production stage (minimal)  
FROM node:18-alpine AS production
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001 -G nodejs

# Copy only built application (no dev tools)
COPY --from=builder --chown=nextjs:nodejs /app /app
USER nextjs
WORKDIR /app
CMD ["npm", "start"]
```

**Command breakdown:**
- `FROM node:18-alpine AS builder` names this stage "builder"
- `COPY --from=builder` copies files from the builder stage only
- `--chown=nextjs:nodejs` sets file ownership during copy
- Final image only contains production files, not build tools

**Build multi-stage image:**
```bash
# Build only production stage (default)
docker build -t myapp .

# Build specific stage for development
docker build --target builder -t myapp:dev .
```

## Security Best Practices

### Scan Images for Vulnerabilities

**Always scan images before production:**
```bash
# Scan image for known vulnerabilities
docker scout cves myapp:latest
```

**What this command does:**
- `docker scout` is Docker's security scanning tool
- `cves` looks for Common Vulnerabilities and Exposures
- Shows severity levels (CRITICAL, HIGH, MEDIUM, LOW)

**Alternative scanning with Trivy:**
```bash
# Install Trivy scanner (one-time setup)
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  -v $HOME/Library/Caches:/root/.cache/ \
  aquasec/trivy image myapp:latest
```

**Command explanation:**
- `-v /var/run/docker.sock:/var/run/docker.sock` gives Trivy access to Docker
- `-v $HOME/Library/Caches:/root/.cache/` caches vulnerability database locally
- `aquasec/trivy image` scans the specified image

### Manage Secrets Properly

**❌ Bad - Secrets in Dockerfile:**
```dockerfile
ENV API_KEY=secret123  # Visible in image history!
```

**✅ Good - Runtime secrets:**
```bash
# Pass secrets at runtime, not build time
docker run -e API_KEY=secret123 myapp

# Or use Docker secrets (swarm mode)
echo "secret123" | docker secret create api_key -
docker service create --secret api_key myapp
```

**Why build-time secrets are dangerous:**
```bash
# Anyone can see secrets in image history
docker history myapp
# Shows all ENV commands with values!
```

**Better secret management:**
```bash
# Use environment files (keep out of version control)
echo "API_KEY=secret123" > .env
docker run --env-file .env myapp

# Mount secrets as files (more secure)
docker run -v ./secrets:/run/secrets myapp
```

### Limit Container Resources

**Always set resource limits:**
```bash
# Limit memory to prevent OOM killing other processes
docker run --memory=512m myapp

# Limit CPU to prevent monopolizing host
docker run --cpus="1.5" myapp

# Combine limits for production
docker run \
  --memory=1g \
  --cpus="2.0" \
  --restart=unless-stopped \
  myapp
```

**Command breakdown:**
- `--memory=512m` limits container to 512 megabytes of RAM
- `--cpus="1.5"` limits container to 1.5 CPU cores worth of processing
- `--restart=unless-stopped` automatically restarts if container crashes

**Monitor resource usage:**
```bash
# Watch real-time resource usage
docker stats myapp

# Get current resource usage once
docker stats --no-stream myapp
```

## Performance Optimization

### Minimize Image Size

**Use .dockerignore:**
```bash
# Create .dockerignore file
cat > .dockerignore << 'EOF'
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.nyc_output
coverage
.vscode
EOF
```

**What .dockerignore does:**
- Works like .gitignore but for Docker builds
- Prevents unnecessary files from being sent to Docker daemon
- Reduces build context size and build time
- Files listed here won't be available for COPY commands

**Test .dockerignore effectiveness:**
```bash
# Check build context size
docker build --no-cache -t myapp . 2>&1 | grep "Sending build context"
# Should show smaller context after adding .dockerignore
```

### Optimize Package Installation

**For Python applications:**
```dockerfile
# Use pip cache and avoid unnecessary files
RUN pip install --no-cache-dir \
    --user \
    --no-warn-script-location \
    -r requirements.txt
```

**Command breakdown:**
- `--no-cache-dir` prevents pip from storing cache (saves space)
- `--user` installs to user directory (works with non-root user)
- `--no-warn-script-location` suppresses warnings about script paths

**For Node.js applications:**
```dockerfile
# Use npm ci for faster, reliable installations
COPY package*.json ./
RUN npm ci --only=production --silent

# Alternative: Use npm cache mount (newer Docker versions)
RUN --mount=type=cache,target=/root/.npm \
    npm ci --only=production
```

**What `npm ci` does:**
- Faster than `npm install` (up to 2x speed improvement)
- Uses package-lock.json for consistent installs
- `--only=production` skips dev dependencies
- `--silent` reduces build output noise

### Leverage Build Cache

**Use BuildKit for advanced caching:**
```bash
# Enable BuildKit (faster builds)
export DOCKER_BUILDKIT=1

# Build with cache mount
docker build -t myapp .
```

**Cache mount example in Dockerfile:**
```dockerfile
# Cache package managers
RUN --mount=type=cache,target=/var/cache/apt \
    --mount=type=cache,target=/var/lib/apt \
    apt-get update && apt-get install -y git

# Cache npm installs
RUN --mount=type=cache,target=/root/.npm \
    npm ci --only=production
```

**What cache mounts do:**
- `--mount=type=cache` creates persistent cache between builds
- Package manager caches persist across different image builds
- Dramatically speeds up dependency installation

## Production Deployment

### Health Checks

**Add health checks to your applications:**
```dockerfile
# Simple HTTP health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```

**Health check parameters explained:**
- `--interval=30s` checks every 30 seconds
- `--timeout=3s` fails if check takes longer than 3 seconds
- `--start-period=5s` grace period before first check
- `--retries=3` marks unhealthy after 3 consecutive failures

**Test health checks:**
```bash
# Run container and monitor health
docker run -d --name myapp myapp

# Check health status
docker ps
# Look for "(healthy)" or "(unhealthy)" in STATUS column

# See health check details
docker inspect myapp | grep -A 10 "Health"
```

### Logging Configuration

**Configure logging for production:**
```bash
# Limit log file size and rotation
docker run -d \
  --log-driver json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  myapp
```

**Logging options explained:**
- `--log-driver json-file` uses JSON format logs (default)
- `--log-opt max-size=10m` limits each log file to 10 megabytes
- `--log-opt max-file=3` keeps maximum 3 rotated log files

**View and manage logs:**
```bash
# View recent logs
docker logs --tail 100 myapp

# Follow logs in real-time
docker logs -f myapp

# View logs with timestamps
docker logs -t myapp

# Check log file sizes
docker inspect myapp | grep LogPath
# Shows actual log file location
```

### Environment-Specific Configurations

**Use Docker Compose overrides:**

**Base configuration (docker-compose.yml):**
```yaml
version: '3.8'
services:
  app:
    image: myapp:latest
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
```

**Development override (docker-compose.override.yml):**
```yaml
version: '3.8'
services:
  app:
    build: .
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - DEBUG=true
```

**How overrides work:**
```bash
# Development (uses override automatically)
docker-compose up
# Combines base + override files

# Production (ignores override)
docker-compose -f docker-compose.yml up
# Uses only base configuration
```

## Monitoring and Maintenance

### Container Resource Monitoring

**Set up monitoring with docker stats:**
```bash
# Monitor all running containers
docker stats

# Monitor specific containers with custom format
docker stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}"
```

**Format options explained:**
- `{{.Container}}` shows container name
- `{{.CPUPerc}}` shows CPU percentage usage
- `{{.MemUsage}}` shows memory usage (used/available)
- `{{.NetIO}}` shows network input/output

**Set up alerts for resource usage:**
```bash
# Simple script to alert on high memory usage
#!/bin/bash
MEMORY_THRESHOLD=80

docker stats --no-stream --format "{{.Container}}\t{{.MemPerc}}" | while read line; do
    container=$(echo $line | cut -f1)
    memory=$(echo $line | cut -f2 | sed 's/%//')
    
    if [ "$memory" -gt "$MEMORY_THRESHOLD" ]; then
        echo "ALERT: $container using ${memory}% memory"
        # Add notification logic here (email, Slack, etc.)
    fi
done
```

### Regular Maintenance

**Clean up unused resources regularly:**
```bash
# Remove stopped containers
docker container prune

# Remove unused images
docker image prune

# Remove unused volumes (BE CAREFUL!)
docker volume prune

# Remove everything unused (VERY CAREFUL!)
docker system prune -a
```

**Scheduled cleanup script:**
```bash
#!/bin/bash
# cleanup.sh - Run weekly via cron

echo "Starting Docker cleanup at $(date)"

# Remove containers stopped more than 24 hours ago
docker container prune --filter "until=24h" -f

# Remove images not used in 7 days
docker image prune --filter "until=168h" -f

# Show disk usage after cleanup
docker system df

echo "Cleanup completed at $(date)"
```

**Set up cron job for cleanup:**
```bash
# Edit crontab
crontab -e

# Add weekly cleanup (every Sunday at 2 AM)
0 2 * * 0 /path/to/cleanup.sh >> /var/log/docker-cleanup.log 2>&1
```

## Development Workflow Best Practices

### Local Development Setup

**Create consistent development environments:**
```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  app:
    build: .
    volumes:
      - .:/app
      - /app/node_modules
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - DEBUG=*
    command: npm run dev

  db:
    image: postgres:13
    environment:
      - POSTGRES_PASSWORD=dev_password
      - POSTGRES_DB=myapp_dev
    ports:
      - "5432:5432"
    volumes:
      - dev_db_data:/var/lib/postgresql/data

volumes:
  dev_db_data:
```

**Start development environment:**
```bash
# Start with development configuration
docker-compose -f docker-compose.dev.yml up -d

# View logs from all services
docker-compose -f docker-compose.dev.yml logs -f

# Stop and clean up
docker-compose -f docker-compose.dev.yml down -v
```

**Command breakdown:**
- `-f docker-compose.dev.yml` specifies configuration file
- `up -d` starts services in background
- `logs -f` follows logs from all services
- `down -v` stops services and removes volumes

## Quick Reference Checklist

### Before Production Deployment

- [ ] **Security scanned:** `docker scout cves myimage`
- [ ] **Non-root user:** `USER` directive in Dockerfile
- [ ] **Resource limits:** `--memory` and `--cpus` flags set
- [ ] **Health checks:** `HEALTHCHECK` configured
- [ ] **Logging configured:** Log rotation and size limits
- [ ] **Secrets secure:** No secrets in image history
- [ ] **Image optimized:** Multi-stage build, minimal base image

### Performance Checklist

- [ ] **Specific base images:** No `latest` tags
- [ ] **Layer caching optimized:** Dependencies before application code
- [ ] **Combined RUN commands:** Fewer layers
- [ ] **.dockerignore present:** Exclude unnecessary files
- [ ] **Package cache management:** `--no-cache-dir` flags used

### Maintenance Checklist

- [ ] **Regular cleanup:** Automated pruning scheduled
- [ ] **Monitoring setup:** Resource usage tracked
- [ ] **Log rotation:** Size and retention configured
- [ ] **Update schedule:** Base images updated regularly
- [ ] **Backup strategy:** Volume data protected

## Key Takeaways

- **Use specific, minimal base images** - `python:3.11-alpine` instead of `python`
- **Optimize layer caching** - Copy dependencies before application code
- **Never run as root in production** - Create and use non-root users
- **Scan for vulnerabilities** - Use `docker scout` or Trivy before deployment
- **Set resource limits** - Prevent containers from consuming all host resources
- **Implement health checks** - Monitor application health automatically
- **Clean up regularly** - Remove unused images, containers, and volumes
- **Use multi-stage builds** - Keep production images minimal and secure

**Next:** [Fun Examples →](fun-examples.md) - Apply these best practices while building impressive real-world applications!