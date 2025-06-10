# Endpoints

## 📋 Overview

Dokumentasi ini menjelaskan endpoint-endpoint yang tersedia dalam API Chatbot Identifikasi Stunting, termasuk request/response format, parameter, dan contoh penggunaan.

## 🌐 Base URL

```bash
https://script.google.com/macros/s/{DEPLOYMENT_ID}/exec
```

## 🔐 Authentication

Semua request harus menyertakan API key dalam header:

```http
Authorization: Bearer {API_KEY}
```

## 📡 Endpoints

### 👤 User Management

#### Register User

<details>

<summary>📝 Register User Endpoint</summary>

```http
POST /api/users/register
```

**Request:**

```json
{
  "name": "John Doe",
  "phone": "+6281234567890",
  "address": "Jl. Example No. 123",
  "birthDate": "1990-01-01"
}
```

**Response:**

```json
{
  "status": "success",
  "data": {
    "id": "user_123",
    "name": "John Doe",
    "phone": "+6281234567890",
    "address": "Jl. Example No. 123",
    "birthDate": "1990-01-01T00:00:00.000Z",
    "createdAt": "2024-01-01T00:00:00.000Z"
  }
}
```

</details>

#### Get User Profile

<details>

<summary>🔍 Get User Endpoint</summary>

```http
GET /api/users/{phone}
```

**Response:**

```json
{
  "status": "success",
  "data": {
    "id": "user_123",
    "name": "John Doe",
    "phone": "+6281234567890",
    "address": "Jl. Example No. 123",
    "birthDate": "1990-01-01T00:00:00.000Z",
    "createdAt": "2024-01-01T00:00:00.000Z",
    "children": [
      {
        "id": "child_123",
        "name": "Jane Doe",
        "birthDate": "2020-01-01T00:00:00.000Z"
      }
    ]
  }
}
```

</details>

### 👶 Child Management

#### Register Child

<details>

<summary>📝 Register Child Endpoint</summary>

```http
POST /api/children
```

**Request:**

```json
{
  "userId": "user_123",
  "name": "Jane Doe",
  "birthDate": "2020-01-01",
  "gender": "female",
  "height": 75,
  "weight": 8.5,
  "headCircumference": 45
}
```

**Response:**

```json
{
  "status": "success",
  "data": {
    "id": "child_123",
    "userId": "user_123",
    "name": "Jane Doe",
    "birthDate": "2020-01-01T00:00:00.000Z",
    "gender": "female",
    "height": 75,
    "weight": 8.5,
    "headCircumference": 45,
    "createdAt": "2024-01-01T00:00:00.000Z"
  }
}
```

</details>

#### Get Child Data

<details>

<summary>�� Get Child Endpoint</summary>

```http
GET /api/children/{childId}
```

**Response:**

```json
{
  "status": "success",
  "data": {
    "id": "child_123",
    "userId": "user_123",
    "name": "Jane Doe",
    "birthDate": "2020-01-01T00:00:00.000Z",
    "gender": "female",
    "height": 75,
    "weight": 8.5,
    "headCircumference": 45,
    "createdAt": "2024-01-01T00:00:00.000Z",
    "questionnaires": [
      {
        "id": "questionnaire_123",
        "type": "initial",
        "score": 7,
        "riskLevel": "medium",
        "createdAt": "2024-01-01T00:00:00.000Z"
      }
    ]
  }
}
```

</details>

### 📊 Questionnaire

#### Submit Questionnaire

<details>

<summary>📝 Submit Questionnaire Endpoint</summary>

```http
POST /api/questionnaires
```

**Request:**

```json
{
  "childId": "child_123",
  "type": "initial",
  "answers": [
    {
      "questionId": "q1",
      "answer": "yes"
    },
    {
      "questionId": "q2",
      "answer": "no"
    }
  ]
}
```

**Response:**

```json
{
  "status": "success",
  "data": {
    "id": "questionnaire_123",
    "childId": "child_123",
    "type": "initial",
    "score": 7,
    "riskLevel": "medium",
    "recommendations": [
      "Regular growth monitoring",
      "Nutritional assessment",
      "Follow-up in 3 months"
    ],
    "createdAt": "2024-01-01T00:00:00.000Z"
  }
}
```

</details>

#### Get Questionnaire Results

<details>

<summary>🔍 Get Questionnaire Results Endpoint</summary>

```http
GET /api/questionnaires/{childId}
```

**Response:**

```json
{
  "status": "success",
  "data": {
    "childId": "child_123",
    "questionnaires": [
      {
        "id": "questionnaire_123",
        "type": "initial",
        "score": 7,
        "riskLevel": "medium",
        "recommendations": [
          "Regular growth monitoring",
          "Nutritional assessment",
          "Follow-up in 3 months"
        ],
        "createdAt": "2024-01-01T00:00:00.000Z"
      }
    ],
    "statistics": {
      "total": 1,
      "highRisk": 0,
      "mediumRisk": 1,
      "lowRisk": 0,
      "averageScore": 7
    }
  }
}
```

</details>

## 💬 Message Endpoints

### Send Message

<details>

<summary>📝 Send Message Endpoint</summary>

```http
POST /api/messages
```

**Request:**

```json
{
  "phone": "+6281234567890",
  "message": "Hello, how can I help you?",
  "type": "text"
}
```

