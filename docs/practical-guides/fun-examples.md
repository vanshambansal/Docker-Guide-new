# Fun Docker Projects

Learn Docker through engaging, hands-on projects that demonstrate real-world applications. These examples combine learning with practical outcomes you can actually use and show off to others.

## Web Development Projects

### Personal Portfolio Website

Build and deploy a static website using Docker:

```dockerfile
# Dockerfile
FROM nginx:alpine

# Copy your website files
COPY index.html /usr/share/nginx/html/
COPY css/ /usr/share/nginx/html/css/
COPY js/ /usr/share/nginx/html/js/
COPY images/ /usr/share/nginx/html/images/

# Custom nginx configuration (optional)
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**Create a simple HTML file:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>My Docker Portfolio</title>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; padding: 50px; }
        .container { max-width: 800px; margin: 0 auto; }
        h1 { color: #2496ED; }
    </style>
</head>
<body>
    <div class="container">
        <h1>Welcome to My Docker-Powered Portfolio!</h1>
        <p>This website is running inside a Docker container.</p>
        <p>Built with Docker, deployed anywhere!</p>
    </div>
</body>
</html>
```

**Build and run:**
```bash
docker build -t my-portfolio .
docker run -d -p 8080:80 --name portfolio my-portfolio
```

Visit `http://localhost:8080` to see your containerized website!

### Multi-Container Blog Platform

Create a WordPress blog with database using Docker Compose:

```yaml
# docker-compose.yml
version: '3.8'
services:
  wordpress:
    image: wordpress:latest
    ports:
      - "8000:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppassword
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wordpress_data:/var/www/html
    depends_on:
      - db

  db:
    image: mysql:5.7
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppassword
      MYSQL_ROOT_PASSWORD: rootpassword
    volumes:
      - db_data:/var/lib/mysql

volumes:
  wordpress_data:
  db_data:
```

**Start your blog:**
```bash
docker-compose up -d
```

Access your blog at `http://localhost:8000` and create your first post!

## Development Environment Projects

### Full-Stack Development Environment

Create a complete development environment with hot-reloading:

```yaml
# docker-compose.dev.yml
version: '3.8'
services:
  frontend:
    image: node:18
    working_dir: /app
    volumes:
      - ./frontend:/app
      - node_modules:/app/node_modules
    ports:
      - "3000:3000"
    command: npm start
    environment:
      - CHOKIDAR_USEPOLLING=true

  backend:
    image: node:18
    working_dir: /app
    volumes:
      - ./backend:/app
      - node_modules_backend:/app/node_modules
    ports:
      - "5000:5000"
    command: npm run dev
    environment:
      - NODE_ENV=development

  database:
    image: postgres:13
    environment:
      POSTGRES_DB: devdb
      POSTGRES_USER: developer
      POSTGRES_PASSWORD: devpassword
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  node_modules:
  node_modules_backend:
  postgres_data:
```

### Code Server (VS Code in Browser)

Run VS Code in your browser using Docker:

```bash
docker run -d \
  --name code-server \
  -p 8443:8443 \
  -v "$HOME/.local/share/code-server:/home/coder/.local/share/code-server" \
  -v "$PWD:/home/coder/project" \
  -u "$(id -u):$(id -g)" \
  -e "DOCKER_USER=$USER" \
  codercom/code-server:latest
```

Access your cloud IDE at `http://localhost:8443`

## Game and Entertainment Projects

### Minecraft Server

Host your own Minecraft server with Docker:

```yaml
# minecraft-compose.yml
version: '3.8'
services:
  minecraft:
    image: itzg/minecraft-server
    ports:
      - "25565:25565"
    environment:
      EULA: "TRUE"
      VERSION: "LATEST"
      MEMORY: "2G"
      DIFFICULTY: "easy"
      MAX_PLAYERS: "10"
      MOTD: "My Docker Minecraft Server!"
    volumes:
      - minecraft_data:/data
    restart: unless-stopped

volumes:
  minecraft_data:
```

**Start your server:**
```bash
docker-compose -f minecraft-compose.yml up -d
```

Connect to your server at `localhost:25565`

### Retro Game Emulator

Run classic games in your browser:

```bash
# Run a web-based game emulator
docker run -d \
  --name retro-games \
  -p 8080:80 \
  linuxserver/emulatorjs

# Access at http://localhost:8080
```

