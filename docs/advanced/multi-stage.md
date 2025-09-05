# Multi-stage Docker Builds

Multi-stage builds are one of the most impactful techniques for creating production-ready Docker images. They allow you to separate your build environment from your runtime environment, resulting in smaller, more secure, and faster-deploying containers.

## The Problem Multi-stage Builds Solve

### Traditional Single-stage Build Issues

Consider a typical Node.js application Dockerfile:

```dockerfile
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["npm", "start"]
```

**Problems with this approach:**
- Final image includes build tools (npm, build dependencies)
- Larger image size (often 500MB+ for Node.js apps)
- Security risk from unnecessary tools in production
- Slower deployments due to image size

### Multi-stage Build Solution

The same application with multi-stage builds:

```dockerfile
# Build stage
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --production
COPY --from=builder /app/dist ./dist
EXPOSE 3000
CMD ["npm", "start"]
```

**Benefits:**
- Final image excludes build tools and dev dependencies
- Smaller image size (often 50-70% reduction)
- More secure production environment
- Faster deployments and scaling

## Understanding Multi-stage Syntax

### Basic Structure

```dockerfile
# First stage
FROM base-image AS stage-name
# Build commands here

# Second stage  
FROM runtime-image
COPY --from=stage-name /source/path /destination/path
# Runtime commands here
```

### Key Components

**Stage Naming:** `AS stage-name`
- Names allow referencing stages in later `COPY --from` commands
- Stages can be referenced by name or index (0, 1, 2...)

**Cross-stage Copying:** `COPY --from=stage-name`
- Copies files from previous stages
- Only brings necessary artifacts to final stage

## Practical Examples

### Node.js Application

```dockerfile
# Stage 1: Build
FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:18-alpine
RUN addgroup -g 1001 -S nodejs && adduser -S nextjs -u 1001
WORKDIR /app
COPY --from=build --chown=nextjs:nodejs /app/dist ./dist
COPY --from=build --chown=nextjs:nodejs /app/node_modules ./node_modules
COPY --from=build --chown=nextjs:nodejs /app/package.json ./package.json

USER nextjs
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

**Size comparison:**
- Single-stage: ~850MB
- Multi-stage: ~200MB

### Go Application

```dockerfile
# Build stage
FROM golang:1.21 AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

# Production stage
FROM alpine:3.18
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/main .
CMD ["./main"]
```

**Size comparison:**
- Single-stage with Go tools: ~900MB
- Multi-stage with Alpine: ~15MB

### Python Application

```dockerfile
# Build stage
FROM python:3.11 AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user -r requirements.txt

# Production stage
FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
CMD ["python", "app.py"]
```

### Java Spring Boot Application

```dockerfile
# Build stage
FROM maven:3.9-openjdk-17 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

# Production stage
FROM openjdk:17-jre-slim
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

## Advanced Multi-stage Patterns

### Build Artifact Caching

```dockerfile
# Dependencies stage (cached separately)
FROM node:18 AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Build stage
FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine AS production
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY --from=build /app/dist ./dist
COPY package*.json ./
CMD ["npm", "start"]
```

### Development vs Production Stages

```dockerfile
FROM node:18 AS base
WORKDIR /app
COPY package*.json ./

# Development stage
FROM base AS development
RUN npm install
COPY . .
CMD ["npm", "run", "dev"]

# Build stage
FROM base AS build
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine AS production
WORKDIR /app
RUN npm ci --only=production && npm cache clean --force
COPY --from=build /app/dist ./dist
CMD ["npm", "start"]
```

**Usage:**
```bash
# Build for development
docker build --target development -t myapp:dev .

# Build for production
docker build --target production -t myapp:prod .
```

### Multi-platform Builds

```dockerfile
FROM --platform=$BUILDPLATFORM golang:1.21 AS builder
ARG TARGETPLATFORM
ARG BUILDPLATFORM
ARG TARGETOS
ARG TARGETARCH

WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 GOOS=$TARGETOS GOARCH=$TARGETARCH go build -o app

FROM alpine:3.18
COPY --from=builder /app/app /usr/local/bin/app
CMD ["app"]
```

## Optimization Techniques

### Layer Caching Strategy

```dockerfile
# Optimize for layer caching
FROM node:18 AS build

# Copy dependency files first (changes less frequently)
WORKDIR /app
COPY package*.json ./
RUN npm ci

# Copy source code last (changes more frequently)
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=build /app/dist ./dist
# ... rest of production stage
```

### Using .dockerignore

Create `.dockerignore` to exclude unnecessary files from build context:

```
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.nyc_output
coverage
.nyc_output
Dockerfile
.dockerignore
```

### Minimizing Final Image

