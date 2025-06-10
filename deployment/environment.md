# Environment Configuration

## 📋 Overview

Dokumentasi ini menjelaskan konfigurasi environment untuk deployment Chatbot Identifikasi Stunting, termasuk variabel environment, konfigurasi server, dan best practices.

## 🔧 Environment Variables

### 1. Server Configuration

```env
# Server
PORT=3000
NODE_ENV=production
API_VERSION=v1
BASE_URL=https://api.chatbot-stunting.com

# Security
JWT_SECRET=your-jwt-secret
JWT_EXPIRES_IN=24h
RATE_LIMIT_WINDOW=15m
RATE_LIMIT_MAX=100
```

### 2. Database Configuration

```env
# MongoDB
MONGODB_URI=mongodb://localhost:27017/chatbot-stunting
MONGODB_USER=admin
MONGODB_PASSWORD=secure-password

# Redis
REDIS_URL=redis://localhost:6379
REDIS_PASSWORD=secure-password
```

### 3. WhatsApp Integration

```env
# WhatsApp Business API
WHATSAPP_API_URL=https://graph.facebook.com/v13.0
WHATSAPP_PHONE_NUMBER_ID=your-phone-number-id
WHATSAPP_ACCESS_TOKEN=your-access-token
WHATSAPP_VERIFY_TOKEN=your-verify-token
```

### 4. Google Apps Script

```env
# Google Apps Script
GOOGLE_APPS_SCRIPT_URL=your-script-url
GOOGLE_APPS_SCRIPT_KEY=your-script-key
```

### 5. Logging

```env
# Winston Logger
LOG_LEVEL=info
LOG_FILE_PATH=logs/app.log
```

## 🛠️ Environment Setup

### 1. Development Environment

```bash
# Create .env file
cp .env.example .env

# Install dependencies
npm install

# Start development server
npm run dev
```

### 2. Production Environment

```bash
# Create production .env file
cp .env.example .env.production

# Build application
npm run build

# Start production server
npm start
```

## 🔐 Security Configuration

### 1. SSL/TLS Setup

```nginx
# nginx.conf
server {
    listen 443 ssl;
    server_name api.chatbot-stunting.com;

    ssl_certificate /etc/letsencrypt/live/api.chatbot-stunting.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.chatbot-stunting.com/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
}
```

### 2. CORS Configuration

```javascript
// cors.config.js
module.exports = {
  origin: ['https://chatbot-stunting.com'],
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true,
  maxAge: 86400
};
```

## 📊 Monitoring Configuration

### 1. Winston Logger Setup

```javascript
// logger.config.js
const winston = require('winston');

module.exports = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.File({
      filename: process.env.LOG_FILE_PATH || 'logs/app.log'
    }),
    new winston.transports.Console({
      format: winston.format.simple()
    })
  ]
});
```

### 2. Health Check Endpoint

```javascript
// health.js
app.get('/health', (req, res) => {
  const health = {
    status: 'UP',
    timestamp: new Date(),
    uptime: process.uptime(),
    memory: process.memoryUsage()
  };
  res.json(health);
});
```

## 🔄 CI/CD Environment

### 1. GitHub Actions Secrets

```yaml
# Required secrets
DOCKERHUB_USERNAME: your-dockerhub-username
DOCKERHUB_TOKEN: your-dockerhub-token
AWS_ACCESS_KEY_ID: your-aws-access-key
AWS_SECRET_ACCESS_KEY: your-aws-secret-key
```

### 2. Deployment Environment

```yaml
# deployment.env
NODE_ENV=production
PORT=3000
MONGODB_URI=mongodb://mongodb:27017/chatbot-stunting
REDIS_URL=redis://redis:6379
```
