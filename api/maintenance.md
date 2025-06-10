# API Maintenance

## Overview

Dokumentasi ini menjelaskan praktik dan prosedur maintenance untuk API chatbot, termasuk regular maintenance, performance optimization, security updates, dan best practices.

## Regular Maintenance

### 1. Dependency Updates

```javascript
// scripts/update-dependencies.js
const { exec } = require('child_process');
const fs = require('fs');
const path = require('path');

const updateDependencies = async () => {
  try {
    // Check for outdated packages
    const { stdout } = await exec('npm outdated --json');
    const outdated = JSON.parse(stdout);
    
    // Update dependencies
    for (const [name, info] of Object.entries(outdated)) {
      console.log(`Updating ${name} from ${info.current} to ${info.latest}`);
      await exec(`npm install ${name}@latest`);
    }
    
    // Update package-lock.json
    await exec('npm install');
    
    // Run tests
    await exec('npm test');
    
    console.log('Dependencies updated successfully');
  } catch (error) {
    console.error('Failed to update dependencies:', error);
    process.exit(1);
  }
};

updateDependencies();
```

### 2. Log Rotation

```javascript
// config/logging.js
const winston = require('winston');
require('winston-daily-rotate-file');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.DailyRotateFile({
      filename: 'logs/error-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      level: 'error',
      maxSize: '20m',
      maxFiles: '14d'
    }),
    new winston.transports.DailyRotateFile({
      filename: 'logs/combined-%DATE%.log',
      datePattern: 'YYYY-MM-DD',
      maxSize: '20m',
      maxFiles: '14d'
    })
  ]
});
```

## Performance Optimization

### 1. Cache Optimization

```javascript
// services/cache.js
const Redis = require('ioredis');
const redis = new Redis(process.env.REDIS_URI);

class CacheService {
  constructor() {
    this.defaultTTL = 3600; // 1 hour
  }
  
  async get(key) {
    const data = await redis.get(key);
    return data ? JSON.parse(data) : null;
  }
  
  async set(key, value, ttl = this.defaultTTL) {
    await redis.set(
      key,
      JSON.stringify(value),
      'EX',
      ttl
    );
  }
  
  async del(key) {
    await redis.del(key);
  }
  
  async clear() {
    await redis.flushall();
  }
  
  // Cache warming
  async warmCache() {
    const users = await User.find().lean();
    for (const user of users) {
      await this.set(`user:${user._id}`, user);
    }
  }
}
```

### 2. Query Optimization

```javascript
// models/user.js
const userSchema = new mongoose.Schema({
  phoneNumber: {
    type: String,
    required: true,
    unique: true,
    index: true
  },
  name: String,
  role: {
    type: String,
    enum: ['user', 'admin'],
    default: 'user',
    index: true
  },
  status: {
    type: String,
    enum: ['active', 'inactive'],
    default: 'active',
    index: true
  }
});

// Compound index
userSchema.index({ role: 1, status: 1 });

// Text index for search
userSchema.index({ name: 'text' });
```

## Security Updates

### 1. Security Scanning

```javascript
// scripts/security-scan.js
const { exec } = require('child_process');
const fs = require('fs');
const path = require('path');

const runSecurityScan = async () => {
  try {
    // Run npm audit
    const { stdout: auditOutput } = await exec('npm audit --json');
    const auditResults = JSON.parse(auditOutput);
    
    // Run snyk test
    const { stdout: snykOutput } = await exec('snyk test --json');
    const snykResults = JSON.parse(snykOutput);
    
    // Generate report
    const report = {
      timestamp: new Date(),
      npmAudit: auditResults,
      snykTest: snykResults
    };
    
    fs.writeFileSync(
      path.join(__dirname, '../reports/security-scan.json'),
      JSON.stringify(report, null, 2)
    );
    
    console.log('Security scan completed');
  } catch (error) {
    console.error('Security scan failed:', error);
    process.exit(1);
  }
};

runSecurityScan();
```

### 2. Security Patches