```dockerfile
# Use minimal base images for production
FROM node:18 AS build
# ... build steps

# Use distroless or alpine for minimal attack surface
FROM gcr.io/distroless/nodejs18
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
CMD ["dist/server.js"]
```

## Security Benefits

### Reduced Attack Surface

**Single-stage image includes:**
- Build tools (compilers, package managers)
- Development dependencies
- Source code
- Build cache and temporary files

**Multi-stage image includes only:**
- Runtime dependencies
- Compiled application
- Minimal base OS

### Non-root User Implementation

```dockerfile
FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine
RUN addgroup -g 1001 -S nodejs && adduser -S appuser -u 1001 -G nodejs
WORKDIR /app
COPY --from=build --chown=appuser:nodejs /app/dist ./dist
COPY --from=build --chown=appuser:nodejs /app/node_modules ./node_modules

USER appuser
CMD ["node", "dist/server.js"]
```

## Build Performance Optimization

### Parallel Builds

```dockerfile
FROM alpine AS base

# These stages can build in parallel
FROM base AS build-frontend
COPY frontend/ /src
RUN cd /src && npm run build

FROM base AS build-backend  
COPY backend/ /src
RUN cd /src && go build -o app

# Combine results
FROM alpine
COPY --from=build-frontend /src/dist /var/www
COPY --from=build-backend /src/app /usr/local/bin/
```

### BuildKit Features

Enable BuildKit for advanced features:

```bash
export DOCKER_BUILDKIT=1
docker build .
```

Or use buildx:

```bash
docker buildx build --platform linux/amd64,linux/arm64 -t myapp .
```

## Common Patterns and Use Cases

### Static Site Generation

```dockerfile
# Build stage
FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage with nginx
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Database Migration

```dockerfile
# Build application
FROM golang:1.21 AS app-build
WORKDIR /app
COPY . .
RUN go build -o app

# Build migration tool
FROM migrate/migrate AS migrate

# Final image
FROM alpine:3.18
RUN apk add --no-cache ca-certificates
COPY --from=app-build /app/app /usr/local/bin/
COPY --from=migrate /usr/local/bin/migrate /usr/local/bin/
COPY migrations/ /migrations/
```

## Best Practices

### Stage Organization

```dockerfile
# 1. Base stage with common dependencies
FROM ubuntu:22.04 AS base
RUN apt-get update && apt-get install -y \
    curl \
    ca-certificates

# 2. Development dependencies
FROM base AS dev-deps
RUN apt-get install -y \
    build-essential \
    git

# 3. Build stage
FROM dev-deps AS build
WORKDIR /app
COPY . .
RUN make build

# 4. Production stage
FROM base
COPY --from=build /app/bin/app /usr/local/bin/
CMD ["app"]
```

### Naming Conventions

Use descriptive stage names:
- `dependencies` - Installing dependencies
- `build` - Compilation/build steps  
- `test` - Running tests
- `production` - Final runtime stage

### Resource Management

```dockerfile
FROM golang:1.21 AS build
# Limit build resources if needed
RUN go build -ldflags="-s -w" -o app

FROM scratch
# Minimal final image
COPY --from=build /app/app /app
ENTRYPOINT ["/app"]
```

## Troubleshooting Multi-stage Builds

### Common Issues

**Stage not found:**
```bash
Error: failed to solve: stage name not found: builder
```
Solution: Check stage naming with `AS` keyword

**File not found in COPY --from:**
```bash
Error: failed to compute cache key: file not found
```  
Solution: Verify file paths exist in source stage

**Permission issues:**
```bash
Error: permission denied
```
Solution: Use `--chown` in COPY commands or RUN chmod

### Debugging Techniques

**Build specific stage:**
```bash
docker build --target build -t debug-build .
docker run -it debug-build /bin/bash
```

**Inspect intermediate stages:**
```bash
docker build --target build .
docker images  # Find intermediate image
docker run -it <image-id> /bin/bash
```

## Measuring Success

### Image Size Comparison

```bash
# Check image sizes
docker images | grep myapp

# Compare layer details  
docker history myapp:single-stage
docker history myapp:multi-stage
```

### Build Time Analysis

```bash
# Time the build process
time docker build -t myapp .

# Use BuildKit for detailed timing
DOCKER_BUILDKIT=1 docker build --progress=plain .
```

### Security Scanning

```bash
# Scan for vulnerabilities
docker scout cves myapp:single-stage
docker scout cves myapp:multi-stage
```

Multi-stage builds represent a fundamental shift toward production-ready containerization. They reduce image size, improve security, and create cleaner separation between build and runtime environments. Master this technique to create professional-grade Docker images that deploy faster and run more securely.