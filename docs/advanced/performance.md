# Docker Performance Optimization

Performance optimization in Docker involves multiple layers: image efficiency, container resource management, runtime tuning, and host system optimization. This guide provides practical techniques to make your containers run faster, use fewer resources, and scale more efficiently.

## Image Performance Optimization

### Base Image Selection

Choose the right base image for your use case:

```dockerfile
# Size and performance comparison
FROM ubuntu:22.04        # ~77MB, full OS features
FROM alpine:3.18         # ~7MB, minimal but compatible
FROM scratch             # ~0MB, static binaries only
FROM gcr.io/distroless/java17-debian11  # ~200MB, secure minimal Java
```

**Performance implications:**
- **Smaller images** = Faster pulls, less network transfer, quicker container starts
- **Alpine Linux** = Minimal overhead but may have musl libc compatibility issues
- **Distroless** = Security + performance, minimal attack surface

### Layer Optimization

Optimize Dockerfile layers for better caching and smaller images:

```dockerfile
# BAD: Each RUN creates a layer
FROM node:18-alpine
RUN apk add --no-cache git
RUN apk add --no-cache python3
RUN apk add --no-cache make
RUN apk add --no-cache g++

# GOOD: Combined into single layer
FROM node:18-alpine
RUN apk add --no-cache \
    git \
    python3 \
    make \
    g++ \
    && rm -rf /var/cache/apk/*

# BETTER: Multi-stage to eliminate build dependencies
FROM node:18-alpine AS build
RUN apk add --no-cache git python3 make g++
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
CMD ["node", "dist/server.js"]
```

### Dependency Management

Optimize package installation for performance:

```dockerfile
# Node.js optimization
FROM node:18-alpine
WORKDIR /app

# Copy dependency files first for better caching
COPY package*.json ./

# Use npm ci for faster, reliable installs
RUN npm ci --only=production \
    && npm cache clean --force \
    && rm -rf /tmp/*

COPY . .
CMD ["node", "server.js"]
```

```dockerfile
# Python optimization
FROM python:3.11-slim
WORKDIR /app

# Install system dependencies in one layer
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        build-essential \
        libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements first for better caching
COPY requirements.txt .

# Use pip cache and optimized flags
RUN pip install --no-cache-dir \
    --disable-pip-version-check \
    --requirement requirements.txt

COPY . .
CMD ["python", "app.py"]
```

## Container Resource Management

### Memory Optimization

Set appropriate memory limits and understand memory usage:

```bash
# Set memory limits
docker run -d \
  --memory="512m" \
  --memory-swap="512m" \
  --oom-kill-disable=false \
  myapp:latest

# Monitor memory usage
docker stats --no-stream
docker exec container-name cat /sys/fs/cgroup/memory/memory.usage_in_bytes
```

**Application-level memory optimization:**

```javascript
// Node.js memory tuning
const app = express();

// Set appropriate heap size
process.env.NODE_OPTIONS = '--max-old-space-size=400';  // 400MB heap

// Memory monitoring
setInterval(() => {
  const memUsage = process.memoryUsage();
  console.log({
    rss: Math.round(memUsage.rss / 1024 / 1024) + 'MB',
    heapTotal: Math.round(memUsage.heapTotal / 1024 / 1024) + 'MB',
    heapUsed: Math.round(memUsage.heapUsed / 1024 / 1024) + 'MB'
  });
}, 60000);
```

### CPU Optimization

Configure CPU limits and scaling:

```bash
# CPU limits
docker run -d \
  --cpus="1.5" \
  --cpu-shares=1024 \
  --cpuset-cpus="0,1" \
  myapp:latest
```

**Docker Compose resource management:**

```yaml
version: '3.8'
services:
  web:
    image: myapp:latest
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 1G
        reservations:
          cpus: '0.5'
          memory: 512M
      replicas: 3
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
```

### I/O Performance

Optimize disk and network I/O:

```dockerfile
# Use tmpfs for temporary files
FROM node:18-alpine
WORKDIR /app
COPY . .

# Configure for better I/O
RUN npm ci --only=production

# Use tmpfs mount for temporary files
VOLUME ["/tmp"]

CMD ["node", "server.js"]
```

```bash
# Run with optimized I/O
docker run -d \
  --tmpfs /tmp:noexec,nosuid,size=100m \
  --device-read-bps /dev/sda:50mb \
  --device-write-bps /dev/sda:50mb \
  myapp:latest
```

## Runtime Performance Tuning

### JVM Applications

Optimize Java applications for containerized environments:

