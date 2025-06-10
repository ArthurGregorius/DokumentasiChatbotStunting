# Configuration

## 📋 Overview

Panduan ini menjelaskan cara mengkonfigurasi Chatbot Identifikasi Stunting untuk berbagai lingkungan (development, staging, production).

## 🔧 Environment Variables

### Server Configuration

```env
# Server
PORT=3000
NODE_ENV=development
LOG_LEVEL=info

# WhatsApp
WHATSAPP_API_KEY=your_api_key
WHATSAPP_PHONE_NUMBER=your_phone_number
WHATSAPP_WEBHOOK_URL=https://your-domain.com/webhook

# Google Apps Script
DEPLOYMENT_ID=your_deployment_id
GOOGLE_CREDENTIALS=your_credentials

# Database
DB_HOST=localhost
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=your_password
DB_NAME=chatbot_stunting
```

## 📱 WhatsApp Configuration

### 1. Setup WhatsApp Business API

1. Daftar di [WhatsApp Business API](https://business.whatsapp.com)
2. Verifikasi nomor telepon
3. Dapatkan API key
4. Konfigurasi webhook

### 2. Webhook Configuration

```javascript
// webhook.js
app.post('/webhook', (req, res) => {
  const { body } = req;
  
  // Verify webhook
  if (body.type === 'verification') {
    return res.json({ status: 'verified' });
  }
  
  // Handle messages
  handleMessage(body);
  
  res.json({ status: 'success' });
});
```

## 📊 Google Apps Script Setup

### 1. Create Project

1. Buka [Google Apps Script](https://script.google.com)
2. Buat project baru
3. Copy script dari `AppScript.gs`
4. Deploy sebagai web app

### 2. Configure Permissions

```javascript
// Set permissions
function doGet(e) {
  return handleRequest(e);
}

function doPost(e) {
  return handleRequest(e);
}
```

## 🔒 Security Configuration

### 1. API Authentication

```javascript
// middleware/auth.js
const authMiddleware = (req, res, next) => {
  const apiKey = req.headers['x-api-key'];
  
  if (!apiKey || apiKey !== process.env.API_KEY) {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  
  next();
};
```

### 2. Rate Limiting

```javascript
// middleware/rateLimit.js
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // limit each IP to 100 requests per windowMs
});

app.use(limiter);
```

## 📝 Logging Configuration

### 1. Winston Logger

```javascript
// utils/logger.js
const winston = require('winston');

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});
```

### 2. Error Handling

```javascript
// middleware/errorHandler.js
const errorHandler = (err, req, res, next) => {
  logger.error(err.stack);
  
  res.status(err.status || 500).json({
    error: {
      message: err.message,
      code: err.code
    }
  });
};
```

## 🔄 Database Configuration

### 1. Connection Setup

```javascript
// config/database.js
const { Pool } = require('pg');

const pool = new Pool({
  host: process.env.DB_HOST,
  port: process.env.DB_PORT,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME
});
```

### 2. Migration Setup

```javascript
// migrations/001_initial.js
exports.up = function(knex) {
  return knex.schema
    .createTable('users', table => {
      table.increments('id');
      table.string('name');
      table.string('phone').unique();
      table.timestamps();
    });
};
```
