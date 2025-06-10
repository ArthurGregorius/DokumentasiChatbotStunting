# API Troubleshooting

## Overview

Dokumentasi ini menjelaskan langkah-langkah troubleshooting untuk API chatbot, termasuk common issues, debugging, error resolution, dan best practices.

## Common Issues

### 1. Connection Issues

```javascript
// services/connection-monitor.js
class ConnectionMonitor {
  constructor() {
    this.issues = new Map();
    this.threshold = 3;
  }
  
  async checkConnections() {
    try {
      // Check MongoDB connection
      await mongoose.connection.db.admin().ping();
      
      // Check Redis connection
      await redis.ping();
      
      // Check WhatsApp connection
      const whatsappStatus = await whatsappService.getStatus();
      
      return {
        mongodb: 'connected',
        redis: 'connected',
        whatsapp: whatsappStatus
      };
    } catch (error) {
      this.trackIssue('connection', error);
      throw error;
    }
  }
  
  trackIssue(type, error) {
    if (!this.issues.has(type)) {
      this.issues.set(type, []);
    }
    
    const issues = this.issues.get(type);
    issues.push({
      timestamp: new Date(),
      error: error.message
    });
    
    // Keep only last 10 issues
    if (issues.length > 10) {
      issues.shift();
    }
    
    // Check threshold
    if (issues.length >= this.threshold) {
      this.alertIssue(type, issues);
    }
  }
  
  alertIssue(type, issues) {
    logger.error({
      message: `Connection issue detected: ${type}`,
      issues: issues
    });
    
    // Send alert (e.g., email, Slack)
  }
}
```

### 2. Authentication Issues

```javascript
// services/auth-troubleshooter.js
class AuthTroubleshooter {
  async diagnoseAuthIssue(token) {
    try {
      // Verify token
      const decoded = jwt.verify(token, process.env.JWT_SECRET);
      
      // Check user exists
      const user = await User.findById(decoded.userId);
      if (!user) {
        return {
          status: 'error',
          message: 'User not found',
          code: 'USER_NOT_FOUND'
        };
      }
      
      // Check user status
      if (user.status !== 'active') {
        return {
          status: 'error',
          message: 'User is inactive',
          code: 'USER_INACTIVE'
        };
      }
      
      // Check token expiration
      if (decoded.exp < Date.now() / 1000) {
        return {
          status: 'error',
          message: 'Token expired',
          code: 'TOKEN_EXPIRED'
        };
      }
      
      return {
        status: 'success',
        message: 'Authentication valid'
      };
    } catch (error) {
      return {
        status: 'error',
        message: error.message,
        code: 'AUTH_ERROR'
      };
    }
  }
}
```

## Debugging

### 1. Request Debugging

```javascript
// middleware/request-debugger.js
const requestDebugger = (req, res, next) => {
  const start = Date.now();
  
  // Log request
  logger.debug({
    method: req.method,
    url: req.url,
    headers: req.headers,
    body: req.body,
    query: req.query,
    params: req.params
  });
  
  // Capture response
  const originalSend = res.send;
  res.send = function(body) {
    const duration = Date.now() - start;
    
    // Log response
    logger.debug({
      status: res.statusCode,
      duration: duration,
      body: body
    });
    
    return originalSend.call(this, body);
  };
  
  next();
};
```

### 2. Error Debugging

```javascript
// services/error-debugger.js
class ErrorDebugger {
  constructor() {
    this.errorLog = new Map();
  }
  
  async debugError(error, context = {}) {
    // Capture error details
    const errorDetails = {
      name: error.name,
      message: error.message,
      stack: error.stack,
      context: context,
      timestamp: new Date()
    };
    
    // Log error
    logger.error(errorDetails);
    
    // Store error
    const errorId = this.generateErrorId();
    this.errorLog.set(errorId, errorDetails);
    
    // Analyze error
    const analysis = await this.analyzeError(error);
    
    return {
      errorId: errorId,
      analysis: analysis
    };
  }
  
  generateErrorId() {
    return `ERR-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }
  
  async analyzeError(error) {
    // Check for common patterns
    const patterns = {
      database: /MongoError|MongooseError/,
      validation: /ValidationError/,
      authentication: /JsonWebTokenError|TokenExpiredError/,
      network: /ECONNREFUSED|ETIMEDOUT/
    };
    
    for (const [type, pattern] of Object.entries(patterns)) {
      if (pattern.test(error.name)) {
        return {
          type: type,
          suggestions: this.getSuggestions(type)
        };
      }
    }
    
    return {
      type: 'unknown',
      suggestions: ['Check application logs for more details']
    };
  }
  
  getSuggestions(type) {
    const suggestions = {
      database: [
        'Check database connection',
        'Verify database credentials',
        'Check database server status'
      ],
      validation: [
        'Verify input data',
        'Check validation rules',
        'Review schema definitions'
      ],
      authentication: [
        'Verify token validity',
        'Check token expiration',
        'Validate user credentials'
      ],
      network: [
        'Check network connectivity',
        'Verify server status',
        'Check firewall settings'
      ]
    };
    
    return suggestions[type] || [];
  }
}
```

## Error Resolution

### 1. Error Recovery

```javascript
// services/error-recovery.js
class ErrorRecovery {
  async recoverFromError(error, context) {
    // Log error
    logger.error({
      error: error,
      context: context
    });
    
    // Determine recovery strategy
    const strategy = await this.determineStrategy(error);
    
    // Execute recovery
    try {
      await this.executeStrategy(strategy, context);
      
      logger.info({
        message: 'Error recovery successful',
        strategy: strategy
      });
      
      return {
        status: 'success',
        message: 'Recovery completed'
      };
    } catch (recoveryError) {
      logger.error({
        message: 'Error recovery failed',
        error: recoveryError
      });
      
      throw recoveryError;
    }
  }
  
