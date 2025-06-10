# API Monitoring

## Overview

Dokumentasi ini menjelaskan strategi dan implementasi monitoring untuk API chatbot, termasuk system monitoring, application monitoring, performance monitoring, dan best practices.

## System Monitoring

### 1. Resource Monitoring

```javascript
// services/system-monitor.js
class SystemMonitor {
  constructor() {
    this.metrics = new Map();
    this.thresholds = {
      cpu: 80,
      memory: 80,
      disk: 80,
      network: 1000000
    };
  }
  
  async monitorResources() {
    // Monitor CPU
    await this.monitorCpu();
    
    // Monitor Memory
    await this.monitorMemory();
    
    // Monitor Disk
    await this.monitorDisk();
    
    // Monitor Network
    await this.monitorNetwork();
  }
  
  async monitorCpu() {
    const usage = await this.getCpuUsage();
    
    if (usage > this.thresholds.cpu) {
      this.alertResourceIssue('cpu', usage);
    }
    
    this.metrics.set('cpu', usage);
  }
  
  async monitorMemory() {
    const usage = await this.getMemoryUsage();
    
    if (usage > this.thresholds.memory) {
      this.alertResourceIssue('memory', usage);
    }
    
    this.metrics.set('memory', usage);
  }
  
  async monitorDisk() {
    const usage = await this.getDiskUsage();
    
    if (usage > this.thresholds.disk) {
      this.alertResourceIssue('disk', usage);
    }
    
    this.metrics.set('disk', usage);
  }
  
  async monitorNetwork() {
    const usage = await this.getNetworkUsage();
    
    if (usage > this.thresholds.network) {
      this.alertResourceIssue('network', usage);
    }
    
    this.metrics.set('network', usage);
  }
  
  alertResourceIssue(type, value) {
    logger.warn({
      message: 'Resource issue detected',
      type: type,
      value: value,
      threshold: this.thresholds[type]
    });
    
    // Send alert (e.g., email, Slack)
  }
}
```

### 2. Process Monitoring

```javascript
// services/process-monitor.js
class ProcessMonitor {
  constructor() {
    this.processes = new Map();
    this.thresholds = {
      uptime: 86400,
      memory: 1000000000,
      cpu: 80
    };
  }
  
  async monitorProcesses() {
    // Monitor Node.js process
    await this.monitorNodeProcess();
    
    // Monitor MongoDB process
    await this.monitorMongoProcess();
    
    // Monitor Redis process
    await this.monitorRedisProcess();
  }
  
  async monitorNodeProcess() {
    const process = {
      pid: process.pid,
      uptime: process.uptime(),
      memory: process.memoryUsage(),
      cpu: process.cpuUsage()
    };
    
    if (process.uptime > this.thresholds.uptime) {
      this.alertProcessIssue('node', 'uptime', process.uptime);
    }
    
    if (process.memory.heapUsed > this.thresholds.memory) {
      this.alertProcessIssue('node', 'memory', process.memory.heapUsed);
    }
    
    if (process.cpu.user > this.thresholds.cpu) {
      this.alertProcessIssue('node', 'cpu', process.cpu.user);
    }
    
    this.processes.set('node', process);
  }
  
  async monitorMongoProcess() {
    const process = await this.getMongoProcess();
    
    if (process.uptime > this.thresholds.uptime) {
      this.alertProcessIssue('mongo', 'uptime', process.uptime);
    }
    
    if (process.memory > this.thresholds.memory) {
      this.alertProcessIssue('mongo', 'memory', process.memory);
    }
    
    if (process.cpu > this.thresholds.cpu) {
      this.alertProcessIssue('mongo', 'cpu', process.cpu);
    }
    
    this.processes.set('mongo', process);
  }
  
  async monitorRedisProcess() {
    const process = await this.getRedisProcess();
    
    if (process.uptime > this.thresholds.uptime) {
      this.alertProcessIssue('redis', 'uptime', process.uptime);
    }
    
    if (process.memory > this.thresholds.memory) {
      this.alertProcessIssue('redis', 'memory', process.memory);
    }
    
    if (process.cpu > this.thresholds.cpu) {
      this.alertProcessIssue('redis', 'cpu', process.cpu);
    }
    
    this.processes.set('redis', process);
  }
  
  alertProcessIssue(process, type, value) {
    logger.warn({
      message: 'Process issue detected',
      process: process,
      type: type,
      value: value,
      threshold: this.thresholds[type]
    });
    
    // Send alert (e.g., email, Slack)
  }
}
```

