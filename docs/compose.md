# Docker Compose

Docker Compose lets you **define and run multi-container Docker applications** easily.  
Instead of starting each container manually, you describe your whole app in a single file and run everything with one command!

## Why Use Docker Compose?

- Manage multiple containers as one application
- Define services, networks, and volumes in one place
- Easy to start, stop, and manage complex setups

## What Is a docker-compose.yml File?

It's a YAML file where you describe all your app's services (containers), networks, and volumes.

**Example:**
```yaml
version: "3"
services:
  web:
    image: nginx
    ports:
      - "8080:80"
  redis:
    image: redis
```

This file defines two services: a web server (nginx) and a redis database.

## Basic Commands

- **Start everything:**  
  ```bash
  docker compose up
  ```
- **Stop everything:**  
  ```bash
  docker compose down
  ```
- **Run in background:**  
  ```bash
  docker compose up -d
  ```

## Hands-On: Try Docker Compose

1. **Create a `docker-compose.yml` file:**
    ```yaml
    version: "3"
    services:
      web:
        image: nginx
        ports:
          - "8080:80"
      redis:
        image: redis
    ```

2. **Start your app:**
    ```bash
    docker compose up
    ```

3. **Visit** `http://localhost:8080` to see nginx running.

4. **Stop everything:**
    ```bash
    docker compose down
    ```

## Common Compose File Options

- `image`: Which image to use
- `build`: Build from a Dockerfile
- `ports`: Map ports (`host:container`)
- `volumes`: Attach storage
- `environment`: Set environment variables
- `depends_on`: Control startup order

## Quick Reference

| **Task**         | **Command**              |
|------------------|-------------------------|
| Start services   | `docker compose up`     |
| Stop services    | `docker compose down`   |
| Run in background| `docker compose up -d`  |
| View logs        | `docker compose logs`   |

## Key Takeaways

🎯 **Compose makes multi-container apps easy**  
🎯 **Define everything in one file**  
🎯 **Start and stop your whole app with one command**

## What's Next?

Next, you'll learn how to **store and share data** between containers using **Docker Volumes**.

- **[Docker Volumes](volumes.md)** - Persist and manage your data

Keep going—you're almost a Docker pro! 🚀
