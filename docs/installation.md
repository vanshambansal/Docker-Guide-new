# Installing Docker

Welcome! Before we start playing with containers, we need Docker installed on your machine. Don't worry - it's easier than you think!

## Quick Check: Is Docker Already Installed?

First, let's see if Docker is already on your computer. Open your terminal and type:

!!! tip "Opening Terminal"
    - **Windows:** Press `Win + R`, type `cmd`, and press Enter  
    - **Mac:** Press `Cmd + Space`, type `terminal`, and press Enter  
    - **Linux:** Press `Ctrl + Alt + T`

```bash
docker --version
```

If you see something like this:

```
Docker version 27.3.1, build ce12230
```

🎉 **Great! Docker is already installed.** Skip to [Test Your Installation](#test-your-installation).

If you get an error like `command not found`, keep reading!

## Installing Docker Desktop

### For Windows 10/11

**Requirements:**

- Windows 10 64-bit (Pro, Enterprise, or Education) Build 19041 or higher
- OR Windows 11 64-bit (any edition)
- WSL 2 feature enabled
- Virtualization enabled in BIOS

**Step-by-step:**

1. **Download Docker Desktop:**
   - Go to [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
   - Click **"Download for Windows"**

2. **Run the installer:**
   - Double-click the downloaded `.exe` file
   - Check **"Enable WSL 2 Windows Features"** during installation
   - Check **"Add shortcut to desktop"** (optional)

3. **Restart your computer** when prompted

4. **Start Docker Desktop:**
   - Look for the Docker whale icon in your system tray
   - If it's not running, search for "Docker Desktop" and open it

5. **Accept the license agreement** when it appears

!!! warning "Windows Troubleshooting"
    If you get a "WSL 2 installation is incomplete" error, run this in PowerShell as Administrator:
    ```powershell
    wsl --install
    ```
    Then restart and try again.

### For Mac

**Requirements:**

- macOS 10.15 or newer
- At least 4GB RAM

**Step-by-step:**

1. **Download Docker Desktop:**
   - Go to [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
   - Choose your Mac type:
     - **Intel Mac:** Download "Mac with Intel chip"
     - **Apple Silicon (M1/M2/M3):** Download "Mac with Apple chip"

2. **Install Docker:**
   - Open the downloaded `.dmg` file
   - Drag the Docker icon to your Applications folder

3. **Launch Docker Desktop:**
   - Go to Applications and double-click Docker
   - You'll see a whale icon in your menu bar when it's running

4. **Grant permissions** when macOS asks (Docker needs system access)

!!! note "Mac Note"
    The first time you run Docker, it might take a few minutes to start up. Be patient!

### For Linux (Ubuntu/Debian)

**Option 1: Easy Script (Recommended for beginners)**

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

**Option 2: Manual Installation**

```bash
# Update package list
sudo apt update

# Install prerequisites
sudo apt install apt-transport-https ca-certificates curl gnupg lsb-release

# Add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Add Docker repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update package list again
sudo apt update

# Install Docker
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

**Start Docker service:**

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

**Add your user to docker group (so you don't need sudo):**

```bash
sudo usermod -aG docker $USER
```

!!! important "Linux Important"
    After adding yourself to the docker group, **log out and log back in** for this to take effect!

## Test Your Installation

Now let's make sure everything works! Run this command:

```bash
docker --version
```

You should see something like:

```
Docker version 27.3.1, build ce12230
```

**Next, let's run your first container:**

```bash
docker run hello-world
```

If everything is working, you'll see a message like this:

```
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
```

🎉 **Congratulations! Docker is working perfectly!**

## Common Issues & Solutions

### "Docker daemon is not running"

**Windows/Mac:** Make sure Docker Desktop is running (look for the whale icon in system tray/menu bar)

**Linux:** Start the service:
```bash
sudo systemctl start docker
```

### "Permission denied" (Linux only)

Make sure you added your user to the docker group:
```bash
sudo usermod -aG docker $USER
```
Then log out and log back in, or use `sudo` before docker commands.

### "WSL 2 installation incomplete" (Windows)

Open PowerShell as Administrator and run:
```powershell
wsl --install
```
Then restart your computer.

### Docker Desktop won't start (Mac)

- Check if you have enough disk space (Docker needs at least 2GB)
- Try restarting your Mac
- Make sure you're running a supported macOS version (10.15+)

### "Virtualization not enabled" (Windows)

You need to enable virtualization in your BIOS/UEFI settings:

1. Restart your computer
2. Enter BIOS (usually F2, F12, or Del during startup)
3. Look for "Virtualization Technology" or "Intel VT-x/AMD-V"
4. Enable it and save changes

## Understanding What You Just Installed

When you installed Docker, you actually got several components:

- **Docker Engine:** The core that runs containers
- **Docker CLI:** The command-line interface you'll use
- **Docker Desktop:** The graphical interface (Windows/Mac only)
- **Docker Compose:** Tool for running multi-container applications

!!! info "Docker Desktop vs Docker Engine"
    - **Docker Desktop** = Easy GUI + Docker Engine (Windows/Mac)
    - **Docker Engine** = Just the core Docker (Linux)
    - Both work the same way from the command line!

## Resource Usage

Docker will use some of your computer's resources:

- **RAM:** Usually 2-4GB by default
- **Disk:** Space for images and containers (starts small, grows as needed)
- **CPU:** Only when containers are running

You can adjust these settings in Docker Desktop preferences if needed.

## What's Next?

Now that Docker is installed, you're ready to start learning! Here's what we'll cover:

1. **[Docker Basics](basics.md)** - Understanding images and containers
2. **[Building Images](images.md)** - Creating your own Docker images
3. **[Your First Container](containers.md)** - Running and managing containers

---

!!! success "You're Ready!"
    Docker is now installed and working on your machine. Let's move on to learning the basics!