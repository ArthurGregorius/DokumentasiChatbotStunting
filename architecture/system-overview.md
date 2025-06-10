# System Overview

## Arsitektur Sistem

Chatbot Identifikasi Stunting dibangun dengan arsitektur modular yang terdiri dari beberapa komponen utama:

### 1. WhatsApp Integration Layer
- Menggunakan library Baileys untuk integrasi WhatsApp
- Menangani koneksi dan autentikasi WhatsApp
- Mengelola pesan masuk dan keluar

### 2. Application Layer
- Node.js sebagai runtime environment
- Express.js untuk routing dan middleware
- State management untuk tracking user session

### 3. Business Logic Layer
- Controllers untuk menangani logika bisnis
- Services untuk operasi data
- Models untuk struktur data

### 4. Data Storage Layer
- Google Sheets sebagai database utama
- Google Apps Script untuk operasi data
- Backup system untuk data penting

## Komponen Utama

### Message Handler
```javascript
// src/handlers/messageHandler.js
async function handleMessage(sock, m, userStateManager) {
  // Menangani pesan masuk
  // Memproses perintah
  // Mengelola state pengguna
}
```

### State Management
```javascript
// src/models/userState.js
class UserState {
  // Mengelola state pengguna
  // Menyimpan progress
  // Memvalidasi alur
}
```

### API Integration
```javascript
// src/services/apiService.js
class ApiService {
  // Komunikasi dengan Google Apps Script
  // Operasi CRUD data
  // Validasi response
}
```

## Alur Data

1. **Input Flow**
   - Pesan masuk dari WhatsApp
   - Validasi input
   - Pemrosesan perintah

2. **Processing Flow**
   - Eksekusi logika bisnis
   - Interaksi dengan database
   - Perhitungan dan analisis

3. **Output Flow**
   - Format response
   - Pengiriman pesan
   - Update state

## Keamanan

### Authentication
- WhatsApp number verification
- Session management
- Rate limiting

### Data Protection
- Enkripsi data sensitif
- Sanitasi input
- Validasi output

### Access Control
- Role-based access
- Permission management
- Audit logging

## Scalability

### Horizontal Scaling
- Stateless architecture
- Load balancing
- Session management

### Vertical Scaling
- Resource optimization
- Caching strategy
- Database indexing

## Monitoring

### System Metrics
- Response time
- Error rate
- Resource usage

### Business Metrics
- User engagement
- Questionnaire completion
- Data accuracy

## Error Handling

### Error Types
- Validation errors
- System errors
- Business logic errors

### Recovery Strategy
- Automatic retry
- Fallback mechanisms
- Error logging

## Integration Points

### External Services
- WhatsApp API
- Google Sheets API
- Google Apps Script

### Internal Services
- Authentication service
- Data processing service
- Notification service

## Deployment Architecture

### Development
- Local development
- Testing environment
- CI/CD pipeline

### Production
- Cloud hosting
- Load balancing
- Backup systems

## Maintenance

### Regular Tasks
- Database backup
- Log rotation
- Performance monitoring

### Emergency Procedures
- System recovery
- Data restoration
- Incident response 