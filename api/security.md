# API Security

## Overview

Dokumentasi ini menjelaskan aspek-aspek keamanan dalam API chatbot, termasuk autentikasi, otorisasi, enkripsi, dan best practices keamanan.

## Authentication

### 1. JWT Authentication

```javascript
const jwtService = {
  generateToken(user) {
    const payload = {
      id: user._id,
      role: user.role
    };
    
    return jwt.sign(payload, process.env.JWT_SECRET, {
      expiresIn: '24h'
    });
  },
  
  verifyToken(token) {
    try {
      return jwt.verify(token, process.env.JWT_SECRET);
    } catch (error) {
      throw new Error('Invalid token');
    }
  },
  
  decodeToken(token) {
    try {
      return jwt.decode(token);
    } catch (error) {
      throw new Error('Invalid token');
    }
  }
};
```

### 2. API Key Authentication

```javascript
const apiKeyService = {
  validateApiKey(apiKey) {
    return apiKey === process.env.API_KEY;
  },
  
  generateApiKey() {
    return crypto.randomBytes(32).toString('hex');
  },
  
  hashApiKey(apiKey) {
    return crypto
      .createHash('sha256')
      .update(apiKey)
      .digest('hex');
  }
};
```

### 3. WhatsApp Authentication

```javascript
const whatsappAuthService = {
  async validatePhoneNumber(phoneNumber) {
    try {
      const user = await User.findOne({ phoneNumber });
      return !!user;
    } catch (error) {
      logger.error('Validate phone number error:', error);
      throw error;
    }
  },
  
  async sendVerificationCode(phoneNumber) {
    try {
      const code = Math.floor(100000 + Math.random() * 900000);
      
      await cacheService.set(
        `verification:${phoneNumber}`,
        code,
        300 // 5 minutes
      );
      
      await whatsappService.sendMessage(
        phoneNumber,
        `Your verification code is: ${code}`
      );
      
      return true;
    } catch (error) {
      logger.error('Send verification code error:', error);
      throw error;
    }
  },
  
  async verifyCode(phoneNumber, code) {
    try {
      const storedCode = await cacheService.get(`verification:${phoneNumber}`);
      
      if (!storedCode || storedCode !== code) {
        return false;
      }
      
      await cacheService.delete(`verification:${phoneNumber}`);
      return true;
    } catch (error) {
      logger.error('Verify code error:', error);
      throw error;
    }
  }
};
```

## Authorization

### 1. Role-Based Access Control

```javascript
const rbacService = {
  roles: {
    admin: ['read', 'write', 'delete', 'manage'],
    user: ['read', 'write']
  },
  
  checkPermission(role, permission) {
    return this.roles[role]?.includes(permission) || false;
  },
  
  middleware(permission) {
    return (req, res, next) => {
      const { role } = req.user;
      
      if (!this.checkPermission(role, permission)) {
        return res.status(403).json({
          success: false,
          error: 'Not authorized'
        });
      }
      
      next();
    };
  }
};
```

### 2. Resource Ownership

```javascript
const ownershipService = {
  async checkChildOwnership(userId, childId) {
    try {
      const child = await Child.findOne({
        _id: childId,
        userId
      });
      
      return !!child;
    } catch (error) {
      logger.error('Check child ownership error:', error);
      throw error;
    }
  },
  
  async checkQuestionnaireOwnership(userId, questionnaireId) {
    try {
      const questionnaire = await Questionnaire.findById(questionnaireId)
        .populate('childId');
      
      return questionnaire?.childId?.userId.toString() === userId;
    } catch (error) {
      logger.error('Check questionnaire ownership error:', error);
      throw error;
    }
  }
};
```

## Encryption

### 1. Data Encryption

```javascript
const encryptionService = {
  encrypt(data) {
    const iv = crypto.randomBytes(16);
    const cipher = crypto.createCipheriv(
      'aes-256-gcm',
      Buffer.from(process.env.ENCRYPTION_KEY, 'hex'),
      iv
    );
    
    let encrypted = cipher.update(JSON.stringify(data), 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    const authTag = cipher.getAuthTag();
    
    return {
      iv: iv.toString('hex'),
      encrypted,
      authTag: authTag.toString('hex')
    };
  },
  
  decrypt(encryptedData) {
    const decipher = crypto.createDecipheriv(
      'aes-256-gcm',
      Buffer.from(process.env.ENCRYPTION_KEY, 'hex'),
      Buffer.from(encryptedData.iv, 'hex')
    );
    
    decipher.setAuthTag(Buffer.from(encryptedData.authTag, 'hex'));
    
    let decrypted = decipher.update(encryptedData.encrypted, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    
    return JSON.parse(decrypted);
  }
};
```

### 2. Password Hashing

```javascript
const passwordService = {
  async hashPassword(password) {
    const salt = await bcrypt.genSalt(10);
    return bcrypt.hash(password, salt);
  },
  
  async comparePassword(password, hash) {
    return bcrypt.compare(password, hash);
  }
};
```

## Security Headers

### 1. Helmet Configuration

