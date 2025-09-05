# Production Docker Deployment

Moving from development to production requires careful attention to security, reliability, and maintainability. This guide covers essential practices for running Docker containers safely and effectively in production environments.

## Production vs Development: Key Differences

### Development Environment Characteristics
- Rapid iteration and debugging
- Full access and root privileges
- Verbose logging and error details
- Development dependencies included
- Hot reloading and file watching

### Production Environment Requirements
- Security hardening and minimal attack surface
- Non-root execution and principle of least privilege
- Structured logging and monitoring
- Minimal dependencies and optimized images
- Stability and predictable behavior

## Container Security Hardening

### Run as Non-Root User

**Problem:** Running containers as root creates unnecessary security risks.

**Solution:** Create and use dedicated user accounts.

```dockerfile
FROM node:18-alpine

# Create app user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S appuser -u 1001 -G nodejs

# Set up application
WORKDIR /app
COPY --chown=appuser:nodejs package*.json ./
RUN npm ci --only=production && npm cache clean --force

COPY --chown=appuser:nodejs . .

# Switch to non-root user
USER appuser

EXPOSE 3000
CMD ["node", "server.js"]
```

### Minimal Base Images

Use distroless or Alpine images to reduce attack surface:

```dockerfile
# Multi-stage build with minimal production image
FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Use distroless for minimal attack surface
FROM gcr.io/distroless/nodejs18-debian11
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/package.json ./package.json
CMD ["dist/server.js"]
```

### Image Vulnerability Scanning

Implement regular security scanning:

```bash
# Using Docker Scout
docker scout cves myapp:latest

# Using Trivy
trivy image myapp:latest

# Using Snyk (requires authentication)
snyk container test myapp:latest
```

### Secrets Management

**Never embed secrets in images:**

```dockerfile
# BAD: Secrets in environment variables
ENV API_KEY=secret123
ENV DB_PASSWORD=mypassword

# GOOD: Secrets injected at runtime
ENV API_KEY_FILE=/run/secrets/api_key
ENV DB_PASSWORD_FILE=/run/secrets/db_password
```

**Use Docker secrets or external secret management:**

```bash
# Docker Swarm secrets
docker service create \
  --secret api_key \
  --secret db_password \
  myapp:latest

# External secret injection
docker run \
  -e API_KEY="$(kubectl get secret api-key -o jsonpath='{.data.key}' | base64 -d)" \
  myapp:latest
```

## Health Checks and Monitoring

### Container Health Checks

Implement comprehensive health checking:

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN npm ci --only=production

# Add health check endpoint
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

EXPOSE 3000
CMD ["node", "server.js"]
```

**Application health check endpoint:**

```javascript
// health.js
app.get('/health', (req, res) => {
  const healthcheck = {
    uptime: process.uptime(),
    message: 'OK',
    timestamp: Date.now(),
    checks: {
      database: checkDatabase(),
      redis: checkRedis(),
      externalService: checkExternalService()
    }
  };
  
  try {
    res.status(200).send(healthcheck);
  } catch (error) {
    healthcheck.message = error;
    res.status(503).send(healthcheck);
  }
});
```

### Readiness and Liveness Probes

For Kubernetes deployments:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: myapp
        image: myapp:latest
        ports:
        - containerPort: 3000
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
```

## Logging Best Practices

### Structured Logging

Implement JSON-structured logging for better parsing:

```javascript
// Using winston for Node.js
const winston = require('winston');

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  transports: [
    new winston.transports.Console()
  ]
});

// Usage
logger.info('User authenticated', {
  userId: user.id,
  method: 'oauth',
  timestamp: new Date().toISOString()
});
```

### Log Collection Strategy

```dockerfile
FROM node:18-alpine
WORKDIR /app

# Install log collection agent (if needed)
RUN apk add --no-cache filebeat

COPY . .
RUN npm ci --only=production

# Configure logging
ENV LOG_LEVEL=info
ENV LOG_FORMAT=json

# Log to stdout for container log drivers
CMD ["node", "server.js"]
```