  async determineStrategy(error) {
    if (error.name === 'MongoError') {
      return {
        type: 'database',
        action: 'reconnect'
      };
    }
    
    if (error.name === 'RedisError') {
      return {
        type: 'cache',
        action: 'clear'
      };
    }
    
    if (error.name === 'WhatsAppError') {
      return {
        type: 'whatsapp',
        action: 'restart'
      };
    }
    
    return {
      type: 'unknown',
      action: 'retry'
    };
  }
  
  async executeStrategy(strategy, context) {
    switch (strategy.type) {
      case 'database':
        await this.recoverDatabase();
        break;
      case 'cache':
        await this.recoverCache();
        break;
      case 'whatsapp':
        await this.recoverWhatsApp();
        break;
      default:
        await this.retryOperation(context);
    }
  }
  
  async recoverDatabase() {
    await mongoose.disconnect();
    await mongoose.connect(process.env.MONGODB_URI);
  }
  
  async recoverCache() {
    await redis.flushall();
  }
  
  async recoverWhatsApp() {
    await whatsappService.disconnect();
    await whatsappService.connect();
  }
  
  async retryOperation(context) {
    const maxRetries = 3;
    let retries = 0;
    
    while (retries < maxRetries) {
      try {
        await context.operation();
        break;
      } catch (error) {
        retries++;
        if (retries === maxRetries) {
          throw error;
        }
        await new Promise(resolve => setTimeout(resolve, 1000 * retries));
      }
    }
  }
}
```

### 2. Error Prevention

```javascript
// services/error-prevention.js
class ErrorPrevention {
  constructor() {
    this.errorPatterns = new Map();
    this.preventionRules = new Map();
  }
  
  async analyzeErrorPatterns() {
    const errors = await this.getRecentErrors();
    
    for (const error of errors) {
      const pattern = this.extractPattern(error);
      this.errorPatterns.set(pattern, (this.errorPatterns.get(pattern) || 0) + 1);
    }
    
    // Generate prevention rules
    for (const [pattern, count] of this.errorPatterns.entries()) {
      if (count >= 5) {
        this.generatePreventionRule(pattern);
      }
    }
  }
  
  extractPattern(error) {
    return {
      type: error.name,
      message: error.message,
      stack: error.stack.split('\n')[1]
    };
  }
  
  generatePreventionRule(pattern) {
    const rule = {
      pattern: pattern,
      prevention: this.getPreventionStrategy(pattern),
      timestamp: new Date()
    };
    
    this.preventionRules.set(pattern, rule);
    
    // Apply prevention rule
    this.applyPreventionRule(rule);
  }
  
  getPreventionStrategy(pattern) {
    const strategies = {
      'MongoError': 'Implement connection pooling',
      'ValidationError': 'Add input validation',
      'JsonWebTokenError': 'Implement token refresh',
      'RedisError': 'Add fallback cache'
    };
    
    return strategies[pattern.type] || 'Add error handling';
  }
  
  applyPreventionRule(rule) {
    logger.info({
      message: 'Applying prevention rule',
      rule: rule
    });
    
    // Implement prevention strategy
    switch (rule.prevention) {
      case 'Implement connection pooling':
        this.implementConnectionPooling();
        break;
      case 'Add input validation':
        this.addInputValidation();
        break;
      case 'Implement token refresh':
        this.implementTokenRefresh();
        break;
      case 'Add fallback cache':
        this.addFallbackCache();
        break;
      default:
        this.addErrorHandling();
    }
  }
}
```

## Performance Issues

### 1. Performance Monitoring

```javascript
// services/performance-monitor.js
class PerformanceMonitor {
  constructor() {
    this.metrics = new Map();
    this.thresholds = {
      responseTime: 1000,
      memoryUsage: 0.8,
      cpuUsage: 0.7
    };
  }
  
