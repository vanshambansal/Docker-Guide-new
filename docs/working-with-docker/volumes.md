# Volumes & Storage

Containers are designed to be temporary - when you remove a container, all its data disappears. Volumes solve this by providing persistent storage that survives container restarts, updates, and deletions.

## Why You Need Volumes

**Without volumes:**
```bash
# Create database container
docker run --name mydb postgres

# Add some data to database
# ... work with database ...

# Remove container
docker rm mydb

# Data is GONE forever! 😱
```

**With volumes:**
```bash
# Create database with persistent storage
docker run --name mydb -v mydata:/var/lib/postgresql/data postgres

# Remove container
docker rm mydb

# Data is safe in the volume! ✅
# Start new container with same data
docker run --name newdb -v mydata:/var/lib/postgresql/data postgres
```

## Types of Mounts

Docker provides three ways to mount data:

| Type | Description | Use Case |
|------|-------------|----------|
| **Named Volumes** | Managed by Docker | Database data, app files |
| **Bind Mounts** | Direct host path | Development, configuration |
| **tmpfs** | Memory-only | Temporary files, secrets |

## Named Volumes

Docker manages these volumes completely. Best for production data.

### Creating and Using Named Volumes

```bash
# Create a named volume
docker volume create mydata
```

The `create` command makes a new volume that Docker will manage and store safely.

```bash
# Use the volume in a container
docker run -d --name db -v mydata:/var/lib/postgresql/data postgres
```

**Command breakdown:**
- `-v mydata:/var/lib/postgresql/data` mounts volume `mydata` to `/var/lib/postgresql/data` inside container
- Format is always `volume_name:container_path`

### Volume Management Commands

```bash
# List all volumes
docker volume ls
```

Shows all volumes on your system with their names and drivers.

```bash
# Inspect a volume (see details)
docker volume inspect mydata
```

This shows where Docker stores the volume on your computer and other metadata.

```bash
# Remove unused volumes
docker volume prune
```

Removes volumes that aren't currently attached to any containers. **Warning:** This deletes data permanently!

```bash
# Remove specific volume
docker volume rm mydata
```

Only works if no containers are using the volume.

## Bind Mounts

Mount a specific folder from your computer directly into the container.

### Development Example

```bash
# Mount current directory to /app in container
docker run -it -v $(pwd):/app node:18 bash
```

**Command explanation:**
- `$(pwd)` gets your current directory path
- `$(pwd):/app` mounts your current folder to `/app` inside container
- Changes in either location are reflected immediately in both

### Practical Development Setup

```bash
# Create project directory
mkdir my-node-app
cd my-node-app

# Create a simple app
echo 'console.log("Hello from Node!");' > app.js

# Run with bind mount for live editing
docker run -it -v $(pwd):/app -w /app node:18 bash
```

**What `-w /app` does:**
- Sets `/app` as the working directory inside container
- Like running `cd /app` automatically when container starts

Inside the container:
```bash
# Your files are available
ls -la
# app.js

# Run your app
node app.js
# Hello from Node!

# Install packages (saves to your computer too)
npm init -y
npm install express
```

All changes save to your computer and persist after container stops.

## Practical Examples

### Example 1: Persistent Database

```bash
# Create named volume for database
docker volume create postgres_data

# Run PostgreSQL with persistent storage
docker run -d \
  --name mydb \
  -e POSTGRES_PASSWORD=secret123 \
  -e POSTGRES_DB=myapp \
  -v postgres_data:/var/lib/postgresql/data \
  postgres:13
```

**Command breakdown:**
- `docker run -d` runs container in background (detached)
- `--name mydb` gives container a memorable name
- `-e` sets environment variables (password and database name)
- `-v postgres_data:/var/lib/postgresql/data` mounts volume to database storage location

**Test persistence:**
```bash
# Connect and create data
docker exec -it mydb psql -U postgres -d myapp
# CREATE TABLE users (id INT, name TEXT);
# INSERT INTO users VALUES (1, 'Alice');
# \q

# Stop and remove container
docker stop mydb
docker rm mydb

# Start new container with same volume
docker run -d --name mydb2 -e POSTGRES_PASSWORD=secret123 -v postgres_data:/var/lib/postgresql/data postgres:13

# Data is still there!
docker exec -it mydb2 psql -U postgres -d myapp
# SELECT * FROM users;
# (shows Alice!)
```

### Example 2: Development Environment

```bash
# Create React app directory
mkdir my-react-app
cd my-react-app

# Create package.json
cat > package.json << 'EOF'
{
  "name": "my-react-app",
  "version": "1.0.0",
  "scripts": {
    "start": "react-scripts start"
  },
  "dependencies": {
    "react": "^18.0.0",
    "react-dom": "^18.0.0",
    "react-scripts": "5.0.1"
  }
}
EOF

# Start development container with bind mount
docker run -it \
  -v $(pwd):/app \
  -v /app/node_modules \
  -w /app \
  -p 3000:3000 \
  node:18 bash
```

**Special volume trick explained:**
- `-v $(pwd):/app` mounts your code
- `-v /app/node_modules` creates anonymous volume for node_modules
- This prevents your local node_modules (if any) from overriding container's installed packages

## Volumes in Docker Compose

Much easier to manage volumes with Compose:

```yaml
version: '3.8'

services:
  db:
    image: postgres:13
    environment:
      POSTGRES_PASSWORD: secret123
      POSTGRES_DB: myapp
    volumes:
      - db_data:/var/lib/postgresql/data    # Named volume
    
  web:
    image: nginx
    volumes:
      - ./html:/usr/share/nginx/html        # Bind mount
    ports:
      - "8080:80"

volumes:
  db_data:    # Declare named volume
```

