# 🛣️ API Routes

<div align="center">

![API Routes](https://img.shields.io/badge/API-Routes-blue)
![Status](https://img.shields.io/badge/Status-Active-green)

</div>

## 📋 Overview

Dokumentasi ini menjelaskan route-route yang tersedia dalam API Chatbot Identifikasi Stunting, termasuk endpoint, method, parameter, dan response format.

## 👤 User Routes

### Register User
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

### Get User Profile
```http
GET /api/users/{phone}
```

### Update User Profile
```http
PUT /api/users/{phone}
Content-Type: application/json

{
  "name": "String",
  "address": "String"
}
```

## 👶 Child Routes

### Register Child
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

### Get Child Data
```http
GET /api/children/{childId}
```

### Update Child Data
```http
PUT /api/children/{childId}
Content-Type: application/json

{
  "height": "Number",
  "weight": "Number",
  "headCircumference": "Number"
}
```

## 📝 Questionnaire Routes

### Submit Questionnaire
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

### Get Questionnaire Results
```http
GET /api/questionnaires/{childId}
```

### Get Questionnaire History
```http
GET /api/questionnaires/{childId}/history
```

## 💬 Message Routes

### Send Message
```http
POST /api/messages
Content-Type: application/json

{
  "phone": "String",
  "message": "String",
  "type": "String"
}
```

### Get Message History
```http
GET /api/messages/{phone}
Query Parameters:
  - limit: Number (default: 50)
  - offset: Number (default: 0)
```

## 🔄 Session Routes

### Create Session
```http
POST /api/sessions
Content-Type: application/json

{
  "userId": "String",
  "type": "String"
}
```

### End Session
```http
PUT /api/sessions/{sessionId}/end
```

### Get Session Status
```http
GET /api/sessions/{sessionId}
```

## 📱 WhatsApp Routes

### Webhook
```http
POST /webhook/whatsapp
Content-Type: application/json

{
  "verifyToken": "String",
  "phone": "String",
  "type": "String",
  "message": "String"
}
```

### Send Template Message
```http
POST /api/whatsapp/template
Content-Type: application/json

{
  "phone": "String",
  "templateName": "String",
  "parameters": [
    {
      "key": "String",
      "value": "String"
    }
  ]
}
```

## 🔒 Authentication Routes

### Get API Key
```http
POST /api/auth/key
Content-Type: application/json

{
  "username": "String",
  "password": "String"
}
```

### Refresh Token
```http
POST /api/auth/refresh
Content-Type: application/json

{
  "refreshToken": "String"
}
```

## 📊 Analytics Routes

### Get User Statistics
```http
GET /api/analytics/users
Query Parameters:
  - startDate: Date
  - endDate: Date
  - groupBy: String (day, week, month)
```

### Get Questionnaire Statistics
```http
GET /api/analytics/questionnaires
Query Parameters:
  - startDate: Date
  - endDate: Date
  - groupBy: String (day, week, month)
```

## 📚 Dokumentasi Terkait

- [API Overview](README.md)
- [Controllers](controllers.md)
- [Services](services.md)
- [Models](models.md)

---

<div align="center">

### 🔗 Navigasi

[⬅️ Kembali ke Services](services.md) | [Lanjut ke Endpoints ➡️](endpoints.md)

</div> 