```javascript
const helmetConfig = {
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", 'data:', 'https:'],
      connectSrc: ["'self'"],
      fontSrc: ["'self'"],
      objectSrc: ["'none'"],
      mediaSrc: ["'self'"],
      frameSrc: ["'none'"]
    }
  },
  crossOriginEmbedderPolicy: true,
  crossOriginOpenerPolicy: true,
  crossOriginResourcePolicy: { policy: 'same-site' },
  dnsPrefetchControl: true,
  expectCt: true,
  frameguard: { action: 'deny' },
  hidePoweredBy: true,
  hsts: true,
  ieNoOpen: true,
  noSniff: true,
  originAgentCluster: true,
  permittedCrossDomainPolicies: true,
  referrerPolicy: { policy: 'strict-origin-when-cross-origin' },
  xssFilter: true
};
```

### 2. CORS Configuration

```javascript
const corsConfig = {
  origin: process.env.ALLOWED_ORIGINS.split(','),
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  exposedHeaders: ['X-Total-Count'],
  credentials: true,
  maxAge: 86400 // 24 hours
};
```

## Rate Limiting

### 1. Global Rate Limiter

```javascript
const globalRateLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: {
    success: false,
    error: 'Too many requests, please try again later'
  }
});
```

### 2. Route-Specific Rate Limiter

```javascript
const routeRateLimiters = {
  auth: rateLimit({
    windowMs: 60 * 60 * 1000, // 1 hour
    max: 5, // limit each IP to 5 requests per windowMs
    message: {
      success: false,
      error: 'Too many authentication attempts, please try again later'
    }
  }),
  
  messages: rateLimit({
    windowMs: 60 * 1000, // 1 minute
    max: 10, // limit each IP to 10 requests per windowMs
    message: {
      success: false,
      error: 'Too many messages, please try again later'
    }
  })
};
```

## Input Validation

### 1. Request Validation

```javascript
const validationService = {
  validateRequest(schema) {
    return (req, res, next) => {
      const { error } = schema.validate(req.body);
      
      if (error) {
        return res.status(400).json({
          success: false,
          error: error.details[0].message
        });
      }
      
      next();
    };
  },
  
  sanitizeInput(data) {
    return Object.keys(data).reduce((acc, key) => {
      acc[key] = typeof data[key] === 'string' ? data[key].trim() : data[key];
      return acc;
    }, {});
  }
};
```

### 2. File Upload Validation

```javascript
const fileValidationService = {
  validateFile(file, options) {
    const { maxSize, allowedTypes } = options;
    
    if (file.size > maxSize) {
      throw new Error('File too large');
    }
    
    if (!allowedTypes.includes(file.mimetype)) {
      throw new Error('Invalid file type');
    }
    
    return true;
  },
  
  sanitizeFilename(filename) {
    return filename
      .replace(/[^a-zA-Z0-9.-]/g, '_')
      .toLowerCase();
  }
};
```

## Security Monitoring

### 1. Audit Logging

```javascript
const auditService = {
  async logAction(action) {
    try {
      const auditLog = new AuditLog({
        userId: action.userId,
        action: action.type,
        resource: action.resource,
        details: action.details,
        ip: action.ip,
        userAgent: action.userAgent
      });
      
      await auditLog.save();
      
      return true;
    } catch (error) {
      logger.error('Audit log error:', error);
      throw error;
    }
  },
  
  async getAuditLogs(query) {
    try {
      const logs = await AuditLog.find(query)
        .sort({ createdAt: -1 })
        .limit(100);
      
      return logs;
    } catch (error) {
      logger.error('Get audit logs error:', error);
      throw error;
    }
  }
};
```

### 2. Security Alerts

```javascript
const securityAlertService = {
  async createAlert(alert) {
    try {
      const securityAlert = new SecurityAlert({
        type: alert.type,
        severity: alert.severity,
        message: alert.message,
        details: alert.details
      });
      
      await securityAlert.save();
      
      // Send notification
      await this.notifySecurityTeam(securityAlert);
      
      return securityAlert;
    } catch (error) {
      logger.error('Create security alert error:', error);
      throw error;
    }
  },
  
  async notifySecurityTeam(alert) {
    try {
      // Send email
      await notificationService.sendEmail(
        process.env.SECURITY_EMAIL,
        `Security Alert: ${alert.type}`,
        this.formatAlertEmail(alert)
      );
      
      // Send Slack notification
      await slackService.sendMessage(
        process.env.SECURITY_SLACK_CHANNEL,
        this.formatAlertSlack(alert)
      );
      
      return true;
    } catch (error) {
      logger.error('Notify security team error:', error);
      throw error;
    }
  }
};
```

## Best Practices

1. **Authentication**
   - Use strong passwords
   - Implement MFA
   - Regular token rotation
   - Secure session management

2. **Authorization**
   - Principle of least privilege
   - Role-based access control
   - Resource ownership checks
   - Regular permission audits

3. **Data Protection**
   - Encrypt sensitive data
   - Secure data transmission
   - Regular backups
   - Data retention policies

4. **API Security**
   - Input validation
   - Rate limiting
   - Security headers
   - CORS configuration

5. **Monitoring**
   - Audit logging
   - Security alerts
   - Performance monitoring
   - Error tracking 