# Your First Steps with Docker 👋

Welcome to your first hands-on Docker experience! We're going to run some containers and see Docker's magic in action. Don't worry about understanding everything yet - just follow along and be amazed! ✨

!!! tip "Before We Start"
    Make sure you've completed the [Installation](installation.md) and Docker is running on your computer. You should see the Docker whale icon in your system tray (Windows/Mac) or Docker service running (Linux).

---

## Step 1: Verify Docker is Working

Let's make sure everything is set up correctly:

```bash
docker --version
```

**Expected output:**
```
Docker version 24.0.7, build afdd53b
```

!!! success "Great! Docker is Ready"
    The exact version number might be different - that's totally fine! As long as you see "Docker version" followed by numbers, you're good to go.

---

## Step 2: Your First Container - Hello World! 🌍

Time for the classic "Hello World" moment in Docker:

```bash
docker run hello-world
```

**What you'll see:**
```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
c1ec31eb5944: Pull complete 
Digest: sha256:4bd78111b6914a99dbc560e6a20eab57c1d6725a8e87bb9a8b52e21ef7c55fdd
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

**🎉 Congratulations!** You just ran your first Docker container!

### What Just Happened? (The Magic Explained)

1. **Docker looked** for the "hello-world" image on your computer
2. **Didn't find it**, so Docker downloaded it from Docker Hub (like an app store)
3. **Created a container** from that image
4. **Ran the program** inside the container (which printed the message)
5. **Container finished** and stopped automatically

!!! info "Think of it like this"
    You just asked Docker to "run the hello-world app" and it automatically:
    
    - Downloaded the app if you didn't have it
    - Set up a perfect environment for it to run
    - Ran it and showed you the results
    - Cleaned up when done
    
    **All with one simple command!**

---

## Step 3: Run a Web Server in 30 Seconds 🚀

Now let's do something more impressive - run a complete web server:

```bash
docker run -d -p 8080:80 --name my-web-server nginx
```

**Expected output:**
```
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
31b3f1ad4ce1: Pull complete 
fd42b079d0f8: Pull complete 
30585fbbeedd: Pull complete 
18f4ffdd6490: Pull complete 
9dc932c8fba2: Pull complete 
9a59d19f9c5b: Pull complete 
9db7feffab07: Pull complete 
Digest: sha256:add4792d930c25dd2abf2ef9ea79e3d3ae1b4cd3b93cb4a76e6e36b87b56ce8
Status: Downloaded newer image for nginx:latest
a1b2c3d4e5f67890abcdef1234567890abcdef1234567890abcdef1234567890
```

### 🌐 **Test Your Web Server:**

Open your web browser and go to: **[http://localhost:8080](http://localhost:8080)**

**You should see:**
```
Welcome to nginx!
If you see this page, the nginx web server is successfully installed and working.
```

**🎉 AMAZING!** You just deployed a web server with one command!

### Breaking Down the Command:

```bash
docker run -d -p 8080:80 --name my-web-server nginx
```

- `docker run` = Create and start a new container
- `-d` = Run in "detached" mode (in the background)
- `-p 8080:80` = Connect your computer's port 8080 to container's port 80
- `--name my-web-server` = Give the container a friendly name
- `nginx` = Use the nginx web server image

!!! tip "Port Mapping Explained"
    **Think of ports like apartment numbers:**
    
    - Your computer (building) has port 8080 (apartment 8080)
    - Container (tenant) is using port 80 (their internal apartment number)  
    - `-p 8080:80` connects them so visitors to 8080 reach the container's 80

---

## Step 4: See What's Running 👀

Let's check what containers are currently running:

```bash
docker ps
```

**Expected output:**
```
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                  NAMES
a1b2c3d4e5f6   nginx     "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes   0.0.0.0:8080->80/tcp   my-web-server
```

**What this tells you:**
- **CONTAINER ID**: Unique identifier (a1b2c3d4e5f6)
- **IMAGE**: Using the nginx image
- **STATUS**: Up 2 minutes (running for 2 minutes)
- **PORTS**: Your port 8080 connects to container's port 80
- **NAMES**: Friendly name you gave it (my-web-server)

---

## Step 5: Look Inside Your Container 🔍

Want to peek inside the running container? Let's run a command inside it:

```bash
docker exec my-web-server ls /usr/share/nginx/html
```

**Expected output:**
```
50x.html
index.html
```

**What just happened?** You ran the `ls` command inside the container to see the web server's files!

### Try Another One:

```bash
docker exec my-web-server cat /usr/share/nginx/html/index.html
```

You'll see the HTML code for the "Welcome to nginx!" page you viewed in your browser!

!!! info "Think of `docker exec` like remote control"
    It's like having a remote control for your container - you can run commands inside it from the outside!

---

## Step 6: Check the Logs 📋

Let's see what your web server has been up to:

```bash
docker logs my-web-server
```

**You might see something like:**
```
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
...
2024/01/15 10:30:45 [notice] 1#1: nginx/1.25.3
2024/01/15 10:30:45 [notice] 1#1: built by gcc 12.2.0
2024/01/15 10:30:45 [notice] 1#1: OS: Linux 5.15.0-88-generic
2024/01/15 10:30:45 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2024/01/15 10:30:45 [notice] 1#1: start worker processes
```

**Now refresh your browser** at [http://localhost:8080](http://localhost:8080) and run `docker logs my-web-server` again. You'll see new log entries showing your visit!

---

## Step 7: Stop and Clean Up 🧹

When you're done experimenting, let's clean up:

### Stop the Container:
```bash
docker stop my-web-server
```

**Expected output:**
```
my-web-server
```

### Verify it's Stopped:
```bash
docker ps
```

You should see no running containers (empty output).

### Remove the Container:
```bash
docker rm my-web-server
```

**Expected output:**
```
my-web-server
```

### Check All Containers (including stopped ones):
```bash
docker ps -a
```

The `my-web-server` container should now be gone!

---

## Step 8: Try Something Interactive 🎮

Let's run a container you can interact with directly:

```bash
docker run -it ubuntu bash
```

**What happens:**
- Downloads Ubuntu Linux image
- Starts a container
- Gives you a bash shell **inside** the Ubuntu container
- You're now "inside" a Ubuntu Linux system!

**You'll see a prompt like:**
```
root@a1b2c3d4e5f6:/#
```

### Try Some Commands Inside Ubuntu:
```bash
# Check what Linux distribution you're in
cat /etc/os-release

