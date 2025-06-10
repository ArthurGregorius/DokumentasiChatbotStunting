# 🛠️ Implementation: Utilities

<div align="center">

![Implementation](https://img.shields.io/badge/Implementation-Utilities-blue)
![Status](https://img.shields.io/badge/Status-Active-green)

</div>

## 📋 Overview

Dokumentasi ini menjelaskan utility functions dan helpers yang digunakan dalam Chatbot Identifikasi Stunting, termasuk validasi, formatting, dan helper functions.

## 🔍 Validation Utils

### 1. Input Validation
```typescript
// src/utils/validation.ts
import { z } from 'zod';

export const phoneRegex = /^\+?[1-9]\d{1,14}$/;

export const userSchema = z.object({
  name: z.string().min(2).max(100),
  phone: z.string().regex(phoneRegex),
  address: z.string().min(5).max(200),
  birthDate: z.date()
});

export const childSchema = z.object({
  name: z.string().min(2).max(100),
  birthDate: z.date(),
  gender: z.enum(['male', 'female']),
  height: z.number().positive(),
  weight: z.number().positive(),
  headCircumference: z.number().positive()
});

export const validateInput = <T>(schema: z.Schema<T>, data: unknown): T => {
  try {
    return schema.parse(data);
  } catch (error) {
    if (error instanceof z.ZodError) {
      throw new ValidationError(error.errors);
    }
    throw error;
  }
};
```

### 2. Error Handling
```typescript
// src/utils/errors.ts
export class ValidationError extends Error {
  constructor(public errors: Array<{ path: string; message: string }>) {
    super('Validation failed');
    this.name = 'ValidationError';
  }
}

export class NotFoundError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'NotFoundError';
  }
}

export class ConflictError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'ConflictError';
  }
}

export const handleError = (error: Error) => {
  if (error instanceof ValidationError) {
    return {
      status: 400,
      message: 'Validation failed',
      errors: error.errors
    };
  }

  if (error instanceof NotFoundError) {
    return {
      status: 404,
      message: error.message
    };
  }

  if (error instanceof ConflictError) {
    return {
      status: 409,
      message: error.message
    };
  }

  return {
    status: 500,
    message: 'Internal server error'
  };
};
```

## 📊 Formatting Utils

### 1. Date Formatting
```typescript
// src/utils/date.ts
import { format, parseISO } from 'date-fns';
import { id } from 'date-fns/locale';

export const formatDate = (date: Date | string, formatStr: string = 'dd MMMM yyyy'): string => {
  const parsedDate = typeof date === 'string' ? parseISO(date) : date;
  return format(parsedDate, formatStr, { locale: id });
};

export const formatDateTime = (date: Date | string): string => {
  return formatDate(date, 'dd MMMM yyyy HH:mm');
};

export const formatAge = (birthDate: Date | string): string => {
  const parsedDate = typeof birthDate === 'string' ? parseISO(birthDate) : birthDate;
  const today = new Date();
  const age = today.getFullYear() - parsedDate.getFullYear();
  const monthDiff = today.getMonth() - parsedDate.getMonth();
  
  if (monthDiff < 0 || (monthDiff === 0 && today.getDate() < parsedDate.getDate())) {
    return `${age - 1} tahun ${12 + monthDiff} bulan`;
  }
  
  return `${age} tahun ${monthDiff} bulan`;
};
```

### 2. Number Formatting
```typescript
// src/utils/number.ts
export const formatNumber = (num: number): string => {
  return new Intl.NumberFormat('id-ID').format(num);
};

export const formatCurrency = (amount: number): string => {
  return new Intl.NumberFormat('id-ID', {
    style: 'currency',
    currency: 'IDR'
  }).format(amount);
};

export const formatPercentage = (value: number): string => {
  return new Intl.NumberFormat('id-ID', {
    style: 'percent',
    minimumFractionDigits: 1,
    maximumFractionDigits: 1
  }).format(value / 100);
};
```

## 🔄 Helper Functions

### 1. WhatsApp Message Helpers
```typescript
// src/utils/whatsapp.ts
export const formatWhatsAppMessage = (template: string, params: Record<string, string>): string => {
  return template.replace(/\{(\w+)\}/g, (match, key) => params[key] || match);
};

export const parseWhatsAppMessage = (message: string): {
  type: 'text' | 'image' | 'document';
  content: string;
  metadata?: Record<string, string>;
} => {
  // Implementation
};

export const validateWhatsAppNumber = (phone: string): boolean => {
  return /^\+?[1-9]\d{1,14}$/.test(phone);
};
```

### 2. Cache Helpers
```typescript
// src/utils/cache.ts
import { Redis } from 'ioredis';

export class CacheManager {
  constructor(private readonly redis: Redis) {}

  async get<T>(key: string): Promise<T | null> {
    const data = await this.redis.get(key);
    return data ? JSON.parse(data) : null;
  }

  async set(key: string, value: any, ttl?: number): Promise<void> {
    const stringValue = JSON.stringify(value);
    if (ttl) {
      await this.redis.setex(key, ttl, stringValue);
    } else {
      await this.redis.set(key, stringValue);
    }
  }

  async delete(key: string): Promise<void> {
    await this.redis.del(key);
  }

  async clear(): Promise<void> {
    await this.redis.flushall();
  }
}
```

## 📝 Logger Utils

### 1. Winston Logger
```typescript
// src/utils/logger.ts
import winston from 'winston';

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' }),
    new winston.transports.Console({
      format: winston.format.simple()
    })
  ]
});

export const logError = (error: Error, context?: string): void => {
  logger.error({
    message: error.message,
    stack: error.stack,
    context
  });
};

export const logInfo = (message: string, data?: any): void => {
  logger.info({
    message,
    data
  });
};

export default logger;
```

### 2. Request Logger
```typescript
// src/utils/request-logger.ts
import { Request, Response, NextFunction } from 'express';
import logger from './logger';

export const requestLogger = (req: Request, res: Response, next: NextFunction): void => {
  const start = Date.now();

  res.on('finish', () => {
    const duration = Date.now() - start;
    logger.info({
      method: req.method,
      url: req.url,
      status: res.statusCode,
      duration: `${duration}ms`,
      ip: req.ip,
      userAgent: req.get('user-agent')
    });
  });

  next();
};
```

## 📚 Dokumentasi Terkait

- [Controllers](controllers.md)
- [Services](services.md)
- [Models](models.md)

---

<div align="center">

### 🔗 Navigasi

### User Input Validator
```javascript
// utils/validators.js
const validateUserInput = (data) => {
  const errors = [];
  
  // Validate name
  if (!data.name || data.name.length < 3) {
    errors.push('Name must be at least 3 characters long');
  }
  
  // Validate phone
  if (!data.phone || !data.phone.match(/^62\d{9,12}$/)) {
    errors.push('Please enter a valid Indonesian phone number');
  }
  
  // Validate address
  if (!data.address || data.address.length < 10) {
    errors.push('Address must be at least 10 characters long');
  }
  
  // Validate birth date
  if (!data.birthDate || new Date(data.birthDate) > new Date()) {
    errors.push('Birth date cannot be in the future');
  }
  
  return {
    isValid: errors.length === 0,
    errors
  };
};
```

### Child Input Validator
```javascript
const validateChildInput = (data) => {
  const errors = [];
  
  // Validate name
  if (!data.name || data.name.length < 3) {
    errors.push('Name must be at least 3 characters long');
  }
  
  // Validate birth date
  if (!data.birthDate || new Date(data.birthDate) > new Date()) {
    errors.push('Birth date cannot be in the future');
  }
  
  // Validate gender
  if (!data.gender || !['male', 'female'].includes(data.gender)) {
    errors.push('Gender must be either male or female');
  }
  
  // Validate measurements
  if (!data.height || data.height < 30 || data.height > 200) {
    errors.push('Height must be between 30 and 200 cm');
  }
  
  if (!data.weight || data.weight < 1 || data.weight > 100) {
    errors.push('Weight must be between 1 and 100 kg');
  }
  
  if (!data.headCircumference || data.headCircumference < 20 || data.headCircumference > 60) {
    errors.push('Head circumference must be between 20 and 60 cm');
  }
  
  return {
    isValid: errors.length === 0,
    errors
  };
};
```

### Questionnaire Input Validator
```javascript
const validateQuestionnaireInput = (data) => {
  const errors = [];
  
  // Validate type
  if (!data.type || !['initial', 'followup', 'monitoring'].includes(data.type)) {
    errors.push('Invalid questionnaire type');
  }
  
  // Validate answers
  if (!data.answers || !Array.isArray(data.answers) || data.answers.length === 0) {
    errors.push('At least one answer is required');
  } else {
    data.answers.forEach((answer, index) => {
      if (!answer.questionId) {
        errors.push(`Question ID is required for answer ${index + 1}`);
      }
      if (!answer.answer) {
        errors.push(`Answer is required for question ${index + 1}`);
      }
    });
  }
  
  return {
    isValid: errors.length === 0,
    errors
  };
};
```

## ⚠️ Error Handling

### Custom Error Classes
```javascript
// utils/errors.js
class ValidationError extends Error {
  constructor(message) {
    super(message);
    this.name = 'ValidationError';
    this.status = 400;
  }
}

class NotFoundError extends Error {
  constructor(message) {
    super(message);
    this.name = 'NotFoundError';
    this.status = 404;
  }
}

class ConflictError extends Error {
  constructor(message) {
    super(message);
    this.name = 'ConflictError';
    this.status = 409;
  }
}

class UnauthorizedError extends Error {
  constructor(message) {
    super(message);
    this.name = 'UnauthorizedError';
    this.status = 401;
  }
}
```

### Error Handler
```javascript
// utils/errorHandler.js
const handleError = (error, res) => {
  console.error(error);
  
  if (error.name === 'ValidationError') {
    return res.status(400).json({
      status: 'error',
      code: 'VALIDATION_ERROR',
      message: error.message
    });
  }
  
  if (error.name === 'NotFoundError') {
    return res.status(404).json({
      status: 'error',
      code: 'NOT_FOUND',
      message: error.message
    });
  }
  
  if (error.name === 'ConflictError') {
    return res.status(409).json({
      status: 'error',
      code: 'CONFLICT',
      message: error.message
    });
  }
  
  if (error.name === 'UnauthorizedError') {
    return res.status(401).json({
      status: 'error',
      code: 'UNAUTHORIZED',
      message: error.message
    });
  }
  
  return res.status(500).json({
    status: 'error',
    code: 'INTERNAL_SERVER_ERROR',
    message: 'An unexpected error occurred'
  });
};
```

## 🔄 Cache Helpers

### Redis Cache Helper
```javascript
// utils/cache.js
const Redis = require('ioredis');
const config = require('../config');

const redis = new Redis(config.redis);

const cache = {
  async get(key) {
    try {
      const data = await redis.get(key);
      return data ? JSON.parse(data) : null;
    } catch (error) {
      console.error('Cache get error:', error);
      return null;
    }
  },
  
  async set(key, value, ttl = 3600) {
    try {
      await redis.set(key, JSON.stringify(value), 'EX', ttl);
      return true;
    } catch (error) {
      console.error('Cache set error:', error);
      return false;
    }
  },
  
  async del(key) {
    try {
      await redis.del(key);
      return true;
    } catch (error) {
      console.error('Cache delete error:', error);
      return false;
    }
  }
};
```

## 📊 Logger

### Winston Logger
```javascript
// utils/logger.js
const winston = require('winston');
const config = require('../config');

const logger = winston.createLogger({
  level: config.log.level,
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.File({
      filename: 'logs/error.log',
      level: 'error'
    }),
    new winston.transports.File({
      filename: 'logs/combined.log'
    })
  ]
});

if (config.env !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple()
  }));
}
```

## 🔐 Security Helpers

### JWT Helper
```javascript
// utils/jwt.js
const jwt = require('jsonwebtoken');
const config = require('../config');

const jwtHelper = {
  generateToken(payload) {
    return jwt.sign(payload, config.jwt.secret, {
      expiresIn: config.jwt.expiresIn
    });
  },
  
  verifyToken(token) {
    try {
      return jwt.verify(token, config.jwt.secret);
    } catch (error) {
      throw new UnauthorizedError('Invalid token');
    }
  }
};
```

### Password Helper
```javascript
// utils/password.js
const bcrypt = require('bcrypt');

const passwordHelper = {
  async hash(password) {
    const salt = await bcrypt.genSalt(10);
    return bcrypt.hash(password, salt);
  },
  
  async compare(password, hash) {
    return bcrypt.compare(password, hash);
  }
};
```

## 📱 WhatsApp Helpers

### Message Formatter
```javascript
// utils/whatsapp.js
const formatMessage = (type, data) => {
  switch (type) {
    case 'welcome':
      return `Selamat datang di Chatbot Identifikasi Stunting! 🎉\n\n` +
             `Silakan pilih menu berikut:\n` +
             `1. Registrasi\n` +
             `2. Konsultasi\n` +
             `3. Riwayat\n` +
             `4. Bantuan`;
      
    case 'questionnaire':
      return `Pertanyaan ${data.current}/${data.total}:\n\n` +
             `${data.question}\n\n` +
             `Jawaban:\n` +
             `${data.options.join('\n')}`;
      
    case 'result':
      return `Hasil Identifikasi:\n\n` +
             `Skor: ${data.score}\n` +
             `Level Risiko: ${data.riskLevel}\n\n` +
             `Rekomendasi:\n` +
             `${data.recommendations.join('\n')}`;
      
    default:
      return data.message;
  }
};
```

## 📚 Dokumentasi Terkait

- [Controllers](controllers.md)
- [Services](services.md)
- [Models](models.md)

---

<div align="center">

### 🔗 Navigasi

[⬅️ Kembali ke Models](models.md) | [Lanjut ke Local Setup ➡️](../development/local-setup.md)

</div> 