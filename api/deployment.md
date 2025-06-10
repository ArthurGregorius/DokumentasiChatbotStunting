# API Deployment

## Overview

Dokumentasi ini menjelaskan proses dan praktik deployment untuk API chatbot, termasuk environment setup, deployment process, monitoring, scaling, dan maintenance.

## Deployment Environment

### 1. Environment Variables

```javascript
// .env.production
NODE_ENV=production
PORT=3000

# Database
MONGODB_URI=mongodb://user:pass@host:port/db
REDIS_URI=redis://user:pass@host:port

# JWT
JWT_SECRET=your-secret-key
JWT_EXPIRES_IN=7d

# WhatsApp
WHATSAPP_SESSION_PATH=/path/to/session
WHATSAPP_QR_PATH=/path/to/qr

# Google Sheets
GOOGLE_SHEETS_ID=your-sheet-id
GOOGLE_SERVICE_ACCOUNT=path/to/service-account.json

# Email
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password

# Monitoring
SENTRY_DSN=your-sentry-dsn
NEW_RELIC_LICENSE_KEY=your-license-key
```

### 2. Server Configuration

```javascript
// config/server.js
module.exports = {
  port: process.env.PORT || 3000,
  host: process.env.HOST || '0.0.0.0',
  
  // SSL/TLS
  ssl: {
    enabled: true,
    cert: '/path/to/cert.pem',
    key: '/path/to/key.pem'
  },
  
  // CORS
  cors: {
    origin: ['https://your-domain.com'],
    methods: ['GET', 'POST', 'PUT', 'DELETE'],
    allowedHeaders: ['Content-Type', 'Authorization']
  },
  
  // Rate Limiting
  rateLimit: {
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100 // limit each IP to 100 requests per windowMs
  }
};
```

## Deployment Process

### 1. Build Process

```javascript
// package.json
{
  "scripts": {
    "build": "npm run clean && npm run compile",
    "clean": "rimraf dist",
    "compile": "babel src -d dist",
    "start": "node dist/index.js",
    "start:prod": "NODE_ENV=production node dist/index.js"
  }
}
```

### 2. Docker Configuration

```dockerfile
# Dockerfile
FROM node:16-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

EXPOSE 3000

CMD ["npm", "run", "start:prod"]
```

### 3. Docker Compose

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
    depends_on:
      - mongodb
      - redis
    volumes:
      - ./logs:/app/logs
      - ./whatsapp-sessions:/app/whatsapp-sessions
    restart: unless-stopped

  mongodb:
    image: mongo:4.4
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db
    restart: unless-stopped

  redis:
    image: redis:6-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    restart: unless-stopped

volumes:
  mongodb_data:
  redis_data:
```

## Monitoring

### 1. Application Monitoring

```javascript
// config/monitoring.js
const Sentry = require('@sentry/node');
const NewRelic = require('newrelic');

module.exports = {
  init: () => {
    // Sentry
    Sentry.init({
      dsn: process.env.SENTRY_DSN,
      environment: process.env.NODE_ENV,
      tracesSampleRate: 1.0
    });
    
    // New Relic
    NewRelic.setLicenseKey(process.env.NEW_RELIC_LICENSE_KEY);
  },
  
  // Error tracking
  captureException: (error) => {
    Sentry.captureException(error);
  },
  
  // Performance monitoring
  startTransaction: (name) => {
    return NewRelic.startWebTransaction(name);
  }
};
```

### 2. Health Checks

```javascript
// routes/health.js
router.get('/health', async (req, res) => {
  try {
    // Check database connection
    await mongoose.connection.db.admin().ping();
    
    // Check Redis connection
    await redis.ping();
    
    // Check WhatsApp connection
    const whatsappStatus = whatsappService.getStatus();
    
    res.json({
      status: 'healthy',
      timestamp: new Date(),
      services: {
        database: 'connected',
        redis: 'connected',
        whatsapp: whatsappStatus
      }
    });
  } catch (error) {
    res.status(503).json({
      status: 'unhealthy',
      timestamp: new Date(),
      error: error.message
    });
  }
});
```

## Scaling

### 1. Horizontal Scaling

```javascript
// config/cluster.js
const cluster = require('cluster');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  console.log(`Master ${process.pid} is running`);
  
  // Fork workers
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
  
  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died`);
    cluster.fork(); // Replace dead worker
  });
} else {
  // Workers can share any TCP connection
  require('./index.js');
  console.log(`Worker ${process.pid} started`);
}
```

### 2. Load Balancer Configuration

```nginx
# nginx.conf
upstream api_servers {
  server 127.0.0.1:3001;
  server 127.0.0.1:3002;
  server 127.0.0.1:3003;
  server 127.0.0.1:3004;
}

server {
  listen 80;
  server_name api.your-domain.com;
  
  location / {
    proxy_pass http://api_servers;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }
}
```

## Backup & Recovery

### 1. Database Backup

```javascript
// scripts/backup.js
const { exec } = require('child_process');
const path = require('path');

const backupMongoDB = () => {
  const timestamp = new Date().toISOString().replace(/[:.]/g, '-');
  const backupPath = path.join(__dirname, '../backups', `mongodb-${timestamp}`);
  
  const command = `mongodump --uri="${process.env.MONGODB_URI}" --out="${backupPath}"`;
  
  exec(command, (error, stdout, stderr) => {
    if (error) {
      console.error(`Backup failed: ${error}`);
      return;
    }
    console.log(`Backup completed: ${backupPath}`);
  });
};

// Schedule daily backup
setInterval(backupMongoDB, 24 * 60 * 60 * 1000);
```

### 2. Recovery Process

```javascript
// scripts/recovery.js
const { exec } = require('child_process');
const path = require('path');

const recoverMongoDB = (backupPath) => {
  const command = `mongorestore --uri="${process.env.MONGODB_URI}" --dir="${backupPath}"`;
  
  exec(command, (error, stdout, stderr) => {
    if (error) {
      console.error(`Recovery failed: ${error}`);
      return;
    }
    console.log('Recovery completed successfully');
  });
};
```

## Security

### 1. SSL/TLS Configuration

```javascript
// config/ssl.js
const fs = require('fs');
const path = require('path');

module.exports = {
  key: fs.readFileSync(path.join(__dirname, '../ssl/private.key')),
  cert: fs.readFileSync(path.join(__dirname, '../ssl/certificate.crt')),
  ca: fs.readFileSync(path.join(__dirname, '../ssl/ca_bundle.crt'))
};
```

### 2. Security Headers

```javascript
// middleware/security.js
const helmet = require('helmet');

module.exports = helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", 'data:', 'https:'],
      connectSrc: ["'self'", 'https://api.your-domain.com']
    }
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true
  },
  noSniff: true,
  xssFilter: true,
  frameguard: {
    action: 'deny'
  }
});
```

## Best Practices

1. **Environment Setup**
   - Use environment variables
   - Secure sensitive data
   - Configure logging
   - Set up monitoring

2. **Deployment Process**
   - Automated builds
   - Version control
   - CI/CD pipeline
   - Rollback strategy

3. **Monitoring**
   - Health checks
   - Error tracking
   - Performance metrics
   - Alert system

4. **Scaling**
   - Load balancing
   - Horizontal scaling
   - Resource optimization
   - Cache strategy

5. **Security**
   - SSL/TLS
   - Security headers
   - Rate limiting
   - Access control

6. **Backup & Recovery**
   - Regular backups
   - Data redundancy
   - Recovery testing
   - Disaster plan

7. **Maintenance**
   - Regular updates
   - Security patches
   - Performance tuning
   - Log rotation 