```javascript
// scripts/apply-security-patches.js
const { exec } = require('child_process');
const fs = require('fs');
const path = require('path');

const applySecurityPatches = async () => {
  try {
    // Backup package.json
    const packageJson = require('../package.json');
    fs.writeFileSync(
      path.join(__dirname, '../backups/package.json'),
      JSON.stringify(packageJson, null, 2)
    );
    
    // Update dependencies
    await exec('npm audit fix --force');
    
    // Run tests
    await exec('npm test');
    
    console.log('Security patches applied successfully');
  } catch (error) {
    console.error('Failed to apply security patches:', error);
    
    // Restore package.json
    const backup = require('../backups/package.json');
    fs.writeFileSync(
      path.join(__dirname, '../package.json'),
      JSON.stringify(backup, null, 2)
    );
    
    process.exit(1);
  }
};

applySecurityPatches();
```

## Database Maintenance

### 1. Index Optimization

```javascript
// scripts/optimize-indexes.js
const mongoose = require('mongoose');

const optimizeIndexes = async () => {
  try {
    // Get all collections
    const collections = await mongoose.connection.db.collections();
    
    for (const collection of collections) {
      // Get current indexes
      const indexes = await collection.indexes();
      
      // Analyze index usage
      const stats = await collection.stats();
      
      // Remove unused indexes
      for (const index of indexes) {
        if (index.name !== '_id_' && !stats.indexDetails[index.name].accesses.ops) {
          await collection.dropIndex(index.name);
          console.log(`Dropped unused index: ${index.name}`);
        }
      }
    }
    
    console.log('Index optimization completed');
  } catch (error) {
    console.error('Index optimization failed:', error);
    process.exit(1);
  }
};

optimizeIndexes();
```

### 2. Data Cleanup

```javascript
// scripts/cleanup-data.js
const mongoose = require('mongoose');

const cleanupData = async () => {
  try {
    // Clean up expired sessions
    await Session.deleteMany({
      expiresAt: { $lt: new Date() }
    });
    
    // Clean up old messages
    const thirtyDaysAgo = new Date();
    thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);
    
    await Message.deleteMany({
      createdAt: { $lt: thirtyDaysAgo }
    });
    
    // Clean up inactive users
    const sixMonthsAgo = new Date();
    sixMonthsAgo.setMonth(sixMonthsAgo.getMonth() - 6);
    
    await User.updateMany(
      {
        lastActive: { $lt: sixMonthsAgo },
        status: 'active'
      },
      {
        $set: { status: 'inactive' }
      }
    );
    
    console.log('Data cleanup completed');
  } catch (error) {
    console.error('Data cleanup failed:', error);
    process.exit(1);
  }
};

cleanupData();
```

## Log Management

### 1. Log Analysis

```javascript
// scripts/analyze-logs.js
const fs = require('fs');
const path = require('path');
const readline = require('readline');

const analyzeLogs = async () => {
  const logDir = path.join(__dirname, '../logs');
  const files = fs.readdirSync(logDir);
  
  const stats = {
    errors: 0,
    warnings: 0,
    info: 0,
    errorTypes: {},
    slowQueries: []
  };
  
  for (const file of files) {
    if (!file.endsWith('.log')) continue;
    
    const fileStream = fs.createReadStream(path.join(logDir, file));
    const rl = readline.createInterface({
      input: fileStream,
      crlfDelay: Infinity
    });
    
    for await (const line of rl) {
      const log = JSON.parse(line);
      
      // Count log levels
      stats[log.level]++;
      
      // Track error types
      if (log.level === 'error') {
        const errorType = log.error?.name || 'Unknown';
        stats.errorTypes[errorType] = (stats.errorTypes[errorType] || 0) + 1;
      }
      
      // Track slow queries
      if (log.duration > 1000) {
        stats.slowQueries.push({
          query: log.query,
          duration: log.duration,
          timestamp: log.timestamp
        });
      }
    }
  }
  
  // Generate report
  fs.writeFileSync(
    path.join(__dirname, '../reports/log-analysis.json'),
    JSON.stringify(stats, null, 2)
  );
  
  console.log('Log analysis completed');
};

analyzeLogs();
```

### 2. Log Archiving

```javascript
// scripts/archive-logs.js
const fs = require('fs');
const path = require('path');
const archiver = require('archiver');

const archiveLogs = async () => {
  const logDir = path.join(__dirname, '../logs');
  const archivePath = path.join(__dirname, '../archives/logs.zip');
  
  const output = fs.createWriteStream(archivePath);
  const archive = archiver('zip', {
    zlib: { level: 9 }
  });
  
  output.on('close', () => {
    console.log(`Logs archived: ${archive.pointer()} bytes`);
  });
  
  archive.on('error', (err) => {
    throw err;
  });
  
  archive.pipe(output);
  
  // Archive old log files
  const files = fs.readdirSync(logDir);
  for (const file of files) {
    if (file.endsWith('.log')) {
      const filePath = path.join(logDir, file);
      const stats = fs.statSync(filePath);
      
      // Archive files older than 30 days
      if (Date.now() - stats.mtime.getTime() > 30 * 24 * 60 * 60 * 1000) {
        archive.file(filePath, { name: file });
        fs.unlinkSync(filePath);
      }
    }
  }
  
  await archive.finalize();
};

archiveLogs();
```

