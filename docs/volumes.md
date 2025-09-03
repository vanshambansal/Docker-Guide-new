# Docker Volumes

Docker volumes let you **store data outside your containers**.  
This means your data stays safe even if you delete or recreate containers!

## Why Use Volumes?

- Persist data (like databases, uploads, logs)
- Share data between containers
- Keep your data safe when containers are removed or updated

## How to Use Volumes

### 1. Named Volumes

Docker manages these for you.

```bash
docker run -d --name db -v mydata:/var/lib/mysql mysql
```
- `mydata` is the volume name
- `/var/lib/mysql` is the path inside the container

### 2. Bind Mounts

Use a specific folder on your computer.

```bash
docker run -d -v /path/on/host:/data busybox
```

## Hands-On: Try Volumes

1. **Start a container with a volume:**
    ```bash
    docker run -d --name my-redis -v redisdata:/data redis
    ```

2. **Write some data (from inside the container):**
    ```bash
    docker exec -it my-redis redis-cli
    set mykey hello
    exit
    ```

3. **Remove the container:**
    ```bash
    docker rm -f my-redis
    ```

4. **Start a new container with the same volume:**
    ```bash
    docker run -d --name new-redis -v redisdata:/data redis
    ```

5. **Check your data is still there!**

## Useful Volume Commands

- List volumes:  
  ```bash
  docker volume ls
  ```
- Inspect a volume:  
  ```bash
  docker volume inspect mydata
  ```
- Remove a volume:  
  ```bash
  docker volume rm mydata
  ```

## Common Mistakes

- Not using volumes for important data (risk of data loss)
- Confusing named volumes and bind mounts
- Forgetting to clean up unused volumes

## Quick Reference

| **Task**         | **Command**                        |
|------------------|------------------------------------|
| Create volume    | `docker volume create mydata`      |
| Use volume       | `-v mydata:/path/in/container`     |
| List volumes     | `docker volume ls`                 |
| Remove volume    | `docker volume rm mydata`          |

## Key Takeaways

🎯 **Volumes keep your data safe**  
🎯 **Use volumes for databases and important files**  
🎯 **You can share volumes between containers**

## What's Next?

Next, you'll learn about **best practices** for working with Docker, and see some **fun examples** to try out!

- **[Best Practices](best-practices.md)** - Tips for real-world Docker use
- **[Fun Examples](fun-examples.md)** - Cool things you can do with Docker

You're almost at the finish line! 🚀
