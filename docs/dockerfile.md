# Creating Dockerfiles

A **Dockerfile** is like a recipe for building your own Docker images.  
It tells Docker exactly how to set up your application, step by step.

## What Is a Dockerfile?

A Dockerfile is a plain text file with instructions for building a Docker image.  
Each line in the Dockerfile is a command that adds something to your image.

**Example:**
```dockerfile
FROM python:3.9
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

## Basic Dockerfile Instructions

- **FROM**: Start from an existing image (base image)
- **WORKDIR**: Set the working directory inside the image
- **COPY**: Copy files from your computer into the image
- **RUN**: Run a command (like installing packages)
- **CMD**: Set the default command to run when the container starts

## Step-by-Step: Build Your Own Image

Let's create a simple Python app and Dockerize it!

### 1. Create Your App

Make a file called `app.py`:
```python
print("Hello from Docker!")
```

### 2. Write a Dockerfile

```dockerfile
FROM python:3.9
WORKDIR /app
COPY . .
CMD ["python", "app.py"]
```

### 3. Build the Image

```bash
docker build -t my-python-app .
```

### 4. Run a Container from Your Image

```bash
docker run my-python-app
```

You should see:  
```
Hello from Docker!
```

## More Useful Dockerfile Instructions

- **EXPOSE**: Document which ports the app uses (e.g., `EXPOSE 80`)
- **ENV**: Set environment variables (e.g., `ENV DEBUG=true`)
- **ARG**: Define build-time variables

## Tips for Writing Dockerfiles

- Start with an official base image (like `python`, `node`, `nginx`)
- Only copy the files you need
- Use specific versions for base images (avoid `latest`)
- Keep your Dockerfile simple and readable

## Common Mistakes

- Forgetting to set the working directory (`WORKDIR`)
- Not copying all required files
- Using `latest` tag for base images (can cause surprises)
- Not cleaning up temporary files (can make images large)

## Quick Reference

| **Instruction** | **What It Does**                | **Example**                   |
|-----------------|---------------------------------|-------------------------------|
| FROM            | Base image                      | `FROM node:18`                |
| WORKDIR         | Set working directory           | `WORKDIR /app`                |
| COPY            | Copy files into image           | `COPY . .`                    |
| RUN             | Run command during build        | `RUN pip install -r req.txt`  |
| CMD             | Default command for container   | `CMD ["node", "index.js"]`    |
| EXPOSE          | Document port                   | `EXPOSE 8080`                 |
| ENV             | Set environment variable        | `ENV NODE_ENV=production`     |

## Key Takeaways

🎯 **Dockerfiles let you create your own images**  
🎯 **Each instruction adds a layer to your image**  
🎯 **Keep Dockerfiles simple and use official base images**  
🎯 **Build and run your images with `docker build` and `docker run`**

## Conclusion

Now you know how to write Dockerfiles and build your own images!  
This is a key skill for packaging and sharing your applications.

## What's Next?

Next, you'll learn how to run **multi-container applications** using **Docker Compose**.  
After that, you'll discover how to **persist data** using **Docker Volumes**.

- **[Docker Compose](compose.md)** - Manage multi-container apps easily
- **[Docker Volumes](volumes.md)** - Store and share data between containers

Keep going—you're becoming a Docker expert! 🚀
