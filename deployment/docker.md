# 🐳 Deployment: Docker Setup

<div align="center">

![Deployment](https://img.shields.io/badge/Deployment-Docker-blue)
![Status](https://img.shields.io/badge/Status-Active-green)

</div>

## 📋 Overview

Dokumentasi ini menjelaskan konfigurasi dan penggunaan Docker untuk deployment Chatbot Identifikasi Stunting, termasuk container setup, networking, dan best practices.

## 🏗️ Docker Configuration

### 1. Dockerfile
```dockerfile
# Base image
FROM node:14-alpine

# Set working directory
WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci --only=production

# Copy application code
COPY . .

# Build application
RUN npm run build

# Expose port
EXPOSE 3000

# Start application
CMD ["npm", "start"]
```

### 2. Docker Compose
```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - MONGODB_URI=mongodb://mongodb:27017/chatbot-stunting
      - REDIS_URL=redis://redis:6379
    depends_on:
      - mongodb
      - redis
    networks:
      - app-network

  mongodb:
    image: mongo:4.4
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db
    networks:
      - app-network

  redis:
    image: redis:6
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  mongodb_data:
  redis_data:
```

## 🚀 Deployment Steps

### 1. Build Images
```bash
# Build all services
docker-compose build

# Build specific service
docker-compose build app
```

### 2. Start Services
```bash
# Start all services
docker-compose up -d

# Start specific service
docker-compose up -d app
```

### 3. View Logs
```bash
# View all logs
docker-compose logs -f

# View specific service logs
docker-compose logs -f app
```

### 4. Stop Services
```bash
# Stop all services
docker-compose down

# Stop and remove volumes
docker-compose down -v
```

## 🔧 Development Setup

### 1. Development Dockerfile
```dockerfile
# Dockerfile.dev
FROM node:14-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "run", "dev"]
```

### 2. Development Docker Compose
```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - MONGODB_URI=mongodb://mongodb:27017/chatbot-stunting
      - REDIS_URL=redis://redis:6379
    depends_on:
      - mongodb
      - redis
    networks:
      - app-network

  mongodb:
    image: mongo:4.4
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db
    networks:
      - app-network

  redis:
    image: redis:6
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  mongodb_data:
  redis_data:
```

### 3. Run Development Environment
```bash
docker-compose -f docker-compose.dev.yml up -d
```

## 🔍 Monitoring

### 1. Container Health Checks
```yaml
# docker-compose.yml
services:
  app:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

### 2. Resource Limits
```yaml
# docker-compose.yml
services:
  app:
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 1G
        reservations:
          cpus: '0.5'
          memory: 512M
```

## 🔐 Security

### 1. Non-Root User
```dockerfile
# Dockerfile
FROM node:14-alpine

# Create non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

# Set ownership
RUN chown -R appuser:appgroup /app

# Switch to non-root user
USER appuser

EXPOSE 3000

CMD ["npm", "start"]
```

### 2. Security Scanning
```bash
# Install Trivy
brew install aquasecurity/trivy/trivy

# Scan image
trivy image chatbot-stunting:latest
```

## 📦 Multi-Stage Builds

### 1. Production Build
```dockerfile
# Build stage
FROM node:14-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# Production stage
FROM node:14-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY --from=builder /app/dist ./dist

EXPOSE 3000

CMD ["npm", "start"]
```

## 🔄 CI/CD Integration

### 1. GitHub Actions
```yaml
# .github/workflows/docker.yml
name: Docker

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v2
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1
      
      - name: Login to DockerHub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      
      - name: Build and push
        uses: docker/build-push-action@v2
        with:
          context: .
          push: true
          tags: your-org/chatbot-stunting:latest
```

## 📚 Dokumentasi Terkait

- [Environment Configuration](environment.md)
- [Production Guidelines](production.md)
- [Local Setup](../development/local-setup.md)

---

<div align="center">

### 🔗 Navigasi

[⬅️ Kembali ke Code Style](../development/code-style.md) | [Lanjut ke Environment Configuration ➡️](environment.md)

</div> 