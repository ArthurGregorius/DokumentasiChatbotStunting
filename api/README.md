# 🔌 API Documentation

<div align="center">

![API](https://img.shields.io/badge/API-Documentation-blue)
![Version](https://img.shields.io/badge/Version-1.0.0-green)
![Status](https://img.shields.io/badge/Status-Active-green)

</div>

## 📋 Overview

Dokumentasi API untuk Chatbot Identifikasi Stunting. API ini digunakan untuk komunikasi antara frontend, WhatsApp, dan backend services.

## 🔑 Authentication

### API Key
Semua request harus menyertakan API key dalam header:
```http
Authorization: Bearer {API_KEY}
```

### Rate Limiting
- 100 requests per minute per API key
- Rate limit headers included in response:
  ```http
  X-RateLimit-Limit: 100
  X-RateLimit-Remaining: 99
  X-RateLimit-Reset: 1616234400
  ```

## 📡 Endpoints

### User Management
<details>
<summary>🔍 User Endpoints</summary>

#### Register User
```http
POST /api/users/register
Content-Type: application/json

{
  "name": "String",
  "phone": "String",
  "address": "String",
  "birthDate": "Date"
}
```

#### Get User
```http
GET /api/users/{phone}
```
</details>

### Child Management
<details>
<summary>🔍 Child Endpoints</summary>

#### Register Child
```http
POST /api/children
Content-Type: application/json

{
  "userId": "String",
  "name": "String",
  "birthDate": "Date",
  "gender": "String",
  "height": "Number",
  "weight": "Number",
  "headCircumference": "Number"
}
```

#### Get Child
```http
GET /api/children/{childId}
```
</details>

### Questionnaire
<details>
<summary>🔍 Questionnaire Endpoints</summary>

#### Submit Questionnaire
```http
POST /api/questionnaires
Content-Type: application/json

{
  "childId": "String",
  "type": "String",
  "answers": [
    {
      "questionId": "String",
      "answer": "String"
    }
  ]
}
```

#### Get Results
```http
GET /api/questionnaires/{childId}
```
</details>

## 📊 Response Format

### Success Response
```json
{
  "status": "success",
  "data": {
    // Response data
  }
}
```

### Error Response
```json
{
  "status": "error",
  "code": "ERROR_CODE",
  "message": "Error description"
}
```

## 🔒 Security

### 1. Authentication
- API Key Authentication
- JWT for Web Interface
- OAuth2 for Google Services

### 2. Authorization
- Role-based Access Control
- Resource-based Permissions
- API Rate Limiting

### 3. Data Security
- End-to-end Encryption
- Data Encryption at Rest
- Secure Communication (HTTPS)

## 📝 Error Codes

| Code | Description | Solution |
|------|-------------|----------|
| AUTH_001 | Invalid API key | Check API key |
| AUTH_002 | Expired token | Refresh token |
| VAL_001 | Invalid input | Check request body |
| DB_001 | Database error | Check logs |

## 🔄 Webhooks

### 1. WhatsApp Webhook
```http
POST /webhook/whatsapp
Content-Type: application/json

{
  "type": "message",
  "data": {
    // Message data
  }
}
```

### 2. Payment Webhook
```http
POST /webhook/payment
Content-Type: application/json

{
  "type": "payment",
  "data": {
    // Payment data
  }
}
```

## 📚 Dokumentasi Terkait

- [Authentication](authentication.md)
- [Endpoints](endpoints.md)
- [Error Handling](error-handling.md)
- [Webhooks](webhooks.md)

---

<div align="center">

### 🔗 Navigasi

[⬅️ Kembali ke Features](../features/README.md) | [Lanjut ke Implementation ➡️](../implementation/README.md)

</div> 