# See the current directory
pwd

# List files
ls

# Check running processes
ps aux
```

### Exit the Container:
```bash
exit
```

You're back to your regular computer! The Ubuntu container stopped automatically when you exited.

!!! tip "What Just Happened?"
    You had a **complete Ubuntu Linux system** running inside a container on your computer - totally isolated from your main system. When you exited, it all disappeared. Magic! ✨

---

## Understanding What You Accomplished

In less than 15 minutes, you:

### ✅ **Verified Docker Works**
- Confirmed your installation is correct
- Learned the basic `docker` command structure

### ✅ **Ran Multiple Applications**  
- Hello-world program
- Nginx web server
- Ubuntu Linux system

### ✅ **Learned Key Commands**
- `docker run` - Start containers
- `docker ps` - See what's running  
- `docker exec` - Run commands inside containers
- `docker logs` - See container output
- `docker stop` - Stop containers
- `docker rm` - Remove containers

### ✅ **Experienced Docker's Power**
- **Instant deployment** - Web server in one command
- **Perfect isolation** - Each container is separate
- **No conflicts** - Multiple versions can coexist
- **Easy cleanup** - Remove everything when done

---

## Common First-Time Questions

### "Why didn't I need to install nginx or Ubuntu?"

**Docker images contain everything needed!** The nginx image includes:
- The nginx web server software
- All required libraries and dependencies  
- A complete operating system (Linux)
- Pre-configured settings

### "Where did these images come from?"

**Docker Hub** - it's like an app store for Docker images. When you run `docker run nginx`, Docker automatically:
1. Checks if you have the nginx image locally
2. If not, downloads it from Docker Hub
3. Uses it to create your container

### "Are containers using a lot of resources?"

**Surprisingly little!** Containers are very lightweight:
- They share your computer's operating system kernel
- Only use resources when actively running
- Start and stop in seconds
- Much lighter than virtual machines

### "Is this safe?"

**Yes!** Containers are isolated:
- Can't access your personal files (unless you specifically allow it)
- Can't affect other containers
- Can't harm your main operating system
- Easy to remove completely

---

## What's Next?

You've just experienced Docker's core value - **running applications anywhere, instantly, without conflicts**. 

Ready to understand how this all works? Let's dive deeper:

<div class="grid cards" markdown>

-   🐳 **Understand the Fundamentals**

    ---
    
    Learn what images and containers really are.
    
    [Docker Basics →](basics.md){ .md-button .md-button--primary }

-   📦 **Master Docker Images**

    ---
    
    Dive deep into the building blocks of containers.
    
    [Learn About Images →](../core-concepts/images.md){ .md-button }

-   🎮 **Try Fun Projects**

    ---
    
    Build something cool with what you've learned!
    
    [Fun Examples →](../practical-guides/fun-examples.md){ .md-button }

</div>

---

!!! success "You're officially a Docker user!"
    You've just done what many developers find intimidating - and it was actually pretty easy! Docker's power comes from making complex deployment simple. You're ready for the next level!

**Keep going - you're doing great! 🚀**