## Application Monitoring

### 1. Request Monitoring

```javascript
// middleware/request-monitor.js
const requestMonitor = (req, res, next) => {
  const start = Date.now();
  
  // Log request
  logger.info({
    method: req.method,
    url: req.url,
    headers: req.headers,
    body: req.body,
    query: req.query,
    params: req.params,
    timestamp: new Date()
  });
  
  // Capture response
  const originalSend = res.send;
  res.send = function(body) {
    const duration = Date.now() - start;
    
    // Log response
    logger.info({
      status: res.statusCode,
      duration: duration,
      body: body,
      timestamp: new Date()
    });
    
    return originalSend.call(this, body);
  };
  
  next();
};
```

### 2. Error Monitoring

```javascript
// services/error-monitor.js
class ErrorMonitor {
  constructor() {
    this.errors = new Map();
    this.thresholds = {
      count: 10,
      timeWindow: 3600000
    };
  }
  
  async monitorErrors() {
    // Monitor application errors
    await this.monitorApplicationErrors();
    
    // Monitor database errors
    await this.monitorDatabaseErrors();
    
    // Monitor external service errors
    await this.monitorExternalErrors();
  }
  
  async monitorApplicationErrors() {
    const errors = await this.getApplicationErrors();
    
    for (const error of errors) {
      this.trackError('application', error);
    }
  }
  
  async monitorDatabaseErrors() {
    const errors = await this.getDatabaseErrors();
    
    for (const error of errors) {
      this.trackError('database', error);
    }
  }
  
  async monitorExternalErrors() {
    const errors = await this.getExternalErrors();
    
    for (const error of errors) {
      this.trackError('external', error);
    }
  }
  
  trackError(type, error) {
    if (!this.errors.has(type)) {
      this.errors.set(type, []);
    }
    
    const errors = this.errors.get(type);
    errors.push({
      error: error,
      timestamp: new Date()
    });
    
    // Keep only errors within time window
    const cutoff = Date.now() - this.thresholds.timeWindow;
    while (errors[0]?.timestamp < cutoff) {
      errors.shift();
    }
    
    // Check threshold
    if (errors.length >= this.thresholds.count) {
      this.alertErrorIssue(type, errors);
    }
  }
  
  alertErrorIssue(type, errors) {
    logger.warn({
      message: 'Error issue detected',
      type: type,
      count: errors.length,
      errors: errors
    });
    
    // Send alert (e.g., email, Slack)
  }
}
```

## Performance Monitoring

### 1. Response Time Monitoring

```javascript
// services/performance-monitor.js
class PerformanceMonitor {
  constructor() {
    this.metrics = new Map();
    this.thresholds = {
      responseTime: 1000,
      throughput: 1000,
      errorRate: 0.01
    };
  }
  
  async monitorPerformance() {
    // Monitor response time
    await this.monitorResponseTime();
    
    // Monitor throughput
    await this.monitorThroughput();
    
    // Monitor error rate
    await this.monitorErrorRate();
  }
  
  async monitorResponseTime() {
    const responseTimes = await this.getResponseTimes();
    const avgResponseTime = this.calculateAverage(responseTimes);
    
    if (avgResponseTime > this.thresholds.responseTime) {
      this.alertPerformanceIssue('responseTime', avgResponseTime);
    }
    
    this.metrics.set('responseTime', avgResponseTime);
  }
  
  async monitorThroughput() {
    const requests = await this.getRequests();
    const throughput = this.calculateThroughput(requests);
    
    if (throughput > this.thresholds.throughput) {
      this.alertPerformanceIssue('throughput', throughput);
    }
    
    this.metrics.set('throughput', throughput);
  }
  
  async monitorErrorRate() {
    const errors = await this.getErrors();
    const errorRate = this.calculateErrorRate(errors);
    
    if (errorRate > this.thresholds.errorRate) {
      this.alertPerformanceIssue('errorRate', errorRate);
    }
    
    this.metrics.set('errorRate', errorRate);
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

### 2. Resource Usage Monitoring

```javascript
// services/resource-monitor.js
class ResourceMonitor {
  constructor() {
    this.metrics = new Map();
    this.thresholds = {
      memory: 0.8,
      cpu: 0.8,
      disk: 0.8,
      network: 1000000
    };
  }
  
