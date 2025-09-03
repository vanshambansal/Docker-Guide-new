# Docker Basics

Welcome! Now that Docker is installed, let's understand what it actually is and how to use it. Think of this as your first day at Docker school! 🎓

## What is Docker? (In Simple Terms)

Imagine you're moving to a new apartment. Instead of packing everything separately and hoping it fits, you use **shipping containers** - standardized boxes that fit anywhere and contain everything you need.

Docker does the same thing for software:

- **Traditional way:** "It works on my machine!" 😅 (but breaks on yours)
- **Docker way:** "It works in this container!" ✅ (and runs anywhere)

!!! example "Real-world analogy"
    **Without Docker:** Like giving someone a recipe and hoping they have the right ingredients, kitchen tools, and oven temperature.
    
    **With Docker:** Like giving someone a food truck that already has everything prepared and ready to serve!

## How Docker Works (The Simple Picture)

```
You type commands → Docker Client → Docker Daemon → Magic happens! → Container runs
```

Let's break this down:

1. **You** type `docker run nginx` in your terminal
2. **Docker Client** receives your command
3. **Docker Daemon** (the engine) finds the nginx image and creates a container
4. **Container** starts running with nginx web server inside

!!! info "Think of it like this"
    - **Docker Client** = Your TV remote
    - **Docker Daemon** = Your TV
    - **Images** = Netflix shows available to watch
    - **Containers** = The show actually playing on your screen

## Key Concepts You Need to Know

### Images vs Containers

This is the **#1 thing** beginners get confused about:

| **Docker Image** | **Docker Container** |
|------------------|----------------------|
| 📄 Blueprint/Recipe | 🏠 Actual running thing |
| Like a photo | Like a printed copy |
| Stored on disk | Running in memory |
| `docker images` | `docker ps` |

**Example:**
- `nginx` image = The recipe for a web server
- `nginx` container = An actual web server running and serving web pages

### Volumes (Data Storage)

Containers are **temporary** - when they stop, everything inside disappears! 

**Volumes** are like external hard drives for containers:

- Container stops → Data in volume stays safe
- New container starts → Can access the same data

### Networks

Networks let containers talk to each other, like phones connected to the same WiFi network.

## Essential Commands (With Examples)

### Check What's Running

```bash
docker ps
```

**Output example:**
```
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
a1b2c3d4e5f6   nginx     "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes   80/tcp    wonderful_tesla
```

**What this tells you:**
- Container ID: `a1b2c3d4e5f6` (unique identifier)
- Image: `nginx` (what it's running)
- Status: `Up 2 minutes` (it's been running for 2 minutes)
- Name: `wonderful_tesla` (Docker gives random names)

### See All Containers (Including Stopped Ones)

```bash
docker ps -a
```

### Run a Container

```bash
docker run nginx
```

**What happens:**
1. Docker looks for `nginx` image locally
2. If not found, downloads it from Docker Hub
3. Creates and starts a container
4. Nginx web server starts running

### Run a Container in the Background

```bash
docker run -d nginx
```

The `-d` flag means "detached" (runs in background).

### Run a Container with a Custom Name

```bash
docker run -d --name my-web-server nginx
```

Now instead of a random name like "wonderful_tesla", it's called "my-web-server".

### Stop a Container

```bash
docker stop my-web-server
```

Or use the container ID:
```bash
docker stop a1b2c3d4e5f6
```

### Start a Stopped Container

```bash
docker start my-web-server
```

### Remove a Container

```bash
docker rm my-web-server
```

!!! warning "Container must be stopped first"
    You can't remove a running container. Stop it first with `docker stop`, then remove it.

### List All Images

```bash
docker images
```

**Output example:**
```
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
nginx        latest    a6bd71f48f68   2 weeks ago    187MB
hello-world  latest    d2c94e258dcb   10 months ago  13.3kB
```

### Remove an Image

```bash
docker rmi nginx
```

!!! warning "Remove containers first"
    You can't remove an image if containers are using it. Remove all containers using that image first.

## Your First Hands-On Practice

Let's do a complete example together! Follow these steps exactly:

### Step 1: Run a Web Server

```bash
docker run -d -p 8080:80 --name my-first-server nginx
```

**What each part means:**
- `-d` = Run in background
- `-p 8080:80` = Connect your computer's port 8080 to container's port 80
- `--name my-first-server` = Give it a friendly name
- `nginx` = The image to use

### Step 2: Check It's Running

```bash
docker ps
```

You should see your container listed!

### Step 3: Visit Your Web Server

Open your web browser and go to: `http://localhost:8080`

You should see the "Welcome to nginx!" page! 🎉

### Step 4: Check the Logs

```bash
docker logs my-first-server
```

You'll see nginx's access logs showing your visit!

### Step 5: Stop and Clean Up

```bash
# Stop the container
docker stop my-first-server

# Remove the container
docker rm my-first-server

# Check it's gone
docker ps -a
```

## Understanding the Docker Workflow

Here's the typical workflow you'll use:

1. **Find an image** (on Docker Hub or build your own)
2. **Run a container** from that image
3. **Interact with the container** (web browser, terminal, etc.)
4. **Stop the container** when done
5. **Remove the container** to clean up

## Common Beginner Questions

### "Why do containers stop when I close the terminal?"

Some containers (like `hello-world`) run a command and then exit naturally. Others (like `nginx`) keep running until you stop them.

### "Where do images come from?"

- **Docker Hub** - Like an app store for Docker images
- **You build them** - Create your own using Dockerfiles
- **Other registries** - Companies often have private image stores

### "What's the difference between `docker run` and `docker start`?"

- `docker run` = Create a NEW container from an image and start it
- `docker start` = Start an EXISTING container that was stopped

### "Why use `-d` flag?"

Without `-d`, the container runs in "foreground" mode - your terminal is stuck showing the container's output. With `-d`, it runs in the background and gives you your terminal back.

## Quick Reference Cheat Sheet

| **Task** | **Command** | **Example** |
|----------|-------------|-------------|
| Run new container | `docker run <image>` | `docker run nginx` |
| Run in background | `docker run -d <image>` | `docker run -d nginx` |
| List running containers | `docker ps` | - |
| List all containers | `docker ps -a` | - |
| Stop container | `docker stop <name/id>` | `docker stop my-server` |
| Start container | `docker start <name/id>` | `docker start my-server` |
| Remove container | `docker rm <name/id>` | `docker rm my-server` |
| List images | `docker images` | - |
| Remove image | `docker rmi <image>` | `docker rmi nginx` |
| View logs | `docker logs <name/id>` | `docker logs my-server` |

## What's Next?

Now that you understand the basics, let's dive deeper:

- **[Understanding Images](images.md)** - How to find, pull, and manage images
- **[Working with Containers](containers.md)** - Learn advanced container management
- **[Creating Dockerfiles](dockerfile.md)** - Build your own custom images

!!! tip "Practice Makes Perfect"
    The best way to learn Docker is by doing! Try running different images like `redis`, `postgres`, or `python` and see what happens. Don't worry about breaking anything - containers are isolated and safe to experiment with!