  async monitorPerformance() {
    // Monitor response time
    this.monitorResponseTime();
    
    // Monitor memory usage
    this.monitorMemoryUsage();
    
    // Monitor CPU usage
    this.monitorCpuUsage();
    
    // Monitor database performance
    await this.monitorDatabasePerformance();
    
    // Monitor cache performance
    await this.monitorCachePerformance();
  }
  
  monitorResponseTime() {
    const start = Date.now();
    
    return {
      end: () => {
        const duration = Date.now() - start;
        
        if (duration > this.thresholds.responseTime) {
          this.alertPerformanceIssue('responseTime', duration);
        }
        
        return duration;
      }
    };
  }
  
  monitorMemoryUsage() {
    const usage = process.memoryUsage();
    const heapUsed = usage.heapUsed / usage.heapTotal;
    
    if (heapUsed > this.thresholds.memoryUsage) {
      this.alertPerformanceIssue('memoryUsage', heapUsed);
    }
    
    return heapUsed;
  }
  
  monitorCpuUsage() {
    const usage = process.cpuUsage();
    const totalUsage = (usage.user + usage.system) / 1000000;
    
    if (totalUsage > this.thresholds.cpuUsage) {
      this.alertPerformanceIssue('cpuUsage', totalUsage);
    }
    
    return totalUsage;
  }
  
  async monitorDatabasePerformance() {
    const stats = await mongoose.connection.db.stats();
    
    if (stats.avgObjSize > 1000000) {
      this.alertPerformanceIssue('databaseSize', stats.avgObjSize);
    }
    
    return stats;
  }
  
  async monitorCachePerformance() {
    const info = await redis.info();
    
    if (info.used_memory > 1000000000) {
      this.alertPerformanceIssue('cacheSize', info.used_memory);
    }
    
    return info;
  }
  
  alertPerformanceIssue(type, value) {
    logger.warn({
      message: 'Performance issue detected',
      type: type,
      value: value,
      threshold: this.thresholds[type]
    });
    
    // Send alert (e.g., email, Slack)
  }
}
```

### 2. Performance Optimization

```javascript
// services/performance-optimizer.js
class PerformanceOptimizer {
  async optimizePerformance() {
    // Optimize database
    await this.optimizeDatabase();
    
    // Optimize cache
    await this.optimizeCache();
    
    // Optimize memory
    this.optimizeMemory();
    
    // Optimize CPU
    this.optimizeCpu();
  }
  
  async optimizeDatabase() {
    // Analyze slow queries
    const slowQueries = await this.analyzeSlowQueries();
    
    // Optimize indexes
    for (const query of slowQueries) {
      await this.optimizeIndex(query);
    }
    
    // Clean up old data
    await this.cleanupOldData();
  }
  
  async optimizeCache() {
    // Analyze cache usage
    const cacheStats = await this.analyzeCacheUsage();
    
    // Optimize cache size
    if (cacheStats.memoryUsage > 0.8) {
      await this.reduceCacheSize();
    }
    
    // Optimize cache keys
    await this.optimizeCacheKeys();
  }
  
  optimizeMemory() {
    // Force garbage collection
    if (global.gc) {
      global.gc();
    }
    
    // Clear unused variables
    this.clearUnusedVariables();
  }
  
  optimizeCpu() {
    // Optimize event loop
    this.optimizeEventLoop();
    
    // Optimize worker threads
    this.optimizeWorkerThreads();
  }
}
```

## Security Issues

### 1. Security Monitoring

```javascript
// services/security-monitor.js
class SecurityMonitor {
  constructor() {
    this.vulnerabilities = new Map();
    this.attacks = new Map();
  }
  
  async monitorSecurity() {
    // Monitor vulnerabilities
    await this.monitorVulnerabilities();
    
    // Monitor attacks
    await this.monitorAttacks();
    
    // Monitor access
    await this.monitorAccess();
  }
  
  async monitorVulnerabilities() {
    // Check npm vulnerabilities
    const { stdout } = await exec('npm audit --json');
    const audit = JSON.parse(stdout);
    
    for (const vuln of audit.vulnerabilities) {
      this.vulnerabilities.set(vuln.id, {
        name: vuln.name,
        severity: vuln.severity,
        fix: vuln.fix
      });
    }
    
    // Alert high severity vulnerabilities
    const highSeverity = Array.from(this.vulnerabilities.values())
      .filter(v => v.severity === 'high');
    
    if (highSeverity.length > 0) {
      this.alertVulnerability(highSeverity);
    }
  }
  