**Response:**

```json
{
  "status": "success",
  "data": {
    "id": "message_123",
    "phone": "+6281234567890",
    "type": "text",
    "content": "Hello, how can I help you?",
    "direction": "outgoing",
    "status": "delivered",
    "createdAt": "2024-01-01T00:00:00.000Z"
  }
}
```

</details>

### Get Message History

<details>

<summary>🔍 Get Message History Endpoint</summary>

```http
GET /api/messages/{phone}?limit=50&offset=0
```

**Response:**

```json
{
  "status": "success",
  "data": {
    "messages": [
      {
        "id": "message_123",
        "phone": "+6281234567890",
        "type": "text",
        "content": "Hello, how can I help you?",
        "direction": "outgoing",
        "status": "delivered",
        "createdAt": "2024-01-01T00:00:00.000Z"
      }
    ],
    "pagination": {
      "total": 1,
      "limit": 50,
      "offset": 0
    }
  }
}
```

</details>

## 🔄 Session Endpoints

### Create Session

<details>

<summary>📝 Create Session Endpoint</summary>

```http
POST /api/sessions
```

**Request:**

```json
{
  "userId": "user_123",
  "type": "registration"
}
```

**Response:**

```json
{
  "status": "success",
  "data": {
    "id": "session_123",
    "userId": "user_123",
    "type": "registration",
    "state": "initial",
    "expiresAt": "2024-01-02T00:00:00.000Z",
    "createdAt": "2024-01-01T00:00:00.000Z"
  }
}
```

</details>

### End Session

<details>

<summary>📝 End Session Endpoint</summary>

```http
PUT /api/sessions/{sessionId}/end
```

**Response:**

```json
{
  "status": "success",
  "data": {
    "id": "session_123",
    "userId": "user_123",
    "type": "registration",
    "state": "ended",
    "expiresAt": "2024-01-02T00:00:00.000Z",
    "createdAt": "2024-01-01T00:00:00.000Z"
  }
}
```

</details>

## 📱 WhatsApp Endpoints

### Webhook

<details>

<summary>📝 Webhook Endpoint</summary>

```http
POST /webhook/whatsapp
```

**Request:**

```json
{
  "verifyToken": "your_webhook_token",
  "phone": "+6281234567890",
  "type": "text",
  "message": "Hello"
}
```

**Response:**

```json
{
  "status": "success",
  "data": {
    "id": "message_123",
    "phone": "+6281234567890",
    "type": "text",
    "content": "Hello",
    "direction": "incoming",
    "status": "received",
    "createdAt": "2024-01-01T00:00:00.000Z"
  }
}
```

</details>

### Send Template Message

<details>

<summary>📝 Send Template Message Endpoint</summary>

```http
POST /api/whatsapp/template
```

**Request:**

```json
{
  "phone": "+6281234567890",
  "templateName": "assessment_reminder",
  "parameters": [
    {
      "key": "name",
      "value": "John Doe"
    },
    {
      "key": "date",
      "value": "2024-01-15"
    }
  ]
}
```

**Response:**

```json
{
  "status": "success",
  "data": {
    "id": "message_123",
    "phone": "+6281234567890",
    "type": "template",
    "templateName": "assessment_reminder",
    "status": "delivered",
    "createdAt": "2024-01-01T00:00:00.000Z"
  }
}
```

</details>

## 📚 Dokumentasi Terkait

* [API Overview](./)
* [Routes](routes.md)
* [Controllers](controllers.md)
* [Services](services.md)

***

#### 🔗 Navigasi

[⬅️ Kembali ke Routes](routes.md) | [Lanjut ke Error Handling ➡️](error-handling.md)

## Rate Limiting

* 100 requests per minute per API key
*   Rate limit headers included in response:

    ```http
    X-RateLimit-Limit: 100
    X-RateLimit-Remaining: 99
    X-RateLimit-Reset: 1616234400
    ```

## Data Validation

### User Data

* Nama: 3-100 karakter
* Nomor telepon: Format Indonesia (62xxx)
* Alamat: Minimal 10 karakter
* Tanggal lahir: Format valid
* Jenis kelamin: L/P

### Child Data

* Nama: 3-100 karakter
* Tanggal lahir: Format valid
* Jenis kelamin: L/P
* Tinggi badan: 30-200 cm
* Berat badan: 1-100 kg
* Lingkar kepala: 20-60 cm

### Questionnaire Data

* Pertanyaan ID: Format valid
* Jawaban: Sesuai tipe pertanyaan
* Tanggal: Format valid

## Best Practices

### Request

1. Gunakan HTTPS
2. Sertakan API key
3. Validasi data sebelum kirim
4. Gunakan proper content type

### Response

1. Handle semua error
2. Validasi response
3. Cache jika perlu
4. Log untuk debugging

## Testing

### Postman Collection

```json
{
  "info": {
    "name": "Chatbot Stunting API",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "item": [
    {
      "name": "User Management",
      "item": [
        {
          "name": "Register User",
          "request": {
            "method": "POST",
            "url": "{{base_url}}/api/users/register"
          }
        }
      ]
    }
  ]
}
```

### Test Cases

1. Valid request
2. Invalid data
3. Missing required fields
4. Rate limit
5. Authentication
