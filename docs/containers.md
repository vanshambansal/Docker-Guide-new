# Docker Containers

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

Let's keep going and become a Docker pro! 🚀