  async monitorResources() {
    // Monitor memory usage
    await this.monitorMemory();
    
    // Monitor CPU usage
    await this.monitorCpu();
    
    // Monitor disk usage
    await this.monitorDisk();
    
    // Monitor network usage
    await this.monitorNetwork();
  }
  
  async monitorMemory() {
    const usage = process.memoryUsage();
    const heapUsed = usage.heapUsed / usage.heapTotal;
    
    if (heapUsed > this.thresholds.memory) {
      this.alertResourceIssue('memory', heapUsed);
    }
    
    this.metrics.set('memory', heapUsed);
  }
  
  async monitorCpu() {
    const usage = process.cpuUsage();
    const totalUsage = (usage.user + usage.system) / 1000000;
    
    if (totalUsage > this.thresholds.cpu) {
      this.alertResourceIssue('cpu', totalUsage);
    }
    
    this.metrics.set('cpu', totalUsage);
  }
  
  async monitorDisk() {
    const usage = await this.getDiskUsage();
    
    if (usage > this.thresholds.disk) {
      this.alertResourceIssue('disk', usage);
    }
    
    this.metrics.set('disk', usage);
  }
  
  async monitorNetwork() {
    const usage = await this.getNetworkUsage();
    
    if (usage > this.thresholds.network) {
      this.alertResourceIssue('network', usage);
    }
    
    this.metrics.set('network', usage);
  }
  
  alertResourceIssue(type, value) {
    logger.warn({
      message: 'Resource issue detected',
      type: type,
      value: value,
      threshold: this.thresholds[type]
    });
    
    // Send alert (e.g., email, Slack)
  }
}
```

## Security Monitoring

### 1. Access Monitoring

```javascript
// services/security-monitor.js
class SecurityMonitor {
  constructor() {
    this.metrics = new Map();
    this.thresholds = {
      failedLogins: 5,
      suspiciousAccess: 10,
      blockedIPs: 100
    };
  }
  
  async monitorSecurity() {
    // Monitor failed logins
    await this.monitorFailedLogins();
    
    // Monitor suspicious access
    await this.monitorSuspiciousAccess();
    
    // Monitor blocked IPs
    await this.monitorBlockedIPs();
  }
  
  async monitorFailedLogins() {
    const failedLogins = await this.getFailedLogins();
    
    for (const login of failedLogins) {
      if (login.attempts >= this.thresholds.failedLogins) {
        this.alertSecurityIssue('failedLogins', login);
      }
    }
    
    this.metrics.set('failedLogins', failedLogins.length);
  }
  
  async monitorSuspiciousAccess() {
    const suspiciousAccess = await this.getSuspiciousAccess();
    
    if (suspiciousAccess.length >= this.thresholds.suspiciousAccess) {
      this.alertSecurityIssue('suspiciousAccess', suspiciousAccess);
    }
    
    this.metrics.set('suspiciousAccess', suspiciousAccess.length);
  }
  
  async monitorBlockedIPs() {
    const blockedIPs = await this.getBlockedIPs();
    
    if (blockedIPs.length >= this.thresholds.blockedIPs) {
      this.alertSecurityIssue('blockedIPs', blockedIPs);
    }
    
    this.metrics.set('blockedIPs', blockedIPs.length);
  }
  
  alertSecurityIssue(type, data) {
    logger.warn({
      message: 'Security issue detected',
      type: type,
      data: data,
      threshold: this.thresholds[type]
    });
    
    // Send alert (e.g., email, Slack)
  }
}
```

### 2. Vulnerability Monitoring

```javascript
// services/vulnerability-monitor.js
class VulnerabilityMonitor {
  constructor() {
    this.vulnerabilities = new Map();
    this.thresholds = {
      high: 0,
      medium: 5,
      low: 10
    };
  }
  
  async monitorVulnerabilities() {
    // Check npm vulnerabilities
    await this.checkNpmVulnerabilities();
    
    // Check system vulnerabilities
    await this.checkSystemVulnerabilities();
    
    // Check application vulnerabilities
    await this.checkApplicationVulnerabilities();
  }
  
