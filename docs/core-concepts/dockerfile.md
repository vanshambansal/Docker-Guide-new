# Dockerfile

A Dockerfile is like a recipe that tells Docker exactly how to build a custom image. Instead of manually setting up an environment, you write step-by-step instructions that Docker can follow automatically.

## What You'll Learn

By the end of this page, you'll:

- Write your first Dockerfile from scratch
- Understand essential Dockerfile instructions
- Build custom images from your Dockerfiles
- Follow best practices for efficient images
- Debug common Dockerfile issues
- Create multi-stage builds for production

!!! tip "Time to Complete"
    **90-120 minutes** including building practice images

## Understanding Dockerfiles

Think of a Dockerfile as:

- **Cooking recipe** - Step-by-step instructions to create something
- **Installation script** - Automated way to set up software
- **Blueprint** - Plan for building exactly what you need

### Dockerfile vs Manual Setup

| Manual Setup | Dockerfile |
|--------------|------------|
| Run commands one by one | Write all commands in file |
| Hard to remember steps | Document everything |
| Error-prone repetition | Automated and consistent |
| Hard to share setup | Share one simple file |

## Your First Dockerfile

Let's create a simple web server:

Create a new directory:
```bash
mkdir my-docker-app
cd my-docker-app
```

Create a simple HTML file:
```bash
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>My Docker App</title>
</head>
<body>
    <h1>Hello from Docker!</h1>
    <p>This page is served by nginx running in a Docker container.</p>
</body>
</html>
EOF
```

Create your first Dockerfile:
```bash
cat > Dockerfile << 'EOF'
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
EOF
```

Build your image:
```bash
docker build -t my-web-app .
```

Run your container:
```bash
docker run -d -p 8080:80 my-web-app
```

Test it: Open `http://localhost:8080` - you'll see your custom page!

## Essential Dockerfile Instructions

### FROM - Choose Base Image

Every Dockerfile starts with `FROM`:

```dockerfile
# Use official Python image
FROM python:3.9-slim

# Use specific version for consistency
FROM ubuntu:22.04

# Use minimal Alpine Linux
FROM alpine:3.18
```

### COPY and ADD - Add Files

Copy files from your computer into the image:

```dockerfile
# Copy single file
COPY app.py /app/app.py

# Copy entire directory
COPY . /app/

# Copy with different name
COPY config.txt /app/settings.conf

# ADD can also download files (use COPY when possible)
ADD https://example.com/file.tar.gz /tmp/
```

### RUN - Execute Commands

Install software and run commands:

```dockerfile
# Install packages (Ubuntu/Debian)
RUN apt-get update && apt-get install -y \
    curl \
    git \
    && rm -rf /var/lib/apt/lists/*

# Install packages (Alpine)
RUN apk add --no-cache curl git

# Install Python packages
RUN pip install flask requests

# Multiple commands in one RUN (better for image size)
RUN apt-get update && \
    apt-get install -y curl && \
    curl -O https://example.com/file.tar.gz && \
    tar -xzf file.tar.gz && \
    rm file.tar.gz
```

### WORKDIR - Set Working Directory

Set where commands run:

```dockerfile
# Set working directory
WORKDIR /app

# Now all commands run in /app
COPY . .
RUN pip install -r requirements.txt
```

### EXPOSE - Document Ports

Tell Docker which ports your app uses:

```dockerfile
# Web application
EXPOSE 80
EXPOSE 443

# Database
EXPOSE 5432

# Multiple ports
EXPOSE 8000 8001 8002
```

### ENV - Set Environment Variables

```dockerfile
# Set single variable
ENV API_URL=https://api.example.com

# Set multiple variables
ENV DEBUG=true \
    LOG_LEVEL=info \
    PORT=8000
```

### CMD - Default Command

What runs when container starts:

```dockerfile
# Run Python app
CMD ["python", "app.py"]

# Run web server
CMD ["nginx", "-g", "daemon off;"]

# Shell form (less preferred)
CMD python app.py
```

### ENTRYPOINT - Fixed Command

Like CMD but can't be overridden:

```dockerfile
# Always run this script
ENTRYPOINT ["./startup.sh"]

# Combine with CMD for default arguments
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "8000"]
```

## Practical Examples

### Example 1: Python Web App

Create project structure:
```bash
mkdir python-app
cd python-app
```

Create a simple Flask app:
```bash
cat > app.py << 'EOF'
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return '<h1>Hello from Python + Docker!</h1>'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
EOF
```

Create requirements file:
```bash
cat > requirements.txt << 'EOF'
flask==2.3.2
EOF
```

Create Dockerfile:
```bash
cat > Dockerfile << 'EOF'
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
EOF
```

Build and run:
```bash
docker build -t python-web .
docker run -d -p 5000:5000 python-web
```

### Example 2: Node.js Application

```dockerfile
FROM node:18-alpine

WORKDIR /app

# Copy package files first (better caching)
COPY package*.json ./
RUN npm install

# Copy application code
COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

### Example 3: Multi-Stage Build

For smaller production images:

```dockerfile
# Stage 1: Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Production stage
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

## Building Images

### Basic Build