## Utility and Tool Projects

### Personal Cloud Storage

Set up your own cloud storage with Nextcloud:

```yaml
# nextcloud-compose.yml
version: '3.8'
services:
  nextcloud:
    image: nextcloud:apache
    ports:
      - "8080:80"
    volumes:
      - nextcloud_data:/var/www/html
      - ./data:/var/www/html/data
    environment:
      MYSQL_HOST: db
      MYSQL_DATABASE: nextcloud
      MYSQL_USER: nextcloud
      MYSQL_PASSWORD: nextcloud_password
    depends_on:
      - db

  db:
    image: mariadb:10.5
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: nextcloud
      MYSQL_USER: nextcloud
      MYSQL_PASSWORD: nextcloud_password
    volumes:
      - db_data:/var/lib/mysql

volumes:
  nextcloud_data:
  db_data:
```

### URL Shortener Service

Create your own URL shortener:

```dockerfile
# Dockerfile for simple URL shortener
FROM python:3.9-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY app.py .
COPY templates/ ./templates/

EXPOSE 5000

CMD ["python", "app.py"]
```

**Simple Flask application:**
```python
# app.py
from flask import Flask, request, redirect, render_template_string
import string
import random

app = Flask(__name__)
url_mapping = {}

def generate_short_url():
    return ''.join(random.choices(string.ascii_letters + string.digits, k=6))

@app.route('/')
def home():
    return '''
    <form method="post" action="/shorten">
        <input type="url" name="url" placeholder="Enter URL" required>
        <button type="submit">Shorten</button>
    </form>
    '''

@app.route('/shorten', methods=['POST'])
def shorten():
    original_url = request.form['url']
    short_code = generate_short_url()
    url_mapping[short_code] = original_url
    return f'Shortened URL: <a href="/{short_code}">http://localhost:5000/{short_code}</a>'

@app.route('/<short_code>')
def redirect_url(short_code):
    original_url = url_mapping.get(short_code)
    if original_url:
        return redirect(original_url)
    return 'URL not found', 404

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
```

## Learning and Educational Projects

### Interactive Python Environment

Create a Jupyter notebook environment:

```bash
docker run -d \
  --name jupyter \
  -p 8888:8888 \
  -v "$PWD":/home/jovyan/work \
  jupyter/scipy-notebook
```

Check logs for the access token:
```bash
docker logs jupyter
```

### Database Learning Environment

Set up multiple databases for learning:

```yaml
# database-lab.yml
version: '3.8'
services:
  postgres:
    image: postgres:13
    environment:
      POSTGRES_DB: learning
      POSTGRES_USER: student
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"

  mysql:
    image: mysql:8.0
    environment:
      MYSQL_DATABASE: learning
      MYSQL_USER: student
      MYSQL_PASSWORD: password
      MYSQL_ROOT_PASSWORD: rootpass
    ports:
      - "3306:3306"

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  mongo:
    image: mongo:5
    environment:
      MONGO_INITDB_DATABASE: learning
    ports:
      - "27017:27017"

  adminer:
    image: adminer
    ports:
      - "8080:8080"
```

Access database management at `http://localhost:8080`

## API and Microservices Projects

### Weather API Service

Build a simple weather API:

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

**Simple weather API:**
```javascript
// server.js
const express = require('express');
const app = express();
const port = 3000;

// Mock weather data
const weatherData = {
  'new-york': { temp: 22, condition: 'sunny' },
  'london': { temp: 15, condition: 'rainy' },
  'tokyo': { temp: 28, condition: 'cloudy' }
};

app.get('/weather/:city', (req, res) => {
  const city = req.params.city.toLowerCase();
  const weather = weatherData[city];
  
  if (weather) {
    res.json({ city, ...weather });
  } else {
    res.status(404).json({ error: 'City not found' });
  }
});

app.listen(port, '0.0.0.0', () => {
  console.log(`Weather API running on port ${port}`);
});
```

### Chat Application

Create a real-time chat using Socket.io:

```yaml
# chat-app.yml
version: '3.8'
services:
  chat-app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    volumes:
      - ./public:/app/public

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
```

## Creative Projects

### Digital Photo Frame

Display rotating images from a folder:

