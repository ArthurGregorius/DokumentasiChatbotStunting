# 🚀 Deployment: Production Guidelines

<div align="center">

![Deployment](https://img.shields.io/badge/Deployment-Production-blue)
![Status](https://img.shields.io/badge/Status-Active-green)

</div>

## 📋 Overview

Dokumentasi ini menjelaskan panduan deployment ke production untuk Chatbot Identifikasi Stunting, termasuk best practices, monitoring, dan maintenance.

## 🛠️ Production Setup

### 1. Server Requirements
```bash
# Minimum Requirements
CPU: 2 cores
RAM: 4GB
Storage: 50GB SSD
OS: Ubuntu 20.04 LTS

# Recommended Requirements
CPU: 4 cores
RAM: 8GB
Storage: 100GB SSD
OS: Ubuntu 20.04 LTS
```

### 2. System Dependencies
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install dependencies
sudo apt install -y \
    curl \
    git \
    nginx \
    certbot \
    python3-certbot-nginx \
    docker.io \
    docker-compose
```

### 3. Application Setup
```bash
# Clone repository
git clone https://github.com/your-org/chatbot-stunting.git
cd chatbot-stunting

# Create production environment file
cp .env.example .env.production

# Build and start containers
docker-compose -f docker-compose.prod.yml up -d
```

## 🔒 Security Measures

### 1. Firewall Configuration
```bash
# Enable UFW
sudo ufw enable

# Allow SSH
sudo ufw allow ssh

# Allow HTTP/HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Allow application port
sudo ufw allow 3000/tcp
```

### 2. SSL Certificate
```bash
# Install SSL certificate
sudo certbot --nginx -d api.chatbot-stunting.com

# Auto-renewal
sudo certbot renew --dry-run
```

### 3. Database Security
```bash
# Enable authentication
mongod --auth

# Create admin user
mongo admin
db.createUser({
  user: "admin",
  pwd: "secure-password",
  roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
})
```

## 📊 Monitoring

### 1. Application Monitoring
```javascript
// monitoring.js
const prometheus = require('prom-client');
const collectDefaultMetrics = prometheus.collectDefaultMetrics;

// Collect default metrics
collectDefaultMetrics();

// Custom metrics
const httpRequestDuration = new prometheus.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code']
});

// Export metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', prometheus.register.contentType);
  res.end(await prometheus.register.metrics());
});
```

### 2. Log Management
```javascript
// logger.js
const winston = require('winston');
const { ElasticsearchTransport } = require('winston-elasticsearch');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' }),
    new ElasticsearchTransport({
      level: 'info',
      index: 'chatbot-stunting-logs',
      clientOpts: { node: 'http://localhost:9200' }
    })
  ]
});
```

## 🔄 Backup Strategy

### 1. Database Backup
```bash
#!/bin/bash
# backup.sh

# Set variables
BACKUP_DIR="/backup/mongodb"
DATE=$(date +%Y%m%d_%H%M%S)
DB_NAME="chatbot-stunting"

# Create backup
mongodump --db $DB_NAME --out $BACKUP_DIR/$DATE

# Compress backup
tar -czf $BACKUP_DIR/$DATE.tar.gz $BACKUP_DIR/$DATE

# Remove uncompressed backup
rm -rf $BACKUP_DIR/$DATE

# Keep only last 7 days of backups
find $BACKUP_DIR -type f -mtime +7 -delete
```

### 2. Application Backup
```bash
#!/bin/bash
# app-backup.sh

# Set variables
BACKUP_DIR="/backup/app"
DATE=$(date +%Y%m%d_%H%M%S)
APP_DIR="/app/chatbot-stunting"

# Create backup
tar -czf $BACKUP_DIR/app_$DATE.tar.gz $APP_DIR

# Keep only last 7 days of backups
find $BACKUP_DIR -type f -mtime +7 -delete
```

## 🔧 Maintenance

### 1. Update Procedure
```bash
#!/bin/bash
# update.sh

# Pull latest changes
git pull origin main

# Build new image
docker-compose -f docker-compose.prod.yml build

# Stop current containers
docker-compose -f docker-compose.prod.yml down

# Start new containers
docker-compose -f docker-compose.prod.yml up -d

# Remove old images
docker image prune -f
```

### 2. Health Check
```bash
#!/bin/bash
# health-check.sh

# Check application health
response=$(curl -s http://localhost:3000/health)
if [[ $response == *"UP"* ]]; then
    echo "Application is healthy"
else
    echo "Application is unhealthy"
    # Send notification
    curl -X POST https://api.notification-service.com/alert \
        -H "Content-Type: application/json" \
        -d '{"message": "Application is unhealthy"}'
fi
```

## 📈 Scaling

### 1. Horizontal Scaling
```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  app:
    image: chatbot-stunting:latest
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
    networks:
      - app-network

  nginx:
    image: nginx:latest
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - app
    networks:
      - app-network
```

### 2. Load Balancer Configuration
```nginx
# nginx.conf
upstream app_servers {
    server app:3000;
    server app:3000;
    server app:3000;
}

server {
    listen 80;
    server_name api.chatbot-stunting.com;

    location / {
        proxy_pass http://app_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

## 📚 Dokumentasi Terkait

- [Docker Setup](docker.md)
- [Environment Configuration](environment.md)
- [Local Setup](../development/local-setup.md)

---

<div align="center">

### 🔗 Navigasi

[⬅️ Kembali ke Environment Configuration](environment.md) | [Lanjut ke Local Setup ➡️](../development/local-setup.md)

</div> 