# 🔧 Troubleshooting Guide

<div align="center">

![Troubleshooting](https://img.shields.io/badge/Troubleshooting-Guide-blue)
![Status](https://img.shields.io/badge/Status-Active-green)

</div>

## 📋 Overview

Panduan ini berisi solusi untuk masalah umum yang mungkin terjadi saat menggunakan atau mengembangkan Chatbot Identifikasi Stunting.

## 🚨 Common Issues

### 1. WhatsApp Connection Issues
<details>
<summary>🔍 Masalah Koneksi WhatsApp</summary>

#### Gejala
- Bot tidak merespon pesan
- Status koneksi terputus
- Error "Connection failed"

#### Solusi
1. Periksa koneksi internet
2. Verifikasi API key WhatsApp
3. Restart service
4. Periksa log error
</details>

### 2. Google Apps Script Errors
<details>
<summary>🔍 Error Google Apps Script</summary>

#### Gejala
- API calls gagal
- Data tidak tersimpan
- Error "Script execution failed"

#### Solusi
1. Periksa deployment ID
2. Verifikasi credentials
3. Periksa quota limits
4. Update script jika perlu
</details>

### 3. Database Issues
<details>
<summary>🔍 Masalah Database</summary>

#### Gejala
- Data tidak tersimpan
- Query timeout
- Connection errors

#### Solusi
1. Periksa koneksi database
2. Verifikasi credentials
3. Periksa query performance
4. Backup dan restore jika perlu
</details>

## 🔍 Debugging Guide

### 1. Log Analysis
```bash
# View application logs
docker logs chatbot-stunting

# View specific error logs
docker logs chatbot-stunting | grep ERROR

# Follow logs in real-time
docker logs -f chatbot-stunting
```

### 2. Performance Monitoring
```bash
# Check container resources
docker stats chatbot-stunting

# Monitor CPU usage
top -p $(pgrep -f chatbot-stunting)

# Check memory usage
free -m
```

### 3. Network Debugging
```bash
# Check network connectivity
ping your-server

# Test API endpoints
curl -v https://your-api-endpoint

# Check SSL certificate
openssl s_client -connect your-server:443
```

## 📝 Error Codes

| Code | Description | Solution |
|------|-------------|----------|
| W001 | WhatsApp connection failed | Check API key and internet connection |
| G001 | Google Apps Script error | Verify deployment ID and credentials |
| D001 | Database connection error | Check database credentials and connection |
| A001 | Authentication failed | Verify API key and permissions |

## 🛠️ Maintenance Tasks

### Daily Checks
- Monitor error logs
- Check system resources
- Verify API status
- Backup database

### Weekly Tasks
- Review performance metrics
- Update dependencies
- Clean up logs
- Security audit

### Monthly Tasks
- Full system backup
- Performance optimization
- Security updates
- Documentation review

## 📚 Dokumentasi Terkait

- [Common Issues](common-issues.md)
- [FAQ](faq.md)

---

<div align="center">

### 🔗 Navigasi

[⬅️ Kembali ke Deployment](../deployment/README.md) | [Kembali ke Dokumentasi Utama ➡️](../README.md)

</div> 