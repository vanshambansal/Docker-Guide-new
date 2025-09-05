<!-- # Docker Containers

If Docker images are like **recipe cards**, then containers are the **actual dishes you cook and eat**! 🍲

## What Is a Docker Container?

A Docker container is a **running instance** of an image.  
Think of it as **baking a pizza** from a frozen pizza image:

- The image is the frozen pizza (unchanging)
- The container is the hot, ready-to-eat pizza (running app)
- You can make many pizzas (containers) from one image!

!!! info "Key Point"
    **Containers are temporary and disposable.** You can start, stop, delete, and create new ones anytime!

## How Containers Work

```
Image (Recipe) → docker run → Container (Running App)
```

- **Image** = Instructions and ingredients
- **Container** = The actual running application

## Why Use Containers?

- **Isolation:** Each container runs separately (like apps on your phone)
- **Consistency:** Runs the same everywhere (your laptop, server, cloud)
- **Portability:** Move containers between computers easily

## Working with Containers

### See What Containers Are Running

```bash
docker ps
```

**Example output:**
```
CONTAINER ID   IMAGE     COMMAND                  STATUS         PORTS                  NAMES
e3f1c1a2b3c4   nginx     "/docker-entrypoint.…"   Up 2 minutes   0.0.0.0:8080->80/tcp   web
```

### See All Containers (Including Stopped)

```bash
docker ps -a
```

### Start a New Container

```bash
docker run -d --name web -p 8080:80 nginx
```

- `-d` = Run in background (detached)
- `--name web` = Name your container "web"
- `-p 8080:80` = Map port 8080 on your computer to port 80 in the container

### Stop a Running Container

```bash
docker stop web
```

### Start a Stopped Container

```bash
docker start web
```

### Remove a Container

```bash
docker rm web
```

!!! warning "Can't remove running containers"
    Stop the container before removing it!

### See Container Logs (Output)

```bash
docker logs web
```

### Run a Command Inside a Container

```bash
docker exec -it web bash
```

- `-it` = Interactive terminal
- `bash` = Start a shell inside the container

Type `exit` to leave the shell.

## Hands-On Practice Exercise

Let's try it out step by step!

### Step 1: Run a Web Server

```bash
docker run -d --name my-nginx -p 8080:80 nginx
```

Visit `http://localhost:8080` in your browser.

### Step 2: See Running Containers

```bash
docker ps
```

### Step 3: View Logs

```bash
docker logs my-nginx
```

### Step 4: Enter the Container

```bash
docker exec -it my-nginx bash
```

Look around, then type `exit` to leave.

### Step 5: Stop and Remove the Container

```bash
docker stop my-nginx
docker rm my-nginx
```

## Common Mistakes Beginners Make

### Mistake 1: Forgetting to Stop Before Removing

❌ `docker rm my-nginx` (fails if running)  
✅ `docker stop my-nginx` then `docker rm my-nginx`

### Mistake 2: Losing Data

Containers are **temporary**. If you delete a container, its data is gone unless you use **volumes** (covered later).

### Mistake 3: Confusing Images and Containers

- **Image** = Blueprint (unchanging)
- **Container** = Running app (can be started/stopped/deleted)

## Quick Reference

| **Task** | **Command** | **Example** |
|----------|-------------|-------------|
| List running containers | `docker ps` | - |
| List all containers | `docker ps -a` | - |
| Run new container | `docker run ...` | `docker run -d --name web nginx` |
| Stop container | `docker stop <name>` | `docker stop web` |
| Start container | `docker start <name>` | `docker start web` |
| Remove container | `docker rm <name>` | `docker rm web` |
| View logs | `docker logs <name>` | `docker logs web` |
| Enter container shell | `docker exec -it <name> bash` | `docker exec -it web bash` |

## Key Takeaways

🎯 **Containers are running apps** created from images  
🎯 **You can start, stop, and remove containers anytime**  
🎯 **Containers are isolated and portable**  
🎯 **Images are blueprints, containers are the real thing**