  async checkNpmVulnerabilities() {
    const { stdout } = await exec('npm audit --json');
    const audit = JSON.parse(stdout);
    
    for (const vuln of audit.vulnerabilities) {
      this.trackVulnerability('npm', vuln);
    }
  }
  
  async checkSystemVulnerabilities() {
    const vulnerabilities = await this.getSystemVulnerabilities();
    
    for (const vuln of vulnerabilities) {
      this.trackVulnerability('system', vuln);
    }
  }
  
  async checkApplicationVulnerabilities() {
    const vulnerabilities = await this.getApplicationVulnerabilities();
    
    for (const vuln of vulnerabilities) {
      this.trackVulnerability('application', vuln);
    }
  }
  
  trackVulnerability(type, vulnerability) {
    if (!this.vulnerabilities.has(type)) {
      this.vulnerabilities.set(type, []);
    }
    
    const vulnerabilities = this.vulnerabilities.get(type);
    vulnerabilities.push({
      vulnerability: vulnerability,
      timestamp: new Date()
    });
    
    // Check threshold
    const highSeverity = vulnerabilities.filter(v => v.vulnerability.severity === 'high');
    if (highSeverity.length > this.thresholds.high) {
      this.alertVulnerabilityIssue(type, 'high', highSeverity);
    }
    
    const mediumSeverity = vulnerabilities.filter(v => v.vulnerability.severity === 'medium');
    if (mediumSeverity.length > this.thresholds.medium) {
      this.alertVulnerabilityIssue(type, 'medium', mediumSeverity);
    }
    
    const lowSeverity = vulnerabilities.filter(v => v.vulnerability.severity === 'low');
    if (lowSeverity.length > this.thresholds.low) {
      this.alertVulnerabilityIssue(type, 'low', lowSeverity);
    }
  }
  
  alertVulnerabilityIssue(type, severity, vulnerabilities) {
    logger.warn({
      message: 'Vulnerability issue detected',
      type: type,
      severity: severity,
      vulnerabilities: vulnerabilities,
      threshold: this.thresholds[severity]
    });
    
    // Send alert (e.g., email, Slack)
  }
}
```

## Database Monitoring

### 1. Performance Monitoring

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
  }
  
  async monitorConnections() {
    const stats = await mongoose.connection.db.stats();
    
    if (stats.connections > this.thresholds.connections) {
      this.alertDatabaseIssue('connections', stats.connections);
    }
    
    this.metrics.set('connections', stats.connections);
  }
  
  async monitorQueries() {
    const stats = await mongoose.connection.db.stats();
    
    if (stats.queries > this.thresholds.queries) {
      this.alertDatabaseIssue('queries', stats.queries);
    }
    
    this.metrics.set('queries', stats.queries);
  }
  
  async monitorSize() {
    const stats = await mongoose.connection.db.stats();
    
    if (stats.dataSize > this.thresholds.size) {
      this.alertDatabaseIssue('size', stats.dataSize);
    }
    
    this.metrics.set('size', stats.dataSize);
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

### 2. Health Monitoring

```javascript
// services/database-health-monitor.js
class DatabaseHealthMonitor {
  constructor() {
    this.health = new Map();
    this.thresholds = {
      replicationLag: 60,
      indexSize: 1000000000,
      cacheHitRatio: 0.8
    };
  }
  
  async monitorHealth() {
    // Monitor replication
    await this.monitorReplication();
    
    // Monitor indexes
    await this.monitorIndexes();
    
    // Monitor cache
    await this.monitorCache();
  }
  
  async monitorReplication() {
    const status = await mongoose.connection.db.admin().replSetGetStatus();
    
    for (const member of status.members) {
      if (member.lag > this.thresholds.replicationLag) {
        this.alertHealthIssue('replication', member);
      }
    }
    
    this.health.set('replication', status);
  }
  
  async monitorIndexes() {
    const stats = await mongoose.connection.db.stats();
    
    if (stats.indexSize > this.thresholds.indexSize) {
      this.alertHealthIssue('indexes', stats);
    }
    
    this.health.set('indexes', stats);
  }
  
  async monitorCache() {
    const stats = await mongoose.connection.db.stats();
    const hitRatio = stats.cacheHitRatio;
    
    if (hitRatio < this.thresholds.cacheHitRatio) {
      this.alertHealthIssue('cache', stats);
    }
    
    this.health.set('cache', stats);
  }
  
