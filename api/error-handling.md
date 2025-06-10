# API Error Handling

## Overview

Dokumentasi ini menjelaskan strategi dan implementasi penanganan error dalam API chatbot.

## Error Types

### 1. Validation Errors

```javascript
class ValidationError extends Error {
  constructor(message, details) {
    super(message);
    this.name = 'ValidationError';
    this.status = 400;
    this.details = details;
  }
}

// Usage
function validateUserData(data) {
  const errors = [];
  
  if (!data.nama || data.nama.length < 3) {
    errors.push('Nama harus minimal 3 karakter');
  }
  
  if (!data.nomor_telepon || !/^62\d{9,12}$/.test(data.nomor_telepon)) {
    errors.push('Nomor telepon tidak valid');
  }
  
  if (errors.length > 0) {
    throw new ValidationError('Data tidak valid', errors);
  }
}
```

### 2. Authentication Errors

```javascript
class AuthenticationError extends Error {
  constructor(message) {
    super(message);
    this.name = 'AuthenticationError';
    this.status = 401;
  }
}

// Usage
function validateToken(token) {
  if (!token) {
    throw new AuthenticationError('Token tidak ditemukan');
  }
  
  if (!token.startsWith('Bearer ')) {
    throw new AuthenticationError('Format token tidak valid');
  }
  
  const apiKey = token.split(' ')[1];
  if (apiKey !== process.env.API_KEY) {
    throw new AuthenticationError('Token tidak valid');
  }
}
```

### 3. Authorization Errors

```javascript
class AuthorizationError extends Error {
  constructor(message) {
    super(message);
    this.name = 'AuthorizationError';
    this.status = 403;
  }
}

// Usage
function checkPermission(userId, resourceId) {
  if (userId !== resourceId) {
    throw new AuthorizationError('Tidak memiliki akses ke resource ini');
  }
}
```

### 4. Not Found Errors

```javascript
class NotFoundError extends Error {
  constructor(message) {
    super(message);
    this.name = 'NotFoundError';
    this.status = 404;
  }
}

// Usage
function findUser(userId) {
  const user = users.get(userId);
  if (!user) {
    throw new NotFoundError(`User dengan ID ${userId} tidak ditemukan`);
  }
  return user;
}
```

### 5. Rate Limit Errors

```javascript
class RateLimitError extends Error {
  constructor(message, retryAfter) {
    super(message);
    this.name = 'RateLimitError';
    this.status = 429;
    this.retryAfter = retryAfter;
  }
}

// Usage
function checkRateLimit(userId) {
  const requests = rateLimiter.get(userId);
  if (requests >= MAX_REQUESTS) {
    throw new RateLimitError('Terlalu banyak request', 60);
  }
}
```

## Error Handler Middleware

```javascript
function errorHandler(err, req, res, next) {
  // Log error
  console.error({
    timestamp: new Date().toISOString(),
    error: {
      name: err.name,
      message: err.message,
      stack: err.stack
    },
    request: {
      method: req.method,
      url: req.url,
      body: req.body,
      headers: req.headers
    }
  });
  
  // Determine status code
  const status = err.status || 500;
  
  // Prepare error response
  const response = {
    status: 'error',
    message: process.env.NODE_ENV === 'production' 
      ? 'Terjadi kesalahan pada server' 
      : err.message
  };
  
  // Add details if available
  if (err.details) {
    response.details = err.details;
  }
  
  // Add retry after for rate limit errors
  if (err.name === 'RateLimitError') {
    res.set('Retry-After', err.retryAfter);
  }
  
  // Send response
  res.status(status).json(response);
}
```

## Error Response Format

### 1. Validation Error
```json
{
  "status": "error",
  "message": "Data tidak valid",
  "details": [
    "Nama harus minimal 3 karakter",
    "Nomor telepon tidak valid"
  ]
}
```

### 2. Authentication Error
```json
{
  "status": "error",
  "message": "Token tidak valid"
}
```