## Conclusion

Awesome! Now you know how to run, manage, and explore Docker containers.  
Containers are the heart of Docker—they let you run any app, anywhere, with just a few commands.

## What's Next?

Ready to build your own images?  
Next, you'll learn about **Dockerfiles**—the recipes for creating custom images!

- **[Creating Dockerfiles](dockerfile.md)** - Build your own images step by step

Let's keep going and become a Docker pro! 🚀 -->











# Docker Containers

Containers are where Docker images come to life! If images are blueprints, containers are the actual running applications built from those blueprints.

## What You'll Learn

By the end of this page, you'll:

- Create containers from images and manage their lifecycle
- Work with interactive and background containers
- Debug container issues using logs and inspection tools
- Understand container isolation and resource management
- Master essential container commands for daily use

!!! tip "Time to Complete"
    **60-90 minutes** including hands-on practice

## Understanding Containers

Think of containers like **lightweight virtual machines** that:

- **Share the host OS** but have isolated processes
- **Start instantly** compared to traditional VMs
- **Use minimal resources** while providing complete isolation
- **Are portable** - run identically anywhere Docker is installed

### Container vs Virtual Machine

| Container | Virtual Machine |
|-----------|----------------|
| Shares host OS kernel | Includes full guest OS |
| Starts in seconds | Takes minutes to boot |
| Minimal resource overhead | Heavy resource usage |
| Lightweight (MBs) | Heavy (GBs) |

## Your First Container

Let's create and run your first container:

```bash
docker run hello-world
```

What happens:
```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

Docker automatically:
1. Downloaded the `hello-world` image
2. Created a container from it
3. Ran the program inside the container
4. Showed you the output

## Interactive Containers

Get inside a container and explore:

```bash
docker run -it ubuntu:22.04 bash
```

Command breakdown:
- `-it` = Interactive mode with terminal
- `ubuntu:22.04` = The image to use
- `bash` = Command to run inside container

You'll see a prompt like:
```
root@a1b2c3d4e5f6:/#
```

Try these commands inside the container:

```bash
# Check the OS
cat /etc/os-release

# List files
ls -la

# Install something
apt update && apt install -y curl

# Test it works
curl --version

# Exit the container
exit
```

## Background Containers

Run services that keep running:

```bash
# Run nginx web server in background
docker run -d -p 8080:80 nginx
```

- `-d` = Detached mode (runs in background)
- `-p 8080:80` = Map port 8080 on your machine to port 80 in container

Test it: Open your browser to `http://localhost:8080`

## Essential Commands

### List Containers

See running containers:
```bash
docker ps
```

See all containers (including stopped):
```bash
docker ps -a
```

### Start/Stop Containers

```bash
# Stop a running container
docker stop container_name

# Start a stopped container  
docker start container_name

# Restart a container
docker restart container_name

# Force kill a container
docker kill container_name
```

### Get Container Information

```bash
# View container logs
docker logs container_name

# Follow logs in real-time
docker logs -f container_name

# See resource usage
docker stats container_name

# Get detailed container info
docker inspect container_name
```

### Execute Commands in Running Containers

```bash
# Get a shell inside running container
docker exec -it container_name bash

# Run a single command
docker exec container_name ls /app

# Run command as specific user
docker exec -u root container_name whoami
```

### Container Cleanup

```bash
# Remove a stopped container
docker rm container_name

# Force remove a running container
docker rm -f container_name

# Remove all stopped containers
docker container prune

# Remove everything unused
docker system prune
```

## Hands-On Practice

### Exercise 1: Web Server Setup

```bash
# Step 1: Run nginx with custom name
docker run -d -p 8080:80 --name my-web nginx

# Step 2: Check it's running
docker ps

# Step 3: View logs
docker logs my-web

# Step 4: Get inside the container
docker exec -it my-web bash

# Step 5: Explore web files
ls /usr/share/nginx/html/

# Step 6: Exit and cleanup
exit
docker stop my-web
docker rm my-web
```

