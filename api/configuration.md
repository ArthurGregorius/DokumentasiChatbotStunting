# ⚙️ API Configuration

<div align="center">

![API Configuration](https://img.shields.io/badge/API-Configuration-blue)
![Status](https://img.shields.io/badge/Status-Active-green)

</div>

## 📋 Overview

Dokumentasi ini menjelaskan konfigurasi API untuk Chatbot Identifikasi Stunting, termasuk pengaturan server, middleware, dan integrasi dengan layanan eksternal.

## 🔧 Server Configuration

### Basic Configuration
```javascript
// config/server.js
module.exports = {
  port: process.env.PORT || 3000,
  env: process.env.NODE_ENV || 'development',
  cors: {
    origin: process.env.CORS_ORIGIN || '*',
    methods: ['GET', 'POST', 'PUT', 'DELETE'],
    allowedHeaders: ['Content-Type', 'Authorization']
  }
};
```

### Express Configuration
```javascript
// app.js
const express = require('express');
const app = express();

app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(cors(config.cors));
```

## 🔒 Security Configuration

### JWT Configuration
```javascript
// config/jwt.js
module.exports = {
  secret: process.env.JWT_SECRET,
  expiresIn: '24h',
  algorithm: 'HS256'
};
```

### Rate Limiting
```javascript
// config/rateLimit.js
module.exports = {
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP'
};
```

## 📡 WhatsApp Integration

### WhatsApp API Configuration
```javascript
// config/whatsapp.js
module.exports = {
  apiKey: process.env.WHATSAPP_API_KEY,
  phoneNumber: process.env.WHATSAPP_PHONE_NUMBER,
  webhookUrl: process.env.WHATSAPP_WEBHOOK_URL,
  version: 'v1.0'
};
```

### Webhook Configuration
```javascript
// config/webhook.js
module.exports = {
  verifyToken: process.env.WEBHOOK_VERIFY_TOKEN,
  path: '/webhook/whatsapp',
  methods: ['POST']
};
```

## 📊 Database Configuration

### PostgreSQL Configuration
```javascript
// config/database.js
module.exports = {
  client: 'pg',
  connection: {
    host: process.env.DB_HOST,
    port: process.env.DB_PORT,
    user: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_NAME
  },
  pool: {
    min: 2,
    max: 10
  }
};
```

### Redis Configuration
```javascript
// config/redis.js
module.exports = {
  host: process.env.REDIS_HOST,
  port: process.env.REDIS_PORT,
  password: process.env.REDIS_PASSWORD,
  db: process.env.REDIS_DB
};
```

## 📝 Logging Configuration

### Winston Logger Configuration
```javascript
// config/logger.js
module.exports = {
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
};
```

## 🔄 Cache Configuration

### Redis Cache Configuration
```javascript
// config/cache.js
module.exports = {
  ttl: 3600, // 1 hour
  prefix: 'chatbot:',
  maxSize: 1000
};
```

## 📱 Google Apps Script Configuration

### Google Apps Script Settings
```javascript
// config/gscript.js
module.exports = {
  deploymentId: process.env.GSCRIPT_DEPLOYMENT_ID,
  credentials: process.env.GOOGLE_CREDENTIALS,
  spreadsheetId: process.env.SPREADSHEET_ID
};
```

## 📚 Dokumentasi Terkait

- [API Overview](README.md)
- [Controllers](controllers.md)
- [Models](models.md)
- [Services](services.md)

---

<div align="center">

### 🔗 Navigasi

[⬅️ Kembali ke API Overview](README.md) | [Lanjut ke Controllers ➡️](controllers.md)

</div> 