### Docker Logging Configuration

```bash
# Configure log driver
docker run -d \
  --log-driver=fluentd \
  --log-opt fluentd-address=localhost:24224 \
  --log-opt tag=myapp \
  myapp:latest

# Or use syslog
docker run -d \
  --log-driver=syslog \
  --log-opt syslog-address=tcp://log-server:514 \
  myapp:latest
```

## Resource Management

### Memory and CPU Limits

Set appropriate resource constraints:

```bash
docker run -d \
  --memory="512m" \
  --cpus="1.0" \
  --memory-swap="512m" \
  --oom-kill-disable=false \
  myapp:latest
```

**Docker Compose example:**

```yaml
version: '3.8'
services:
  app:
    image: myapp:latest
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M
    restart: unless-stopped
```

### Storage Considerations

```dockerfile
FROM node:18-alpine
WORKDIR /app

# Create non-root user
RUN adduser -D -s /bin/sh appuser

# Set up proper permissions
COPY --chown=appuser:appuser . .
RUN npm ci --only=production

# Use specific volume mounts for persistent data
VOLUME ["/app/data", "/app/logs"]

USER appuser
CMD ["node", "server.js"]
```

## Environment Configuration

### Configuration Management

Use environment-specific configurations:

```dockerfile
FROM node:18-alpine
WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

# Support multiple environments
ENV NODE_ENV=production
ENV CONFIG_FILE=/app/config/production.json

# Allow configuration override
CMD ["sh", "-c", "node -r dotenv/config server.js dotenv_config_path=/app/config/${NODE_ENV}.env"]
```

**Configuration file structure:**

```
/app/config/
├── development.env
├── staging.env
├── production.env
└── local.env
```

### Graceful Shutdown

Implement proper signal handling:

```javascript
// server.js
const express = require('express');
const app = express();
const server = app.listen(3000);

// Graceful shutdown
process.on('SIGTERM', shutdown);
process.on('SIGINT', shutdown);

function shutdown() {
  console.log('Received kill signal, shutting down gracefully');
  
  server.close(() => {
    console.log('Closed out remaining connections');
    
    // Close database connections, cleanup resources
    database.close(() => {
      console.log('Database connection closed');
      process.exit(0);
    });
  });

  // Force close after 10 seconds
  setTimeout(() => {
    console.error('Could not close connections in time, forcefully shutting down');
    process.exit(1);
  }, 10000);
}
```

## Deployment Strategies

### Rolling Updates

```yaml
# docker-compose.yml
version: '3.8'
services:
  app:
    image: myapp:${VERSION:-latest}
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
        failure_action: rollback
        order: start-first
      rollback_config:
        parallelism: 1
        delay: 10s
        failure_action: pause
        order: stop-first
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

### Blue-Green Deployment

```bash
#!/bin/bash
# deploy.sh

NEW_VERSION=$1
CURRENT_COLOR=$(docker ps --format "table {{.Names}}" | grep -E "(blue|green)" | head -1)

if [[ $CURRENT_COLOR == *"blue"* ]]; then
    NEW_COLOR="green"
    OLD_COLOR="blue"
else
    NEW_COLOR="blue" 
    OLD_COLOR="green"
fi

# Deploy new version
docker-compose -f docker-compose.${NEW_COLOR}.yml up -d

# Wait for health check
sleep 30

# Check health
if curl -f http://localhost:8080/health; then
    # Switch traffic
    docker-compose -f docker-compose.proxy.yml up -d --force-recreate
    
    # Remove old version
    docker-compose -f docker-compose.${OLD_COLOR}.yml down
    
    echo "Deployment successful: ${NEW_VERSION} on ${NEW_COLOR}"
else
    # Rollback
    docker-compose -f docker-compose.${NEW_COLOR}.yml down
    echo "Deployment failed, keeping ${OLD_COLOR}"
    exit 1
fi
```

## Monitoring and Observability

### Application Metrics

```javascript
// metrics.js
const promClient = require('prom-client');

