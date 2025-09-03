# Docker Images

Think of Docker images like **recipe cards** 📋 - they contain all the instructions and ingredients needed to create something delicious (in this case, a running application)!

## What Exactly Is a Docker Image?

A Docker image is like a **frozen pizza** 🍕:

- It's **pre-made** with all ingredients included
- It's **packaged** and ready to use
- You can **store many** in your freezer
- When you want pizza, you **cook it** (run it as a container)
- The original frozen pizza stays unchanged - you can make many pizzas from it!

!!! info "Key Point"
    **Images are templates** - they don't do anything by themselves. You need to **run** them to create containers that actually work!

## How Images Work

```
Image (Recipe) → docker run → Container (Actual Running App)
```

**Example:**
- `nginx` image = Instructions for building a web server
- `nginx` container = An actual web server running and serving websites

## Where Do Images Come From?

### 1. Docker Hub (The App Store for Docker)

**Docker Hub** is like Google Play Store, but for Docker images. It has thousands of ready-made images:

- **nginx** - Web server
- **mysql** - Database
- **python** - Python programming environment
- **wordpress** - Website builder
- **redis** - Fast database for caching

Visit [hub.docker.com](https://hub.docker.com) to browse!

### 2. You Build Them Yourself

Later, you'll learn to create your own images using **Dockerfiles** (like writing your own recipe).

### 3. Private Registries

Companies often store their private images in secure registries.

## Working with Images

### See What Images You Have

```bash
docker images
```

**Example output:**
```
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
nginx        latest    a6bd71f48f68   2 weeks ago    187MB
hello-world  latest    d2c94e258dcb   10 months ago  13.3kB
python       3.9       1b33c72fae8d   1 month ago    915MB
```

**What each column means:**
- **REPOSITORY** - The image name (like `nginx`)
- **TAG** - The version (like `latest`, `3.9`, `v1.2`)
- **IMAGE ID** - Unique identifier for this image
- **CREATED** - When this image was built
- **SIZE** - How much space it takes

### Download an Image (Without Running It)

```bash
docker pull nginx
```

This downloads the nginx image but doesn't run it yet. It's like buying a frozen pizza but not cooking it.

### Download a Specific Version

```bash
docker pull python:3.9
```

The `:3.9` part is called a **tag** - it specifies which version you want.

!!! tip "Image Tags Explained"
    - `nginx:latest` = The newest version (default)
    - `python:3.9` = Python version 3.9 specifically
    - `mysql:8.0` = MySQL version 8.0
    - If you don't specify a tag, Docker assumes `:latest`

### Remove an Image

```bash
docker rmi nginx
```

Or using the image ID:
```bash
docker rmi a6bd71f48f68
```

!!! warning "Can't remove images being used"
    If any containers are using an image, you can't remove it. Stop and remove those containers first!

## Let's Practice with Real Examples

### Example 1: Running Different Web Servers

**Try Apache web server:**
```bash
docker run -d -p 8080:80 --name apache-server httpd
```

Visit `http://localhost:8080` - you'll see "It works!" page.

**Try nginx web server:**
```bash
docker run -d -p 8081:80 --name nginx-server nginx
```

Visit `http://localhost:8081` - you'll see nginx welcome page.

**Now you have TWO web servers running from different images!**

### Example 2: Running a Database

```bash
docker run -d --name my-database -e MYSQL_ROOT_PASSWORD=mypassword mysql
```

**What's happening:**
- `mysql` image contains a complete MySQL database
- `-e MYSQL_ROOT_PASSWORD=mypassword` sets up the database password
- The database is now running in the background!

### Example 3: Interactive Python Environment

```bash
docker run -it python:3.9
```

**What's happening:**
- `-it` means "interactive terminal"
- You get a Python prompt inside the container
- Type `exit()` to leave

## Understanding Image Names and Tags

### Full Image Name Format

```
registry/username/repository:tag
```

**Examples:**
- `nginx:latest` = `docker.io/library/nginx:latest` (full name)
- `mysql:8.0` = Official MySQL version 8.0
- `node:16-alpine` = Node.js version 16 on Alpine Linux (smaller size)

### Common Tags You'll See

- **latest** - The newest stable version
- **alpine** - Smaller, lightweight version (based on Alpine Linux)
- **slim** - Smaller version with fewer features
- **Version numbers** - `3.9`, `8.0`, `v1.2.3` (specific versions)

!!! tip "Pro Tip: Avoid 'latest' in production"
    For real projects, always use specific version tags like `nginx:1.21` instead of `nginx:latest`. This prevents surprises when newer versions come out!

## Image Management Commands

### See Detailed Information About an Image

```bash
docker inspect nginx
```

This shows you **everything** about the image - like reading the nutrition label on food!

### Search for Images on Docker Hub

```bash
docker search python
```

### See Image History (How It Was Built)

```bash
docker history nginx
```

Shows all the layers that make up the image.

### Clean Up Unused Images

```bash
# Remove unused images
docker image prune

# Remove ALL unused images (be careful!)
docker image prune -a
```

## Hands-On Practice Exercise

Let's put it all together! Follow these steps:

### Step 1: Download Multiple Images

```bash
docker pull nginx
docker pull python:3.9
docker pull redis
```

### Step 2: Check What You Downloaded

```bash
docker images
```

You should see all three images listed.

### Step 3: Run Containers from Each Image

```bash
# Web server
docker run -d -p 8080:80 --name web nginx

# Python environment (interactive)
docker run -it --name python-env python:3.9

# (Type exit() to leave Python, then continue)

# Redis database
docker run -d --name cache redis
```

### Step 4: See Everything Running

```bash
docker ps
```

### Step 5: Clean Up

```bash
# Stop all containers
docker stop web python-env cache

# Remove all containers
docker rm web python-env cache

# Remove images if you want
docker rmi nginx python:3.9 redis
```

## Common Mistakes Beginners Make

### Mistake 1: Confusing Images and Containers

❌ **Wrong thinking:** "I deleted the nginx container, now nginx is gone"  
✅ **Correct:** The nginx image is still there, you can create new containers anytime

### Mistake 2: Using 'latest' Tag Everywhere

❌ **Risky:** `docker run mysql` (gets whatever version is "latest")  
✅ **Better:** `docker run mysql:8.0` (gets specific version you tested)

### Mistake 3: Not Understanding Layers

Images are built in **layers** like a cake:
- Base layer (operating system)
- App layer (your application)
- Config layer (settings)

This makes images efficient - multiple images can share the same base layers!

## Quick Reference

| **Task** | **Command** | **Example** |
|----------|-------------|-------------|
| List images | `docker images` | - |
| Download image | `docker pull <image>` | `docker pull nginx` |
| Download specific version | `docker pull <image>:<tag>` | `docker pull python:3.9` |
| Remove image | `docker rmi <image>` | `docker rmi nginx` |
| Search images | `docker search <term>` | `docker search python` |
| Image details | `docker inspect <image>` | `docker inspect nginx` |
| Image history | `docker history <image>` | `docker history nginx` |
| Clean unused images | `docker image prune` | - |

## Key Takeaways

🎯 **Images are blueprints** - they contain everything needed to run an application

🎯 **Containers are instances** - running copies created from images

🎯 **One image, many containers** - you can run multiple containers from the same image

🎯 **Images have versions** - use specific tags for predictable results

🎯 **Docker Hub is your friend** - thousands of pre-built images available

## Conclusion

Congratulations! You now understand Docker images - the building blocks of everything in Docker. You've learned how to find, download, run, and manage images. 

Think of images as your **toolkit** - once you understand how to use them, you can run almost any application without worrying about installation headaches!

## What's Next?

Now that you're comfortable with images, it's time to dive deeper into **containers** - the running instances of your images. You'll learn:

- **[Working with Containers](containers.md)** - Advanced container management, connecting to running containers, environment variables, and networking
- **[Creating Dockerfiles](dockerfile.md)** - How to build your own custom images from scratch

Ready to become a container master? Let's keep going! 🚀