```dockerfile
FROM nginx:alpine

# Install image processing tools
RUN apk add --no-cache imagemagick

# Copy photos and web interface
COPY photos/ /usr/share/nginx/html/photos/
COPY index.html /usr/share/nginx/html/
COPY slideshow.js /usr/share/nginx/html/

EXPOSE 80
```

**Slideshow interface:**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Digital Photo Frame</title>
    <style>
        body { margin: 0; background: black; }
        img { width: 100vw; height: 100vh; object-fit: contain; }
    </style>
</head>
<body>
    <img id="photo" src="" alt="Photo">
    <script>
        const photos = ['photo1.jpg', 'photo2.jpg', 'photo3.jpg'];
        let currentPhoto = 0;
        
        function showNextPhoto() {
            document.getElementById('photo').src = `photos/${photos[currentPhoto]}`;
            currentPhoto = (currentPhoto + 1) % photos.length;
        }
        
        showNextPhoto();
        setInterval(showNextPhoto, 5000); // Change every 5 seconds
    </script>
</body>
</html>
```

### RSS Feed Reader

Create a personal news aggregator:

```yaml
# rss-reader.yml
version: '3.8'
services:
  miniflux:
    image: miniflux/miniflux:latest
    ports:
      - "8080:8080"
    environment:
      DATABASE_URL: postgres://miniflux:password@db/miniflux?sslmode=disable
      RUN_MIGRATIONS: 1
      CREATE_ADMIN: 1
      ADMIN_USERNAME: admin
      ADMIN_PASSWORD: password
    depends_on:
      - db

  db:
    image: postgres:13
    environment:
      POSTGRES_USER: miniflux
      POSTGRES_PASSWORD: password
      POSTGRES_DB: miniflux
    volumes:
      - db_data:/var/lib/postgresql/data

volumes:
  db_data:
```

## Automation Projects

### Automated Backup Service

Create a backup service for your files:

```dockerfile
FROM alpine:latest

RUN apk add --no-cache \
    rsync \
    openssh-client \
    curl

COPY backup.sh /usr/local/bin/backup.sh
RUN chmod +x /usr/local/bin/backup.sh

# Run backup every hour
CMD ["crond", "-f"]
```

**Backup script:**
```bash
#!/bin/sh
# backup.sh

SOURCE_DIR="/data"
BACKUP_DIR="/backup"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

echo "Starting backup at $TIMESTAMP"

# Create timestamped backup
tar czf "$BACKUP_DIR/backup_$TIMESTAMP.tar.gz" -C "$SOURCE_DIR" .

# Keep only last 7 days
find "$BACKUP_DIR" -name "backup_*.tar.gz" -mtime +7 -delete

echo "Backup completed: backup_$TIMESTAMP.tar.gz"
```

### Website Monitor

Monitor websites and send alerts:

```python
# monitor.py
import requests
import time
import smtplib
from email.mime.text import MIMEText

websites = [
    'https://google.com',
    'https://github.com',
    'https://stackoverflow.com'
]

def check_website(url):
    try:
        response = requests.get(url, timeout=10)
        return response.status_code == 200
    except:
        return False

def send_alert(url):
    print(f"ALERT: {url} is down!")
    # Add email notification here

while True:
    for website in websites:
        if not check_website(website):
            send_alert(website)
        else:
            print(f"{website} is up")
    
    time.sleep(300)  # Check every 5 minutes
```

## Getting Started Tips

### Pick Your First Project

**For Beginners:** Start with the Personal Portfolio Website - it's simple but gives you a real result you can show others.

**For Developers:** Try the Full-Stack Development Environment - it demonstrates Docker's power for development workflows.

**For Fun:** Set up the Minecraft Server - friends will be impressed, and you'll learn container networking.

### Project Ideas to Expand

Once you complete a basic project, try these expansions:

- Add SSL certificates to web projects
- Implement user authentication
- Set up monitoring and logging
- Create automated deployment pipelines
- Scale services with Docker Compose
- Add database backups
- Implement health checks

### Sharing Your Projects

- Push your images to Docker Hub
- Share docker-compose files on GitHub
- Write blog posts about your learning experience
- Help others in Docker community forums

These projects demonstrate Docker's versatility while building practical skills you can apply in real development and deployment scenarios. Each project teaches different Docker concepts through hands-on experience rather than abstract theory.