  alertHealthIssue(type, data) {
    logger.warn({
      message: 'Database health issue detected',
      type: type,
      data: data,
      threshold: this.thresholds[type]
    });
    
    // Send alert (e.g., email, Slack)
  }
}
```

## Log Monitoring

### 1. Log Analysis

```javascript
// services/log-monitor.js
class LogMonitor {
  constructor() {
    this.logs = new Map();
    this.thresholds = {
      errorCount: 10,
      warningCount: 50,
      logSize: 1000000000
    };
  }
  
  async monitorLogs() {
    // Monitor error logs
    await this.monitorErrorLogs();
    
    // Monitor warning logs
    await this.monitorWarningLogs();
    
    // Monitor log size
    await this.monitorLogSize();
  }
  
  async monitorErrorLogs() {
    const errors = await this.getErrorLogs();
    
    if (errors.length >= this.thresholds.errorCount) {
      this.alertLogIssue('errors', errors);
    }
    
    this.logs.set('errors', errors);
  }
  
  async monitorWarningLogs() {
    const warnings = await this.getWarningLogs();
    
    if (warnings.length >= this.thresholds.warningCount) {
      this.alertLogIssue('warnings', warnings);
    }
    
    this.logs.set('warnings', warnings);
  }
  
  async monitorLogSize() {
    const size = await this.getLogSize();
    
    if (size > this.thresholds.logSize) {
      this.alertLogIssue('size', size);
    }
    
    this.logs.set('size', size);
  }
  
  alertLogIssue(type, data) {
    logger.warn({
      message: 'Log issue detected',
      type: type,
      data: data,
      threshold: this.thresholds[type]
    });
    
    // Send alert (e.g., email, Slack)
  }
}
```

### 2. Log Rotation

```javascript
// services/log-rotator.js
class LogRotator {
  constructor() {
    this.config = {
      maxSize: 1000000000,
      maxFiles: 10,
      interval: 86400000
    };
  }
  
  async rotateLogs() {
    // Check log size
    await this.checkLogSize();
    
    // Archive old logs
    await this.archiveOldLogs();
    
    // Clean up archived logs
    await this.cleanupArchivedLogs();
  }
  
  async checkLogSize() {
    const size = await this.getLogSize();
    
    if (size > this.config.maxSize) {
      await this.rotateCurrentLog();
    }
  }
  
  async archiveOldLogs() {
    const oldLogs = await this.getOldLogs();
    
    for (const log of oldLogs) {
      await this.archiveLog(log);
    }
  }
  
  async cleanupArchivedLogs() {
    const archivedLogs = await this.getArchivedLogs();
    
    if (archivedLogs.length > this.config.maxFiles) {
      const toDelete = archivedLogs.slice(0, archivedLogs.length - this.config.maxFiles);
      
      for (const log of toDelete) {
        await this.deleteLog(log);
      }
    }
  }
  
  async rotateCurrentLog() {
    const timestamp = new Date().toISOString().replace(/[:.]/g, '-');
    const newLogPath = `logs/app-${timestamp}.log`;
    
    await fs.rename('logs/app.log', newLogPath);
    await this.createNewLog();
  }
  
  async archiveLog(log) {
    const timestamp = new Date().toISOString().replace(/[:.]/g, '-');
    const archivePath = `logs/archive/app-${timestamp}.log`;
    
    await fs.rename(log.path, archivePath);
  }
  
  async deleteLog(log) {
    await fs.unlink(log.path);
  }
  
  async createNewLog() {
    await fs.writeFile('logs/app.log', '');
  }
}
```

## Best Practices

1. **System Monitoring**
   - Monitor resources
   - Track processes
   - Set thresholds
   - Alert issues

2. **Application Monitoring**
   - Monitor requests
   - Track errors
   - Measure performance
   - Log activities

3. **Performance Monitoring**
   - Monitor response time
   - Track throughput
   - Measure resource usage
   - Set benchmarks

4. **Security Monitoring**
   - Monitor access
   - Track vulnerabilities
   - Detect threats
   - Alert issues

5. **Database Monitoring**
   - Monitor performance
   - Track health
   - Measure usage
   - Alert issues

6. **Log Monitoring**
   - Analyze logs
   - Rotate logs
   - Archive logs
   - Clean up logs

7. **Alert Management**
   - Set thresholds
   - Configure alerts
   - Escalate issues
   - Track resolutions 