### 3. Authorization Error
```json
{
  "status": "error",
  "message": "Tidak memiliki akses ke resource ini"
}
```

### 4. Not Found Error
```json
{
  "status": "error",
  "message": "User dengan ID 123 tidak ditemukan"
}
```

### 5. Rate Limit Error
```json
{
  "status": "error",
  "message": "Terlalu banyak request"
}
```

## Error Logging

### 1. Console Logging

```javascript
function logError(error, context) {
  console.error({
    timestamp: new Date().toISOString(),
    error: {
      name: error.name,
      message: error.message,
      stack: error.stack
    },
    context
  });
}
```

### 2. File Logging

```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: 'error',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ 
      filename: 'error.log',
      level: 'error'
    })
  ]
});

function logError(error, context) {
  logger.error({
    timestamp: new Date().toISOString(),
    error: {
      name: error.name,
      message: error.message,
      stack: error.stack
    },
    context
  });
}
```

## Error Recovery

### 1. Retry Mechanism

```javascript
async function withRetry(fn, maxRetries = 3) {
  let retries = 0;
  
  while (retries < maxRetries) {
    try {
      return await fn();
    } catch (error) {
      retries++;
      
      if (retries === maxRetries) {
        throw error;
      }
      
      // Exponential backoff
      const delay = Math.pow(2, retries) * 1000;
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}
```

### 2. Circuit Breaker

```javascript
class CircuitBreaker {
  constructor(fn, options = {}) {
    this.fn = fn;
    this.state = 'CLOSED';
    this.failureCount = 0;
    this.lastFailureTime = null;
    this.threshold = options.threshold || 5;
    this.timeout = options.timeout || 60000;
  }
  
  async execute(...args) {
    if (this.state === 'OPEN') {
      if (this.lastFailureTime && 
          Date.now() - this.lastFailureTime > this.timeout) {
        this.state = 'HALF-OPEN';
      } else {
        throw new Error('Circuit breaker is open');
      }
    }
    
    try {
      const result = await this.fn(...args);
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
  
  onSuccess() {
    this.failureCount = 0;
    this.state = 'CLOSED';
  }
  
  onFailure() {
    this.failureCount++;
    this.lastFailureTime = Date.now();
    
    if (this.failureCount >= this.threshold) {
      this.state = 'OPEN';
    }
  }
}
```

## Error Monitoring

### 1. Health Checks

```javascript
async function checkHealth() {
  try {
    // Check database connection
    await db.ping();
    
    // Check external services
    await checkWhatsAppConnection();
    
    return {
      status: 'healthy',
      timestamp: new Date().toISOString()
    };
  } catch (error) {
    return {
      status: 'unhealthy',
      error: error.message,
      timestamp: new Date().toISOString()
    };
  }
}
```

### 2. Error Metrics

```javascript
const prometheus = require('prom-client');

const errorCounter = new prometheus.Counter({
  name: 'api_errors_total',
  help: 'Total number of API errors',
  labelNames: ['error_type']
});

function trackError(error) {
  errorCounter.inc({ error_type: error.name });
}
```

## Best Practices

1. **Error Classification**
   - Use appropriate error types
   - Include relevant details
   - Maintain consistent format

2. **Error Handling**
   - Handle errors at appropriate level
   - Log errors with context
   - Implement recovery mechanisms

3. **Error Responses**
   - Use appropriate status codes
   - Include helpful messages
   - Hide sensitive information

4. **Error Monitoring**
   - Track error rates
   - Set up alerts
   - Monitor system health

5. **Error Prevention**
   - Validate input
   - Implement timeouts
   - Use circuit breakers

## Maintenance

### Regular Tasks

1. **Daily**
   - Review error logs
   - Check error rates
   - Monitor system health

2. **Weekly**
   - Analyze error patterns
   - Update error handling
   - Clean up old logs

3. **Monthly**
   - Review error metrics
   - Update error documentation
   - Optimize error handling 