### Exercise 2: Database Container

```bash
# Run MySQL database
docker run -d --name my-db \
  -e MYSQL_ROOT_PASSWORD=secret123 \
  -e MYSQL_DATABASE=testdb \
  mysql:8.0

# Wait for startup (watch logs)
docker logs -f my-db

# Connect to database
docker exec -it my-db mysql -u root -psecret123

# Try some SQL commands
SHOW DATABASES;
USE testdb;
CREATE TABLE users (id INT, name VARCHAR(50));
INSERT INTO users VALUES (1, 'Alice');
SELECT * FROM users;
exit

# Cleanup
docker stop my-db
docker rm my-db
```

## Container Options

### Port Mapping

```bash
# Map single port
docker run -d -p 8080:80 nginx

# Map multiple ports
docker run -d -p 8080:80 -p 8443:443 nginx

# Map to specific interface
docker run -d -p 127.0.0.1:8080:80 nginx
```

### Volume Mounting

```bash
# Mount current directory
docker run -it -v $(pwd):/app ubuntu bash

# Mount specific folder
docker run -it -v /home/user/data:/data ubuntu bash

# Named volume (persisted)
docker run -it -v mydata:/data ubuntu bash
```

### Environment Variables

```bash
# Set single variable
docker run -e API_KEY=secret123 myapp

# Set multiple variables
docker run -e DB_HOST=localhost -e DB_PORT=5432 myapp

# Load from file
docker run --env-file .env myapp
```

### Resource Limits

```bash
# Limit memory
docker run --memory=512m nginx

# Limit CPU
docker run --cpus="1.5" nginx

# Limit both
docker run --memory=1g --cpus="2.0" nginx
```

## Troubleshooting

### Container Won't Start

Check the logs first:
```bash
docker logs container_name
```

Common issues:
- Port already in use
- Missing environment variables
- Wrong command or entrypoint
- Insufficient permissions

### Can't Connect to Service

Debug checklist:
```bash
# Is container running?
docker ps

# Check port mapping
docker port container_name

# Test from inside container
docker exec container_name curl localhost:80

# Check if service is listening
docker exec container_name netstat -tlnp
```

### Container Exits Immediately

Run in interactive mode to see errors:
```bash
docker run -it problematic_image bash
```

Or check what command is being run:
```bash
docker inspect image_name --format='{{.Config.Cmd}}'
```

## Best Practices

### Naming and Organization

```bash
# Good: Use descriptive names
docker run -d --name web-server nginx
docker run -d --name user-db mysql

# Avoid: Random names
docker run -d nginx
```

### Resource Management

```bash
# Good: Set resource limits
docker run -d --memory=512m --cpus="1.0" myapp

# Monitor resource usage
docker stats
```

### Security

```bash
# Good: Run as non-root user
docker run -d -u nginx nginx

# Good: Use read-only filesystem when possible
docker run -d --read-only nginx

# Good: Drop unnecessary capabilities
docker run -d --cap-drop=ALL --cap-add=NET_BIND_SERVICE nginx
```

### Cleanup

```bash
# Regular maintenance
docker container prune
docker image prune
docker system prune
```

## Container Lifecycle

Understanding container states:

1. **Created** - Container exists but not running
2. **Running** - Container is actively executing
3. **Paused** - Container processes frozen (can resume)
4. **Stopped** - Container finished or was stopped
5. **Removed** - Container deleted completely

```bash
# See container state
docker ps -a

# Pause/unpause containers
docker pause container_name
docker unpause container_name
```

## What's Next?

Great! You now understand how to create, manage, and troubleshoot Docker containers. 

Next up: **[Dockerfile →](dockerfile.md)** - Learn to create your own custom images by writing Dockerfiles.

## Key Takeaways

- Containers are running instances of images
- Use `-it` for interactive containers, `-d` for background services
- Always name your containers for easier management
- Use `docker exec` to get inside running containers for debugging
- Clean up unused containers regularly to save space
- Set resource limits in production environments