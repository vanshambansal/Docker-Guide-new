# Installing Docker

Before we start containerizing applications, we need Docker installed on your machine. This guide will get you up and running quickly on any operating system!

## Quick Check: Is Docker Already Installed?

Let's first see if Docker is already on your computer:

!!! tip "Opening Terminal"
    - **Windows:** Press `Win + R`, type `cmd`, and press Enter  
    - **Mac:** Press `Cmd + Space`, type `terminal`, and press Enter  
    - **Linux:** Press `Ctrl + Alt + T`

```bash
docker --version
```

**If you see output like this:**
```
Docker version 27.3.1, build ce12230
```

🎉 **Great! Docker is already installed.** Skip to [Verify Installation](#verify-installation).

**If you get an error** like `command not found` or `docker is not recognized`, continue with the installation steps below.

---

## Choose Your Operating System

Select your operating system for specific installation instructions:

=== "Windows"

    ### System Requirements
    
    - Windows 10 64-bit (Pro, Enterprise, or Education) Build 19041 or higher
    - OR Windows 11 64-bit (any edition)  
    - At least 4GB RAM
    - Virtualization enabled in BIOS

    ### Installation Steps

    1. **Download Docker Desktop:**
       - Visit [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
       - Click **"Download for Windows"**

    2. **Run the installer:**
       - Double-click the downloaded `Docker Desktop Installer.exe`
       - **Important:** Check **"Enable WSL 2 Windows Features"** during installation
       - Optional: Check **"Add shortcut to desktop"**

    3. **Complete installation:**
       - Restart your computer when prompted
       - After restart, Docker Desktop should start automatically
       - Look for the Docker whale icon in your system tray

    4. **First-time setup:**
       - Accept the Docker subscription service agreement
       - You may be prompted to enable WSL 2 - click **"Restart"** if asked

    ### Windows Troubleshooting

    !!! warning "WSL 2 Issues"
        If you get "WSL 2 installation is incomplete":
        
        1. Open PowerShell as Administrator
        2. Run: `wsl --install`
        3. Restart your computer
        4. Start Docker Desktop again

    **Virtualization not enabled:**
    1. Restart computer and enter BIOS (usually F2, F12, or Del key during startup)
    2. Find "Virtualization Technology" or "Intel VT-x/AMD-V" 
    3. Enable it and save changes

=== "Mac"

    ### System Requirements
    
    - macOS 10.15 (Catalina) or newer
    - At least 4GB RAM

    ### Installation Steps

    1. **Choose the right version:**
       - **Intel Mac:** Download "Docker Desktop for Mac with Intel chip"
       - **Apple Silicon (M1/M2/M3):** Download "Docker Desktop for Mac with Apple chip"
       - Visit [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)

    2. **Install Docker:**
       - Open the downloaded `.dmg` file
       - Drag the Docker icon to your Applications folder
       - This copies Docker to your system

    3. **Launch Docker Desktop:**
       - Go to Applications folder
       - Double-click Docker
       - You'll see a whale icon appear in your menu bar when it's running

    4. **Grant permissions:**
       - macOS will ask for permissions - click **"Allow"**
       - You may need to enter your password
       - Docker needs these permissions to function properly

    ### Mac Troubleshooting

    !!! note "First Launch"
        Docker Desktop may take 2-3 minutes to start the first time. The whale icon will stop animating when ready.

    **Docker Desktop won't start:**
    - Check available disk space (Docker needs at least 2GB free)
    - Restart your Mac and try again
    - Verify macOS version is 10.15 or newer

=== "Linux"

    ### For Ubuntu/Debian

    **Option 1: Convenience Script (Recommended for beginners)**

    ```bash
    # Download and run Docker's installation script
    curl -fsSL https://get.docker.com -o get-docker.sh
    sudo sh get-docker.sh
    ```

    **Option 2: Manual Installation (More control)**

    ```bash
    # Update package index
    sudo apt update

    # Install prerequisite packages
    sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release

    # Add Docker's official GPG key
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

    # Add Docker repository
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

    # Update package index again
    sudo apt update

    # Install Docker Engine and related tools
    sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```

    ### Post-Installation Setup (Important!)

    **Start Docker service:**
    ```bash
    sudo systemctl start docker
    sudo systemctl enable docker  # Start automatically on boot
    ```

    **Add your user to docker group (avoids needing sudo):**
    ```bash
    sudo usermod -aG docker $USER
    ```

    !!! important "Must Restart Session"
        After adding yourself to the docker group, **log out and log back in** (or restart) for changes to take effect!

    ### Linux Troubleshooting

    **Permission denied errors:**
    ```bash
    # Check if you're in docker group
    groups $USER
    
    # If 'docker' is not listed, add yourself
    sudo usermod -aG docker $USER
    # Then log out and back in
    ```

    **Docker daemon not running:**
    ```bash
    sudo systemctl status docker  # Check status
    sudo systemctl start docker   # Start if stopped
    ```

---

## Verify Installation

Now let's confirm Docker is working correctly:

### Step 1: Check Version
```bash
docker --version
```

**Expected output:**
```
Docker version 24.0.7, build afdd53b
```

### Step 2: Check Docker Engine
```bash
docker info
```

This should display detailed information about your Docker installation without errors.

### Step 3: Test with Hello World
```bash
docker run hello-world
```

**If successful, you'll see:**
```
Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```

!!! success "Installation Complete!"
    If all three tests pass, Docker is properly installed and ready to use!

---

## Understanding Your Docker Installation

### What Got Installed?

- **Docker Engine:** Core runtime that manages containers
- **Docker CLI:** Command-line interface for interacting with Docker
- **Docker Desktop** (Windows/Mac): Graphical management interface
- **Docker Compose:** Tool for multi-container applications (included)

### Resource Usage

Docker uses system resources efficiently:

- **RAM:** 2-4GB allocated by default (configurable)
- **Disk Space:** Starts minimal, grows with images and containers
- **CPU:** Only active when containers are running

**Adjust resources:** Open Docker Desktop → Settings → Resources (Windows/Mac)

---

## Common Installation Issues

### Issue: Docker Desktop won't start

**Windows:**
- Ensure WSL 2 is enabled: `wsl --list --verbose`
- Check virtualization is enabled in BIOS
- Run Docker Desktop as Administrator

**Mac:**
- Check system requirements (macOS 10.15+)
- Ensure sufficient disk space (2GB+ free)
- Try restarting your Mac

### Issue: "docker: command not found"

**All systems:**
- Restart your terminal/command prompt
- On Linux: ensure Docker service is running
- Check PATH includes Docker installation directory

### Issue: Permission denied (Linux)

```bash
# Add user to docker group
sudo usermod -aG docker $USER

# Verify group membership
groups $USER

# If docker group missing, log out and back in
```

---

## What's Next?

Now that Docker is installed and verified, you're ready to start your container journey!

**Recommended next steps:**

<div class="grid cards" markdown>

-   👋 **Take Your First Steps**

    ---
    
    Run your first containers and see Docker in action.
    
    [First Steps →](first-steps.md){ .md-button .md-button--primary }

-   🐳 **Learn the Basics**

    ---
    
    Understand core Docker concepts and commands.
    
    [Docker Basics →](basics.md){ .md-button }

-   📚 **Get Help When Stuck**

    ---
    
    Troubleshooting guide for common issues.
    
    [Troubleshooting →](../practical-guides/troubleshooting.md){ .md-button }

</div>

---

!!! tip "Installation Tips"
    - **Windows users:** WSL 2 provides the best performance
    - **Mac users:** Choose the correct chip architecture (Intel vs Apple Silicon)  
    - **Linux users:** The docker group setup eliminates need for sudo
    - **All users:** Docker Desktop provides a nice GUI for beginners

Your Docker installation is complete and ready for action! 🚀