```dockerfile
FROM openjdk:17-jre-slim

# JVM tuning for containers
ENV JAVA_OPTS="-XX:+UseContainerSupport \
               -XX:MaxRAMPercentage=75.0 \
               -XX:+UseG1GC \
               -XX:+UseStringDeduplication \
               -XX:+ExitOnOutOfMemoryError"

COPY app.jar /app.jar

# Use exec form to ensure proper signal handling
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar /app.jar"]
```

**Advanced JVM tuning:**

```dockerfile
FROM openjdk:17-jre-slim

# Container-aware JVM settings
ENV JAVA_OPTS="-server \
               -XX:+UseContainerSupport \
               -XX:InitialRAMPercentage=50.0 \
               -XX:MaxRAMPercentage=80.0 \
               -XX:+UseG1GC \
               -XX:MaxGCPauseMillis=100 \
               -XX:+UseStringDeduplication \
               -XX:+OptimizeStringConcat \
               -XX:+UseCompressedOops \
               -XX:+ExitOnOutOfMemoryError \
               -Djava.security.egd=file:/dev/./urandom"

# Application profiling
ENV PROFILING_OPTS="-XX:+FlightRecorder \
                   -XX:StartFlightRecording=duration=60s,filename=/tmp/flight.jfr \
                   -XX:+UnlockDiagnosticVMOptions \
                   -XX:+DebugNonSafepoints"

COPY app.jar /app.jar
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS $PROFILING_OPTS -jar /app.jar"]
```

### Node.js Applications

Optimize Node.js for production performance:

```dockerfile
FROM node:18-alpine
WORKDIR /app

# Node.js performance environment variables
ENV NODE_ENV=production
ENV NODE_OPTIONS="--max-old-space-size=460 --optimize-for-size"

COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

COPY . .

# Use cluster mode for CPU utilization
CMD ["node", "cluster.js"]
```

**Cluster implementation:**

```javascript
// cluster.js
const cluster = require('cluster');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  console.log(`Master ${process.pid} is running`);
  
  // Fork workers
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died`);
    cluster.fork();  // Restart worker
  });
} else {
  // Workers can share any TCP port
  require('./server.js');
  console.log(`Worker ${process.pid} started`);
}
```

### Python Applications

Optimize Python applications for container performance:

```dockerfile
FROM python:3.11-slim

# Python performance optimizations
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONHASHSEED=random \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

WORKDIR /app

# Use multiple workers
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

# Use gunicorn with multiple workers
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "4", "--worker-class", "gevent", "app:app"]
```

## Build Performance Optimization

### Docker BuildKit

Enable BuildKit for faster builds:

```bash
export DOCKER_BUILDKIT=1
docker build -t myapp .

# Or use buildx
docker buildx build --platform linux/amd64,linux/arm64 -t myapp .
```

**BuildKit advanced features:**

```dockerfile
# syntax=docker/dockerfile:1.4
FROM node:18-alpine

# Use cache mounts
RUN --mount=type=cache,target=/root/.npm \
    npm install

# Use bind mounts for build context
RUN --mount=type=bind,source=package.json,target=package.json \
    --mount=type=bind,source=package-lock.json,target=package-lock.json \
    npm ci

# Use secrets for build-time credentials
RUN --mount=type=secret,id=npmrc,target=/root/.npmrc \
    npm install private-package
```

### Build Caching Strategies

Optimize layer caching for faster builds:

```dockerfile
FROM node:18-alpine

# Install system dependencies first (rarely change)
RUN apk add --no-cache git python3 make g++

WORKDIR /app

# Copy dependency files (change less frequently)
COPY package*.json ./
RUN npm ci --only=production

# Copy source code last (changes most frequently)
COPY . .

# Build application
RUN npm run build

CMD ["npm", "start"]
```

### Multi-platform Builds

Build for multiple architectures efficiently:

```bash
# Create and use multi-platform builder
docker buildx create --name multiplatform --use
docker buildx inspect --bootstrap

# Build for multiple platforms
docker buildx build \
  --platform linux/amd64,linux/arm64,linux/arm/v7 \
  --tag myapp:latest \
  --push .
```

## Monitoring and Profiling

### Container Metrics Collection

Monitor container performance:

```bash
# Real-time resource monitoring
docker stats

# Historical data with cAdvisor
docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:rw \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  google/cadvisor:latest
```

### Application Performance Monitoring

Implement application-level monitoring:

```javascript
// Node.js APM with Prometheus
const promClient = require('prom-client');
const express = require('express');
const app = express();