  async monitorAttacks() {
    // Monitor failed login attempts
    const failedLogins = await this.getFailedLogins();
    
    for (const login of failedLogins) {
      if (login.attempts >= 5) {
        this.attacks.set(login.ip, {
          type: 'brute_force',
          attempts: login.attempts,
          timestamp: new Date()
        });
      }
    }
    
    // Alert suspicious activity
    if (this.attacks.size > 0) {
      this.alertAttack(Array.from(this.attacks.values()));
    }
  }
  
  async monitorAccess() {
    // Monitor API access
    const accessLogs = await this.getAccessLogs();
    
    for (const log of accessLogs) {
      if (this.isSuspiciousAccess(log)) {
        this.alertSuspiciousAccess(log);
      }
    }
  }
  
  isSuspiciousAccess(log) {
    return (
      log.status === 403 ||
      log.status === 401 ||
      log.method === 'DELETE' ||
      log.path.includes('/admin')
    );
  }
  
  alertVulnerability(vulnerabilities) {
    logger.warn({
      message: 'Security vulnerabilities detected',
      vulnerabilities: vulnerabilities
    });
    
    // Send alert (e.g., email, Slack)
  }
  
  alertAttack(attacks) {
    logger.warn({
      message: 'Security attacks detected',
      attacks: attacks
    });
    
    // Send alert (e.g., email, Slack)
  }
  
  alertSuspiciousAccess(log) {
    logger.warn({
      message: 'Suspicious access detected',
      log: log
    });
    
    // Send alert (e.g., email, Slack)
  }
}
```

### 2. Security Resolution

```javascript
// services/security-resolver.js
class SecurityResolver {
  async resolveSecurityIssue(issue) {
    // Log issue
    logger.warn({
      message: 'Security issue detected',
      issue: issue
    });
    
    // Determine resolution strategy
    const strategy = await this.determineStrategy(issue);
    
    // Execute resolution
    try {
      await this.executeStrategy(strategy);
      
      logger.info({
        message: 'Security issue resolved',
        strategy: strategy
      });
      
      return {
        status: 'success',
        message: 'Resolution completed'
      };
    } catch (error) {
      logger.error({
        message: 'Security resolution failed',
        error: error
      });
      
      throw error;
    }
  }
  
  async determineStrategy(issue) {
    switch (issue.type) {
      case 'vulnerability':
        return {
          type: 'patch',
          action: 'update_dependencies'
        };
      case 'attack':
        return {
          type: 'block',
          action: 'block_ip'
        };
      case 'access':
        return {
          type: 'restrict',
          action: 'revoke_access'
        };
      default:
        return {
          type: 'unknown',
          action: 'investigate'
        };
    }
  }
  
  async executeStrategy(strategy) {
    switch (strategy.type) {
      case 'patch':
        await this.patchVulnerability();
        break;
      case 'block':
        await this.blockAttacker();
        break;
      case 'restrict':
        await this.restrictAccess();
        break;
      default:
        await this.investigateIssue();
    }
  }
  
  async patchVulnerability() {
    await exec('npm audit fix --force');
  }
  
  async blockAttacker() {
    // Add IP to blacklist
    await redis.sadd('blacklist', attacker.ip);
  }
  
  async restrictAccess() {
    // Revoke user access
    await User.updateOne(
      { _id: user._id },
      { $set: { status: 'blocked' } }
    );
  }
  
  async investigateIssue() {
    // Collect logs
    const logs = await this.collectLogs();
    
    // Analyze patterns
    const patterns = await this.analyzePatterns(logs);
    
    // Generate report
    await this.generateReport(patterns);
  }
}
```

## Database Issues

### 1. Database Monitoring

```javascript
// services/database-monitor.js
class DatabaseMonitor {
  constructor() {
    this.metrics = new Map();
    this.thresholds = {
      connections: 100,
      queries: 1000,
      size: 1000000000
    };
  }
  
  async monitorDatabase() {
    // Monitor connections
    await this.monitorConnections();
    
    // Monitor queries
    await this.monitorQueries();
    
    // Monitor size
    await this.monitorSize();
    
    // Monitor performance
    await this.monitorPerformance();
  }
  
  async monitorConnections() {
    const stats = await mongoose.connection.db.stats();
    
    if (stats.connections > this.thresholds.connections) {
      this.alertDatabaseIssue('connections', stats.connections);
    }
    
    return stats.connections;
  }
  
