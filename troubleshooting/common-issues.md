# 🔧 Troubleshooting: Common Issues

<div align="center">

![Troubleshooting](https://img.shields.io/badge/Troubleshooting-Common%20Issues-blue)
![Status](https://img.shields.io/badge/Status-Active-green)

</div>

## 📋 Overview

Dokumentasi ini menjelaskan masalah-masalah umum yang mungkin terjadi saat menggunakan Chatbot Identifikasi Stunting dan solusi untuk mengatasinya.

## 🔍 Application Issues

### 1. Server Not Starting
```bash
# Error: Port already in use
Error: listen EADDRINUSE: address already in use :::3000

# Solution:
# 1. Find process using port 3000
lsof -i :3000

# 2. Kill the process
kill -9 <PID>

# 3. Restart server
npm start
```

### 2. Database Connection Issues
```bash
# Error: MongoDB connection failed
MongoServerError: connect ECONNREFUSED 127.0.0.1:27017

# Solution:
# 1. Check MongoDB service
sudo systemctl status mongodb

# 2. Start MongoDB if stopped
sudo systemctl start mongodb

# 3. Check connection string
echo $MONGODB_URI
```

### 3. Redis Connection Issues
```bash
# Error: Redis connection failed
Redis connection to 127.0.0.1:6379 failed

# Solution:
# 1. Check Redis service
sudo systemctl status redis

# 2. Start Redis if stopped
sudo systemctl start redis

# 3. Check connection string
echo $REDIS_URL
```

## 📱 WhatsApp Integration Issues

### 1. Webhook Verification Failed
```bash
# Error: Webhook verification failed
Error: Invalid verify token

# Solution:
# 1. Check verify token in .env
echo $WHATSAPP_VERIFY_TOKEN

# 2. Update verify token in WhatsApp Business API
# 3. Restart application
```

### 2. Message Sending Failed
```bash
# Error: Message sending failed
Error: Invalid phone number

# Solution:
# 1. Check phone number format
# 2. Ensure phone number is registered
# 3. Check WhatsApp Business API status
```

## 🔐 Authentication Issues

### 1. JWT Token Invalid
```bash
# Error: Invalid token
Error: jwt malformed

# Solution:
# 1. Check token format
# 2. Ensure token is not expired
# 3. Verify JWT secret
```

### 2. Rate Limiting
```bash
# Error: Too many requests
Error: Rate limit exceeded

# Solution:
# 1. Wait for rate limit window to reset
# 2. Implement request caching
# 3. Contact support for limit increase
```

## 💾 Database Issues

### 1. Data Not Saving
```bash
# Error: Validation failed
Error: ValidationError: User validation failed

# Solution:
# 1. Check required fields
# 2. Verify data types
# 3. Check validation rules
```

### 2. Slow Queries
```bash
# Issue: Slow database queries

# Solution:
# 1. Add indexes
db.collection.createIndex({ field: 1 })

# 2. Optimize queries
# 3. Monitor query performance
```

## 📊 Performance Issues

### 1. High Memory Usage
```bash
# Issue: High memory consumption

# Solution:
# 1. Check memory usage
pm2 monit

# 2. Optimize code
# 3. Increase server resources
```

### 2. Slow Response Time
```bash
# Issue: Slow API responses

# Solution:
# 1. Implement caching
# 2. Optimize database queries
# 3. Use compression
```

## 🔄 Deployment Issues

### 1. Docker Container Issues
```bash
# Error: Container not starting
Error: Container failed to start

# Solution:
# 1. Check container logs
docker logs <container_id>

# 2. Verify environment variables
# 3. Check resource limits
```

### 2. Nginx Configuration
```bash
# Error: Nginx configuration error
Error: nginx: [emerg] invalid number of arguments

# Solution:
# 1. Check nginx configuration
nginx -t

# 2. Verify SSL certificates
# 3. Check proxy settings
```

## 📝 Logging Issues

### 1. Log Files Not Created
```bash
# Issue: Log files not being created

# Solution:
# 1. Check directory permissions
sudo chown -R app:app /var/log/app

# 2. Verify logger configuration
# 3. Check disk space
```

### 2. Log Rotation
```bash
# Issue: Log files too large

# Solution:
# 1. Configure log rotation
# /etc/logrotate.d/app
/var/log/app/*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 0640 app app
}
```

## 📚 Dokumentasi Terkait

- [FAQ](faq.md)
- [Production Guidelines](../deployment/production.md)
- [Local Setup](../development/local-setup.md)

---

<div align="center">

### 🔗 Navigasi

[⬅️ Kembali ke Production Guidelines](../deployment/production.md) | [Lanjut ke FAQ ➡️](faq.md)

</div> 