**Compose volume syntax:**
- `db_data:/var/lib/postgresql/data` uses named volume
- `./html:/usr/share/nginx/html` uses bind mount (relative path)
- Must declare named volumes in `volumes:` section

### Development Compose Example

```yaml
version: '3.8'

services:
  app:
    image: node:18
    working_dir: /app
    command: npm start
    volumes:
      - .:/app                    # Bind mount source code
      - /app/node_modules         # Anonymous volume for dependencies
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
```

**How this helps development:**
- Edit files on your computer, see changes instantly in container
- `node_modules` stays in container (faster, cleaner)
- Container restart preserves installed packages

## Volume Backup and Restore

### Backup a Volume

```bash
# Create backup of named volume
docker run --rm \
  -v mydata:/data \
  -v $(pwd):/backup \
  alpine \
  tar czf /backup/mydata-backup.tar.gz -C /data .
```

**Command breakdown:**
- `--rm` removes container after it finishes
- `-v mydata:/data` mounts the volume you want to backup
- `-v $(pwd):/backup` mounts current directory to save backup file
- `alpine` is small Linux image perfect for utilities
- `tar czf` creates compressed archive of all files in `/data`

### Restore from Backup

```bash
# Create new volume and restore data
docker volume create mydata_restored

docker run --rm \
  -v mydata_restored:/data \
  -v $(pwd):/backup \
  alpine \
  tar xzf /backup/mydata-backup.tar.gz -C /data
```

**What `tar xzf` does:**
- `x` extracts files
- `z` handles gzip compression
- `f` specifies file to extract
- `-C /data` extracts to `/data` directory

## Troubleshooting Common Issues

### Permission Problems

```bash
# Problem: Can't write to bind mounted directory
docker run -v $(pwd):/app ubuntu touch /app/test.txt
# touch: cannot touch '/app/test.txt': Permission denied
```

**Solution - Match user IDs:**
```bash
# Check your user ID
id -u
# 1000

# Run container with your user ID
docker run -u 1000:1000 -v $(pwd):/app ubuntu touch /app/test.txt
```

The `-u 1000:1000` makes the container process run as user 1000 (your user) instead of root.

### Volume Not Mounting

```bash
# Check if volume exists
docker volume ls | grep mydata

# If not found, create it
docker volume create mydata

# Check volume details
docker volume inspect mydata
```

### Data Not Persisting

Common mistake - using container paths instead of volumes:

```bash
# Wrong: Data goes away with container
docker run postgres

# Right: Data persists in volume
docker run -v postgres_data:/var/lib/postgresql/data postgres
```

Always mount volumes to the application's data directory (varies by application).

## File System Types

### tmpfs Mounts (Memory Storage)

For temporary files that shouldn't be written to disk:

```bash
# Mount memory-based filesystem
docker run -it --tmpfs /tmp ubuntu bash
```

**When to use tmpfs:**
- Sensitive temporary files
- High-performance temporary storage
- Files that should never persist

**Inside container:**
```bash
# Files written to /tmp stay in memory only
echo "secret data" > /tmp/secret.txt
# This never touches your hard drive
```

## Best Practices

### Choose the Right Mount Type

```bash
# Named volumes - for application data
docker run -v db_data:/var/lib/mysql mysql

# Bind mounts - for development and config
docker run -v $(pwd):/app -v ./config:/etc/myapp myapp

# tmpfs - for sensitive temporary data
docker run --tmpfs /tmp myapp
```

### Development vs Production

**Development - Use bind mounts:**
```yaml
services:
  app:
    volumes:
      - .:/app                    # Live code editing
      - ./config:/app/config      # Local config files
```

**Production - Use named volumes:**
```yaml
services:
  app:
    volumes:
      - app_data:/app/data        # Managed by Docker
      - config_data:/app/config   # Controlled storage
```

### Regular Backups

```bash
# Backup script for important volumes
#!/bin/bash
DATE=$(date +%Y%m%d)
docker run --rm -v important_data:/data -v $(pwd):/backup alpine \
  tar czf /backup/backup-$DATE.tar.gz -C /data .
```

## Volume Commands Reference

```bash
# Volume management
docker volume create myvolume          # Create named volume
docker volume ls                       # List all volumes
docker volume inspect myvolume         # Show volume details
docker volume rm myvolume             # Delete volume
docker volume prune                   # Remove unused volumes

# Using volumes in containers
docker run -v myvolume:/path image     # Mount named volume
docker run -v $(pwd):/path image      # Mount current directory
docker run --tmpfs /path image        # Mount memory filesystem

# Volume info in running containers
docker inspect container_name          # Shows all mounts
docker exec container df -h           # Check disk usage inside container
```

## What's Next?

Excellent! You now understand how to persist data and share files between containers and your host system.

**Next:** [Networks →](networks.md) - Learn to control how containers communicate with each other and the outside world.

## Key Takeaways

- **Named volumes** are best for persistent application data - Docker manages them safely
- **Bind mounts** are perfect for development - edit files on your computer, see changes in container instantly  
- **Always mount volumes to application data directories** - varies by application (PostgreSQL uses `/var/lib/postgresql/data`)
- **Use volume backups** for important data - simple tar commands can save your data
- **Match user permissions** with `-u` flag to avoid file permission issues
- **Anonymous volumes** (like `/app/node_modules`) prevent local files from overriding container files