# Docker Compose

Docker Compose lets you define and run multi-container applications using a YAML file. Instead of running multiple `docker run` commands, describe your entire application stack in one file.

## Why Use Docker Compose?

**Without Compose - Multiple commands:**
```bash
docker run -d --name db postgres
docker run -d --name web --link db -p 8000:8000 myapp
```

**With Compose - One command:**
```yaml
services:
  web:
    image: myapp
    ports:
      - "8000:8000"
  db:
    image: postgres
```

Then run: `docker-compose up`

## Your First Compose File

Create `docker-compose.yml`:
```yaml
version: '3.8'

services:
  web:
    image: nginx
    ports:
      - "8080:80"
  
  db:
    image: postgres:13
    environment:
      POSTGRES_PASSWORD: mypassword
```

Start everything:
```bash
docker-compose up -d
```

The `-d` flag runs containers in the background (detached mode).

## Essential Commands

### Starting and Stopping

```bash
# Start all services
docker-compose up

# Start in background 
docker-compose up -d

# Stop all services
docker-compose down
```

**What happens:**
- `up` creates and starts containers
- `down` stops and removes containers
- Networks and volumes remain unless you use `down -v`

### Viewing Information

```bash
# List running services
docker-compose ps

# View logs from all services
docker-compose logs

# Follow logs for specific service
docker-compose logs -f web
```

The `ps` command shows container status, ports, and names.

### Working with Services

```bash
# Restart a specific service
docker-compose restart web

# Execute command in running service
docker-compose exec web bash

# Scale a service (run multiple copies)
docker-compose up --scale web=3
```

`exec` lets you run commands inside a running container, just like `docker exec`.

## Service Configuration Explained

### Basic Service Setup
```yaml
services:
  myapp:
    image: python:3.9          # Base image to use
    ports:
      - "5000:5000"            # host_port:container_port
    environment:
      - DEBUG=true             # Set environment variables
    volumes:
      - .:/app                 # Mount current directory to /app
    working_dir: /app          # Set working directory inside container
    command: python app.py     # Override default command
```

### Building from Source
```yaml
services:
  web:
    build: .                   # Build from Dockerfile in current directory
    ports:
      - "3000:3000"
```

When you use `build`, Compose builds the image first, then creates the container.

### Service Dependencies
```yaml
services:
  web:
    image: nginx
    depends_on:                # Wait for these services before starting
      - api
      - db
  
  api:
    image: myapi
    depends_on:
      - db
  
  db:
    image: postgres
```

`depends_on` controls startup order but doesn't wait for services to be "ready".

## Practical Example: Web App with Database

```yaml
version: '3.8'

services:
  web:
    image: nginx
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
    depends_on:
      - db

  db:
    image: postgres:13
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: secret
    volumes:
      - db_data:/var/lib/postgresql/data

volumes:
  db_data:                     # Named volume for persistent data
```

**To use this:**
1. Create `html/index.html` with your website content
2. Run `docker-compose up -d`
3. Visit `http://localhost:8080`

The database data persists in the `db_data` volume even if containers are removed.

## Environment Variables

### Method 1: In Compose File
```yaml
services:
  app:
    image: myapp
    environment:
      - API_KEY=secret123
      - DEBUG=true
```

### Method 2: From .env File
Create `.env`:
```
API_KEY=secret123
DEBUG=true
```

Use in compose:
```yaml
services:
  app:
    image: myapp
    env_file:
      - .env
```

## Networks in Compose

By default, all services can reach each other by service name:

```yaml
services:
  web:
    image: nginx
    # Can connect to database using hostname "db"
  
  db:
    image: postgres
```

Inside the web container, you can connect to `db:5432` instead of an IP address.

## Development vs Production

### Override Files
Create `docker-compose.override.yml`:
```yaml
services:
  app:
    volumes:
      - .:/app               # Mount source code for development
    environment:
      - NODE_ENV=development
```

- `docker-compose up` automatically uses override file
- `docker-compose -f docker-compose.yml up` ignores override

## Common Issues and Solutions

### Port Already in Use
```bash
# Error: port is already allocated
# Solution: Change the host port
ports:
  - "8081:80"  # Instead of "8080:80"
```

### Service Can't Start
```bash
# Check what's wrong
docker-compose logs service_name

# Check if config is valid
docker-compose config
```

### Can't Connect Between Services
```bash
# Test connectivity
docker-compose exec web ping db

# Make sure both services are running
docker-compose ps
```

## Quick Command Reference

```bash
# Basic operations
docker-compose up -d          # Start in background
docker-compose down           # Stop and remove
docker-compose restart web    # Restart one service
docker-compose logs -f web    # Follow logs

# Development helpers  
docker-compose exec web bash  # Get shell in container
docker-compose up --build     # Rebuild images first
docker-compose pull           # Update all images

# Scaling and management
docker-compose up --scale web=3    # Run 3 web containers
docker-compose ps                  # List services
docker-compose top                 # Show running processes
```

## Key Takeaways

- Compose files define your entire application stack
- Services communicate using service names as hostnames
- Use volumes for data that needs to persist
- `depends_on` controls startup order
- Override files are perfect for development configurations

**Next:** [Volumes & Storage →](volumes.md) - Learn to manage persistent data and file sharing.