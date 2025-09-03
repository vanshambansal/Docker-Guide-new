# Fun Docker Examples

Let's try some cool and fun things you can do with Docker!

## 1. Run a Tiny Web Server in One Command

```bash
docker run -d -p 8080:80 --name hello-web nginx
```
Visit [http://localhost:8080](http://localhost:8080) and you'll see the nginx welcome page.

## 2. Try the "Hello World" Container

```bash
docker run hello-world
```
Docker will print a welcome message to show everything is working.

## 3. Launch an Interactive Python Shell

```bash
docker run -it python:3.9
```
You can use Python right inside the container. Type `exit()` to leave.

## 4. Run a Temporary Alpine Linux Shell

```bash
docker run -it --rm alpine sh
```
Try out Linux commands in a super lightweight container.

## 5. Play With Fortune and Cowsay

```bash
docker run --rm docker/whalesay cowsay "Docker is fun!"
```
See a talking whale in your terminal!

## 6. Serve Your Current Folder With Python

```bash
docker run -d -p 8000:8000 -v ${PWD}:/usr/src/app -w /usr/src/app python:3.9 python -m http.server
```
Visit [http://localhost:8000](http://localhost:8000) to browse your files.

---

## What's Next?

Now that you've had some fun, let's look at **advanced Docker topics and best practices** to level up your skills!

- **[Best Practices](best-practices.md)**
- **[Advanced Topics](advanced.md)**
