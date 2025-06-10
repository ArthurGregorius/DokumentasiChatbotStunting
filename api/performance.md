# API Performance

## Overview

Dokumentasi ini menjelaskan aspek-aspek performa dalam API chatbot, termasuk caching, database optimization, load balancing, dan monitoring performa.

## Caching

### 1. Redis Cache

```javascript
const redisCache = {
  async get(key) {
    try {
      const value = await redis.get(key);
      return value ? JSON.parse(value) : null;
    } catch (error) {
      logger.error('Redis cache get error:', error);
      return null;
    }
  },
  
  async set(key, value, ttl = 3600) {
    try {
      const serialized = JSON.stringify(value);
      await redis.set(key, serialized, 'EX', ttl);
      return true;
    } catch (error) {
      logger.error('Redis cache set error:', error);
      return false;
    }
  },
  
  async delete(key) {
    try {
      await redis.del(key);
      return true;
    } catch (error) {
      logger.error('Redis cache delete error:', error);
      return false;
    }
  }
};
```

### 2. Memory Cache

```javascript
const memoryCache = {
  cache: new Map(),
  
  get(key) {
    const item = this.cache.get(key);
    
    if (!item) {
      return null;
    }
    
    if (item.expiry < Date.now()) {
      this.cache.delete(key);
      return null;
    }
    
    return item.value;
  },
  
  set(key, value, ttl = 600) {
    this.cache.set(key, {
      value,
      expiry: Date.now() + (ttl * 1000)
    });
  },
  
  delete(key) {
    this.cache.delete(key);
  }
};
```

## Database Optimization

### 1. Indexing

```javascript
const dbOptimization = {
  async createIndexes() {
    try {
      // User indexes
      await User.collection.createIndex({ phoneNumber: 1 }, { unique: true });
      await User.collection.createIndex({ role: 1 });
      
      // Child indexes
      await Child.collection.createIndex({ userId: 1 });
      await Child.collection.createIndex({ status: 1 });
      
      // Questionnaire indexes
      await Questionnaire.collection.createIndex({ childId: 1 });
      await Questionnaire.collection.createIndex({ riskLevel: 1 });
      
      // Message indexes
      await Message.collection.createIndex({ userId: 1, createdAt: -1 });
      await Message.collection.createIndex({ status: 1 });
      
      // Session indexes
      await Session.collection.createIndex({ userId: 1 }, { unique: true });
      await Session.collection.createIndex({ expiresAt: 1 });
      
      return true;
    } catch (error) {
      logger.error('Create indexes error:', error);
      throw error;
    }
  }
};
```

### 2. Query Optimization

```javascript
const queryOptimization = {
  async getChildWithQuestionnaires(childId) {
    try {
      return await Child.findById(childId)
        .populate({
          path: 'questionnaires',
          select: 'riskLevel completedAt',
          options: {
            sort: { completedAt: -1 },
            limit: 5
          }
        })
        .lean();
    } catch (error) {
      logger.error('Get child with questionnaires error:', error);
      throw error;
    }
  },
  
  async getRecentMessages(userId, limit = 50) {
    try {
      return await Message.find({ userId })
        .sort({ createdAt: -1 })
        .limit(limit)
        .lean();
    } catch (error) {
      logger.error('Get recent messages error:', error);
      throw error;
    }
  }
};
```

## Load Balancing

### 1. Horizontal Scaling

```javascript
const scalingConfig = {
  cluster: {
    workers: os.cpus().length,
    maxMemory: 1024 * 1024 * 1024, // 1GB
    maxCPU: 80 // 80%
  },
  
  loadBalancer: {
    algorithm: 'round-robin',
    healthCheck: {
      path: '/health',
      interval: 30000, // 30 seconds
      timeout: 5000, // 5 seconds
      unhealthyThreshold: 3,
      healthyThreshold: 2
    }
  }
};
```

### 2. Session Management

```javascript
const sessionManagement = {
  async getSession(userId) {
    try {
      // Try Redis first
      const cachedSession = await redisCache.get(`session:${userId}`);
      if (cachedSession) {
        return cachedSession;
      }
      
      // Fallback to database
      const session = await Session.findOne({ userId });
      if (session) {
        // Cache for future requests
        await redisCache.set(`session:${userId}`, session, 300);
      }
      
      return session;
    } catch (error) {
      logger.error('Get session error:', error);
      throw error;
    }
  }
};
```

## Performance Monitoring

### 1. Metrics Collection

```javascript
const metricsService = {
  async collectMetrics() {
    try {
      const metrics = {
        timestamp: new Date(),
        memory: process.memoryUsage(),
        cpu: process.cpuUsage(),
        uptime: process.uptime(),
        activeConnections: await this.getActiveConnections(),
        requestCount: await this.getRequestCount(),
        errorCount: await this.getErrorCount(),
        responseTime: await this.getAverageResponseTime()
      };
      
      await this.saveMetrics(metrics);
      return metrics;
    } catch (error) {
      logger.error('Collect metrics error:', error);
      throw error;
    }
  },
  
  async getActiveConnections() {
    try {
      return await redis.get('active_connections') || 0;
    } catch (error) {
      logger.error('Get active connections error:', error);
      return 0;
    }
  }
};
```

### 2. Performance Alerts

```javascript
const performanceAlertService = {
  async checkPerformance() {
    try {
      const metrics = await metricsService.collectMetrics();
      
      // Check memory usage
      if (metrics.memory.heapUsed > 1024 * 1024 * 1024) { // 1GB
        await this.createAlert('high_memory_usage', metrics);
      }
      
      // Check CPU usage
      if (metrics.cpu.user > 80) { // 80%
        await this.createAlert('high_cpu_usage', metrics);
      }
      
      // Check response time
      if (metrics.responseTime > 1000) { // 1 second
        await this.createAlert('high_response_time', metrics);
      }
      
      return metrics;
    } catch (error) {
      logger.error('Check performance error:', error);
      throw error;
    }
  }
};
```

## Response Optimization

### 1. Compression

```javascript
const compressionConfig = {
  level: 6, // Compression level (0-9)
  threshold: 1024, // Only compress responses larger than 1KB
  filter: (req, res) => {
    if (req.headers['x-no-compression']) {
      return false;
    }
    return compression.filter(req, res);
  }
};
```

### 2. Response Caching

```javascript
const responseCache = {
  async getCachedResponse(key) {
    try {
      return await redisCache.get(`response:${key}`);
    } catch (error) {
      logger.error('Get cached response error:', error);
      return null;
    }
  },
  
  async cacheResponse(key, response, ttl = 300) {
    try {
      await redisCache.set(`response:${key}`, response, ttl);
      return true;
    } catch (error) {
      logger.error('Cache response error:', error);
      return false;
    }
  }
};
```

## Best Practices

1. **Caching Strategy**
   - Use appropriate cache types
   - Set proper TTL
   - Implement cache invalidation
   - Monitor cache hit rates

2. **Database Optimization**
   - Create proper indexes
   - Optimize queries
   - Use connection pooling
   - Implement data archiving

3. **Load Balancing**
   - Horizontal scaling
   - Session management
   - Health checks
   - Failover handling

4. **Performance Monitoring**
   - Collect metrics
   - Set up alerts
   - Track response times
   - Monitor resource usage

5. **Response Optimization**
   - Enable compression
   - Implement caching
   - Optimize payload size
   - Use proper content types 