  async monitorQueries() {
    const stats = await mongoose.connection.db.stats();
    
    if (stats.queries > this.thresholds.queries) {
      this.alertDatabaseIssue('queries', stats.queries);
    }
    
    return stats.queries;
  }
  
  async monitorSize() {
    const stats = await mongoose.connection.db.stats();
    
    if (stats.dataSize > this.thresholds.size) {
      this.alertDatabaseIssue('size', stats.dataSize);
    }
    
    return stats.dataSize;
  }
  
  async monitorPerformance() {
    const stats = await mongoose.connection.db.stats();
    
    if (stats.avgObjSize > 1000000) {
      this.alertDatabaseIssue('performance', stats.avgObjSize);
    }
    
    return stats.avgObjSize;
  }
  
  alertDatabaseIssue(type, value) {
    logger.warn({
      message: 'Database issue detected',
      type: type,
      value: value,
      threshold: this.thresholds[type]
    });
    
    // Send alert (e.g., email, Slack)
  }
}
```

### 2. Database Resolution

```javascript
// services/database-resolver.js
class DatabaseResolver {
  async resolveDatabaseIssue(issue) {
    // Log issue
    logger.warn({
      message: 'Database issue detected',
      issue: issue
    });
    
    // Determine resolution strategy
    const strategy = await this.determineStrategy(issue);
    
    // Execute resolution
    try {
      await this.executeStrategy(strategy);
      
      logger.info({
        message: 'Database issue resolved',
        strategy: strategy
      });
      
      return {
        status: 'success',
        message: 'Resolution completed'
      };
    } catch (error) {
      logger.error({
        message: 'Database resolution failed',
        error: error
      });
      
      throw error;
    }
  }
  
  async determineStrategy(issue) {
    switch (issue.type) {
      case 'connections':
        return {
          type: 'pool',
          action: 'adjust_pool_size'
        };
      case 'queries':
        return {
          type: 'optimize',
          action: 'optimize_queries'
        };
      case 'size':
        return {
          type: 'cleanup',
          action: 'cleanup_data'
        };
      case 'performance':
        return {
          type: 'index',
          action: 'optimize_indexes'
        };
      default:
        return {
          type: 'unknown',
          action: 'investigate'
        };
    }
  }
  
  async executeStrategy(strategy) {
    switch (strategy.type) {
      case 'pool':
        await this.adjustPoolSize();
        break;
      case 'optimize':
        await this.optimizeQueries();
        break;
      case 'cleanup':
        await this.cleanupData();
        break;
      case 'index':
        await this.optimizeIndexes();
        break;
      default:
        await this.investigateIssue();
    }
  }
  
  async adjustPoolSize() {
    await mongoose.connection.close();
    await mongoose.connect(process.env.MONGODB_URI, {
      maxPoolSize: 50
    });
  }
  
  async optimizeQueries() {
    // Analyze slow queries
    const slowQueries = await this.analyzeSlowQueries();
    
    // Optimize each query
    for (const query of slowQueries) {
      await this.optimizeQuery(query);
    }
  }
  
  async cleanupData() {
    // Clean up old data
    const thirtyDaysAgo = new Date();
    thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);
    
    await Message.deleteMany({
      createdAt: { $lt: thirtyDaysAgo }
    });
  }
  
  async optimizeIndexes() {
    // Get all collections
    const collections = await mongoose.connection.db.collections();
    
    // Optimize each collection
    for (const collection of collections) {
      await collection.reIndex();
    }
  }
  
  async investigateIssue() {
    // Collect database stats
    const stats = await mongoose.connection.db.stats();
    
    // Analyze patterns
    const patterns = await this.analyzePatterns(stats);
    
    // Generate report
    await this.generateReport(patterns);
  }
}
```

## Best Practices

1. **Issue Detection**
   - Monitor systems
   - Track metrics
   - Set alerts
   - Log issues

2. **Issue Resolution**
   - Identify root cause
   - Apply fixes
   - Verify resolution
   - Document solution

3. **Performance Optimization**
   - Monitor performance
   - Identify bottlenecks
   - Apply optimizations
   - Measure impact

4. **Security Management**
   - Monitor security
   - Detect threats
   - Apply patches
   - Update systems

5. **Database Management**
   - Monitor database
   - Optimize queries
   - Clean up data
   - Maintain indexes

6. **Error Handling**
   - Track errors
   - Analyze patterns
   - Implement fixes
   - Prevent recurrence

7. **Documentation**
   - Document issues
   - Record solutions
   - Share knowledge
   - Update docs 