```bash
# Build with tag
docker build -t my-app .

# Build with version tag
docker build -t my-app:v1.0 .

# Build from specific directory
docker build -t my-app ./my-project/
```

### Build Options

```bash
# Build without cache (clean build)
docker build --no-cache -t my-app .

# Build with build arguments
docker build --build-arg API_URL=https://api.com -t my-app .

# Build specific target in multi-stage
docker build --target builder -t my-app-dev .
```

### Using .dockerignore

Create `.dockerignore` to exclude files:

```bash
cat > .dockerignore << 'EOF'
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.DS_Store
EOF
```

## Best Practices

### Layer Optimization

```dockerfile
# Bad: Creates many layers
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y git
RUN apt-get clean

# Good: Single layer
RUN apt-get update && \
    apt-get install -y curl git && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### Caching Strategy

```dockerfile
# Bad: Changes to app code invalidate pip install
COPY . /app
RUN pip install -r requirements.txt

# Good: Install dependencies first
COPY requirements.txt /app/
RUN pip install -r requirements.txt
COPY . /app
```

### Security

```dockerfile
# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001

# Switch to non-root user
USER nextjs

# Use specific versions
FROM node:18.17.0-alpine
```

### Size Optimization

```dockerfile
# Use Alpine images
FROM python:3.9-alpine

# Remove package caches
RUN apk add --no-cache curl && \
    curl -O https://example.com/file && \
    rm -rf /var/cache/apk/*

# Multi-stage builds
FROM node:18 AS builder
# ... build steps ...

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
```

## Troubleshooting

### Build Fails

Check each step:
```bash
# Build with verbose output
docker build --progress=plain -t my-app .

# Run specific commands interactively
docker run -it python:3.9-slim bash
```

### Image Too Large

```bash
# Check image size
docker images

# Analyze layers
docker history my-app

# Use dive tool for detailed analysis
docker run --rm -it \
  -v /var/run/docker.sock:/var/run/docker.sock \
  wagoodman/dive:latest my-app
```

### Container Exits Immediately

```bash
# Check what command is running
docker inspect my-app --format='{{.Config.Cmd}}'

# Run with different command to debug
docker run -it my-app bash

# Check logs
docker logs container_name
```

## Advanced Concepts

### Build Arguments

```dockerfile
ARG API_URL=https://default-api.com
ARG VERSION=latest

RUN echo "Building version $VERSION"
ENV API_ENDPOINT=$API_URL
```

Build with arguments:
```bash
docker build --build-arg API_URL=https://prod-api.com --build-arg VERSION=1.0 -t my-app .
```

### Health Checks

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost/ || exit 1
```

### Multiple Architectures

```dockerfile
FROM --platform=$BUILDPLATFORM node:18-alpine
ARG TARGETPLATFORM
ARG BUILDPLATFORM

RUN echo "Building on $BUILDPLATFORM for $TARGETPLATFORM"
```

## Hands-On Exercise

Create a complete web application:

1. **Create project directory:**
```bash
mkdir full-stack-app
cd full-stack-app
```

2. **Create a simple API:**
```bash
cat > app.py << 'EOF'
from flask import Flask, jsonify
import os

app = Flask(__name__)

@app.route('/')
def home():
    return jsonify({
        "message": "Hello from Docker!",
        "version": os.getenv('APP_VERSION', '1.0'),
        "environment": os.getenv('NODE_ENV', 'development')
    })

@app.route('/health')
def health():
    return jsonify({"status": "healthy"})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
EOF
```

3. **Create requirements:**
```bash
echo "flask==2.3.2" > requirements.txt
```

4. **Create Dockerfile:**
```bash
cat > Dockerfile << 'EOF'
FROM python:3.9-slim

# Set build argument
ARG APP_VERSION=1.0
ENV APP_VERSION=$APP_VERSION

# Create non-root user
RUN useradd --create-home --shell /bin/bash app

# Set working directory
WORKDIR /home/app

# Install dependencies as root
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy app and change ownership
COPY app.py .
RUN chown -R app:app /home/app

# Switch to non-root user
USER app

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:5000/health || exit 1

# Expose port
EXPOSE 5000

# Run application
CMD ["python", "app.py"]
EOF
```

5. **Build and test:**
```bash
# Build with version
docker build --build-arg APP_VERSION=2.0 -t my-api:2.0 .

# Run container
docker run -d -p 5000:5000 --name my-api-container my-api:2.0

# Test the API
curl http://localhost:5000
curl http://localhost:5000/health

# Check health status
docker ps

# Cleanup
docker stop my-api-container
docker rm my-api-container
```

## What's Next?

Excellent! You now know how to create custom Docker images using Dockerfiles.

Next up: **[Docker Compose →](../working-with-docker/compose.md)** - Learn to manage multiple containers together with Docker Compose.

## Key Takeaways

- Dockerfiles are recipes for building custom images
- Start with appropriate base images (Alpine for size, Ubuntu for compatibility)
- Optimize layers by combining RUN commands and copying dependencies first  
- Use multi-stage builds for smaller production images
- Always create non-root users for security
- Include health checks for production applications
- Use .dockerignore to exclude unnecessary files