# 🚀 Deployment Guide

<div align="center">

![Deployment](https://img.shields.io/badge/Deployment-Guide-blue)
![Status](https://img.shields.io/badge/Status-Active-green)

</div>

## 📋 Overview

Panduan ini menjelaskan cara melakukan deployment Chatbot Identifikasi Stunting ke lingkungan produksi.

## 🐳 Docker Setup

### Prerequisites
- Docker
- Docker Compose
- Google Cloud Platform account
- WhatsApp Business API access

### Build Docker Image
```bash
docker build -t chatbot-stunting:latest .
```

### Run with Docker Compose
```bash
docker-compose up -d
```

## ⚙️ Environment Configuration

### Required Environment Variables
```env
# Server Configuration
PORT=3000
NODE_ENV=production

# WhatsApp Configuration
WHATSAPP_API_KEY=your_api_key
WHATSAPP_PHONE_NUMBER=your_phone_number

# Google Apps Script
DEPLOYMENT_ID=your_deployment_id
GOOGLE_CREDENTIALS=your_credentials

# Database
DB_HOST=your_db_host
DB_USER=your_db_user
DB_PASSWORD=your_db_password
DB_NAME=your_db_name
```

## 🚀 Production Deployment

### 1. Google Cloud Platform Setup
1. Buat project baru
2. Enable required APIs
3. Setup service account
4. Configure IAM permissions

### 2. Deploy Google Apps Script
1. Update script dengan konfigurasi produksi
2. Deploy sebagai web app
3. Set permissions
4. Copy deployment ID

### 3. Deploy Backend
1. Build Docker image
```bash
docker build -t chatbot-stunting:prod .
```

2. Push ke container registry
```bash
docker tag chatbot-stunting:prod gcr.io/your-project/chatbot-stunting:latest
docker push gcr.io/your-project/chatbot-stunting:latest
```

3. Deploy ke Cloud Run
```bash
gcloud run deploy chatbot-stunting \
  --image gcr.io/your-project/chatbot-stunting:latest \
  --platform managed \
  --region asia-southeast1 \
  --allow-unauthenticated
```

## 📊 Monitoring

### Application Monitoring
- Google Cloud Monitoring
- Custom metrics
- Error tracking
- Performance monitoring

### Log Monitoring
- Google Cloud Logging
- Error logs
- Access logs
- Audit logs

## 🔒 Security

### SSL/TLS
- Enable HTTPS
- Configure SSL certificates
- Regular certificate renewal

### Access Control
- API key authentication
- Rate limiting
- IP whitelisting

## 🔄 Maintenance

### Backup
- Regular database backups
- Configuration backups
- Log archives

### Updates
- Regular dependency updates
- Security patches
- Feature updates

## 📚 Dokumentasi Terkait

- [Docker Setup](docker.md)
- [Environment Variables](environment.md)
- [Production Guidelines](production.md)

---

<div align="center">

### 🔗 Navigasi

[⬅️ Kembali ke Development](../development/README.md) | [Lanjut ke Troubleshooting ➡️](../troubleshooting/README.md)

</div> 