// Create metrics
const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_ms',
  help: 'Duration of HTTP requests in ms',
  labelNames: ['method', 'route', 'status']
});

const httpRequestsTotal = new promClient.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status']
});

// Middleware
function metricsMiddleware(req, res, next) {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - start;
    const labels = {
      method: req.method,
      route: req.route?.path || req.path,
      status: res.statusCode
    };
    
    httpRequestDuration.observe(labels, duration);
    httpRequestsTotal.inc(labels);
  });
  
  next();
}

// Metrics endpoint
app.get('/metrics', (req, res) => {
  res.set('Content-Type', promClient.register.contentType);
  res.end(promClient.register.metrics());
});
```

### Container Monitoring

```dockerfile
FROM node:18-alpine

# Install monitoring agent
RUN apk add --no-cache curl

WORKDIR /app
COPY . .
RUN npm ci --only=production

# Expose metrics port
EXPOSE 3000 9090

# Add monitoring labels
LABEL monitoring.enabled="true"
LABEL monitoring.port="9090"
LABEL monitoring.path="/metrics"

CMD ["node", "server.js"]
```

## Backup and Disaster Recovery

### Data Backup Strategy

```bash
#!/bin/bash
# backup.sh

TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups"

# Database backup
docker exec postgres-container pg_dump -U user dbname > "${BACKUP_DIR}/db_${TIMESTAMP}.sql"

# Volume backup
docker run --rm \
  -v myapp_data:/data \
  -v ${BACKUP_DIR}:/backup \
  alpine tar czf /backup/volumes_${TIMESTAMP}.tar.gz /data

# Upload to cloud storage
aws s3 cp "${BACKUP_DIR}/db_${TIMESTAMP}.sql" s3://my-backups/
aws s3 cp "${BACKUP_DIR}/volumes_${TIMESTAMP}.tar.gz" s3://my-backups/

# Cleanup old backups (keep last 7 days)
find ${BACKUP_DIR} -name "*.sql" -mtime +7 -delete
find ${BACKUP_DIR} -name "*.tar.gz" -mtime +7 -delete
```

### Disaster Recovery

```yaml
# docker-compose.disaster-recovery.yml
version: '3.8'
services:
  app:
    image: myapp:latest
    environment:
      - DB_HOST=disaster-recovery-db
      - REDIS_HOST=disaster-recovery-redis
    deploy:
      replicas: 2
      placement:
        constraints:
          - node.labels.zone==disaster-recovery
    volumes:
      - disaster_recovery_data:/app/data

volumes:
  disaster_recovery_data:
    driver: local
    driver_opts:
      type: nfs
      o: addr=backup-server,rw
      device: ":/backup/volumes"
```

## Security Scanning and Compliance

### Automated Security Scanning

```yaml
# .github/workflows/security.yml
name: Security Scan
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Build image
      run: docker build -t myapp:latest .
    
    - name: Run Trivy vulnerability scanner
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: 'myapp:latest'
        format: 'sarif'
        output: 'trivy-results.sarif'
    
    - name: Upload Trivy scan results
      uses: github/codeql-action/upload-sarif@v2
      with:
        sarif_file: 'trivy-results.sarif'
```

### Runtime Security

```bash
# Run with security options
docker run -d \
  --security-opt=no-new-privileges:true \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  --read-only \
  --tmpfs /tmp \
  --user 1001:1001 \
  myapp:latest
```

Production Docker deployment requires careful attention to security, monitoring, and reliability. These practices ensure your containers run safely and efficiently in production environments while maintaining the agility and consistency that make containers valuable.

## Production Checklist

Before deploying to production, verify:

- [ ] Non-root user configured
- [ ] Minimal base image used
- [ ] Secrets externalized
- [ ] Health checks implemented
- [ ] Resource limits set
- [ ] Logging configured
- [ ] Monitoring enabled
- [ ] Backup strategy defined
- [ ] Security scanning automated
- [ ] Graceful shutdown implemented