## Error Handling

### 1. Error Monitoring

```javascript
// services/error-monitor.js
const Sentry = require('@sentry/node');
const logger = require('../config/logger');

class ErrorMonitor {
  constructor() {
    this.errorCounts = new Map();
    this.errorThreshold = 10;
    this.timeWindow = 3600000; // 1 hour
  }
  
  trackError(error) {
    const errorKey = `${error.name}:${error.message}`;
    const now = Date.now();
    
    // Update error count
    if (!this.errorCounts.has(errorKey)) {
      this.errorCounts.set(errorKey, []);
    }
    
    const timestamps = this.errorCounts.get(errorKey);
    timestamps.push(now);
    
    // Remove old timestamps
    const cutoff = now - this.timeWindow;
    while (timestamps[0] < cutoff) {
      timestamps.shift();
    }
    
    // Check threshold
    if (timestamps.length >= this.errorThreshold) {
      this.alertError(errorKey, timestamps.length);
    }
    
    // Log error
    logger.error({
      error: {
        name: error.name,
        message: error.message,
        stack: error.stack
      },
      count: timestamps.length
    });
    
    // Send to Sentry
    Sentry.captureException(error);
  }
  
  alertError(errorKey, count) {
    logger.warn({
      message: 'Error threshold exceeded',
      error: errorKey,
      count: count
    });
    
    // Send alert (e.g., email, Slack)
  }
}
```

### 2. Error Recovery

```javascript
// services/error-recovery.js
const mongoose = require('mongoose');
const logger = require('../config/logger');

class ErrorRecovery {
  async handleDatabaseError(error) {
    if (error.name === 'MongoError' && error.code === 11000) {
      // Handle duplicate key error
      return this.handleDuplicateKey(error);
    }
    
    if (error.name === 'MongoError' && error.code === 121) {
      // Handle validation error
      return this.handleValidationError(error);
    }
    
    // Handle connection error
    if (error.name === 'MongoError' && error.code === 18) {
      return this.handleConnectionError();
    }
    
    throw error;
  }
  
  async handleDuplicateKey(error) {
    const field = Object.keys(error.keyPattern)[0];
    const value = error.keyValue[field];
    
    logger.warn({
      message: 'Duplicate key error',
      field: field,
      value: value
    });
    
    // Return appropriate error response
    return {
      status: 'error',
      message: `${field} already exists`
    };
  }
  
  async handleValidationError(error) {
    logger.warn({
      message: 'Validation error',
      details: error.errmsg
    });
    
    // Return appropriate error response
    return {
      status: 'error',
      message: 'Validation failed',
      details: error.errmsg
    };
  }
  
  async handleConnectionError() {
    logger.error('Database connection error');
    
    // Attempt to reconnect
    try {
      await mongoose.connect(process.env.MONGODB_URI, {
        useNewUrlParser: true,
        useUnifiedTopology: true
      });
      
      logger.info('Database reconnected successfully');
    } catch (error) {
      logger.error('Failed to reconnect to database');
      throw error;
    }
  }
}
```

## Best Practices

1. **Regular Maintenance**
   - Update dependencies
   - Rotate logs
   - Clean up data
   - Monitor performance

2. **Performance Optimization**
   - Optimize cache
   - Optimize queries
   - Monitor metrics
   - Set benchmarks

3. **Security Updates**
   - Regular scans
   - Apply patches
   - Update dependencies
   - Monitor vulnerabilities

4. **Database Maintenance**
   - Optimize indexes
   - Clean up data
   - Monitor performance
   - Regular backups

5. **Log Management**
   - Rotate logs
   - Analyze logs
   - Archive logs
   - Monitor errors

6. **Error Handling**
   - Monitor errors
   - Track patterns
   - Set alerts
   - Implement recovery

7. **Documentation**
   - Update docs
   - Track changes
   - Document procedures
   - Share knowledge 