// Create performance metrics
const httpDuration = new promClient.Histogram({
  name: 'http_request_duration_ms',
  help: 'Duration of HTTP requests in ms',
  labelNames: ['method', 'route', 'status']
});

const memoryUsage = new promClient.Gauge({
  name: 'nodejs_memory_usage_bytes',
  help: 'Memory usage in bytes',
  labelNames: ['type']
});

// Update memory metrics
setInterval(() => {
  const usage = process.memoryUsage();
  memoryUsage.set({ type: 'rss' }, usage.rss);
  memoryUsage.set({ type: 'heapTotal' }, usage.heapTotal);
  memoryUsage.set({ type: 'heapUsed' }, usage.heapUsed);
}, 5000);

// Metrics endpoint
app.get('/metrics', (req, res) => {
  res.set('Content-Type', promClient.register.contentType);
  res.end(promClient.register.metrics());
});
```

## Performance Testing and Benchmarking

### Load Testing Containers

Test container performance under load:

```bash
# Use Apache Bench for simple HTTP load testing
docker run --rm \
  --network container:myapp-container \
  httpd:2.4 \
  ab -n 10000 -c 100 http://localhost:3000/

# Use wrk for more sophisticated testing
docker run --rm \
  --network container:myapp-container \
  williamyeh/wrk \
  -t12 -c400 -d30s --timeout 2s http://localhost:3000/
```

### Benchmark Container Startup

Measure container startup performance:

```bash
#!/bin/bash
# benchmark-startup.sh

IMAGE_NAME="myapp:latest"
ITERATIONS=10

echo "Benchmarking container startup for $IMAGE_NAME"

total_time=0
for i in $(seq 1 $ITERATIONS); do
    start_time=$(date +%s.%3N)
    
    container_id=$(docker run -d $IMAGE_NAME)
    
    # Wait for container to be ready
    while ! docker exec $container_id curl -f http://localhost:3000/health 2>/dev/null; do
        sleep 0.1
    done
    
    end_time=$(date +%s.%3N)
    startup_time=$(echo "$end_time - $start_time" | bc)
    total_time=$(echo "$total_time + $startup_time" | bc)
    
    echo "Iteration $i: ${startup_time}s"
    
    docker stop $container_id > /dev/null
    docker rm $container_id > /dev/null
done

average_time=$(echo "scale=3; $total_time / $ITERATIONS" | bc)
echo "Average startup time: ${average_time}s"
```

## Optimization Best Practices

### Image Optimization Checklist

- [ ] Use minimal base images (Alpine, distroless)
- [ ] Implement multi-stage builds
- [ ] Combine RUN commands to reduce layers
- [ ] Use .dockerignore to exclude unnecessary files
- [ ] Remove package managers and build tools from final stage
- [ ] Set appropriate USER directive
- [ ] Use COPY instead of ADD when possible

### Runtime Optimization Checklist

- [ ] Set appropriate memory and CPU limits
- [ ] Configure application for container environment
- [ ] Use read-only filesystems where possible
- [ ] Implement proper health checks
- [ ] Configure logging appropriately
- [ ] Use tmpfs for temporary files
- [ ] Enable container security features

### Monitoring and Debugging

- [ ] Implement application metrics
- [ ] Set up container monitoring
- [ ] Configure structured logging
- [ ] Use debugging tools (strace, perf)
- [ ] Monitor resource utilization
- [ ] Set up alerting for performance degradation

## Performance Anti-patterns

### Common Mistakes to Avoid

**Running unnecessary services:**
```dockerfile
# BAD: Full operating system with services
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y \
    systemd \
    ssh \
    cron \
    syslog-ng

# GOOD: Minimal runtime
FROM node:18-alpine
WORKDIR /app
COPY . .
CMD ["node", "server.js"]
```

**Inefficient layer caching:**
```dockerfile
# BAD: Source code copied early
FROM node:18
COPY . .
RUN npm install

# GOOD: Dependencies cached separately
FROM node:18
COPY package*.json ./
RUN npm ci
COPY . .
```

**Resource waste:**
```bash
# BAD: No resource limits
docker run -d myapp:latest

# GOOD: Appropriate limits
docker run -d \
  --memory="512m" \
  --cpus="1.0" \
  myapp:latest
```

Performance optimization is an iterative process. Start with the fundamentals (appropriate base images, resource limits), measure performance, and optimize based on actual bottlenecks rather than assumptions. The techniques in this guide provide a foundation for building efficient, scalable containerized applications.