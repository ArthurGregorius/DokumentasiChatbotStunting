# API Documentation

## Overview

Dokumentasi ini menjelaskan penggunaan API chatbot, termasuk endpoint, autentikasi, request/response format, dan best practices.

## Authentication

### JWT Authentication

```javascript
// Example JWT token
const token = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOiIxMjM0NTY3ODkwIiwiaWF0IjoxNTE2MjM5MDIyfQ';

// Headers
{
  'Authorization': 'Bearer ' + token
}
```

### API Key Authentication

```javascript
// Example API key
const apiKey = 'sk_test_1234567890';

// Headers
{
  'X-API-Key': apiKey
}
```

### WhatsApp Authentication

```javascript
// Example phone number
const phone = '6281234567890';

// Headers
{
  'X-WhatsApp-Phone': phone
}
```

## Endpoints

### User Endpoints

#### Register User

```http
POST /api/users/register
Content-Type: application/json

{
  "name": "John Doe",
  "phone": "6281234567890",
  "password": "password123"
}
```

Response:
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "1234567890",
      "name": "John Doe",
      "phone": "6281234567890",
      "createdAt": "2024-01-01T00:00:00.000Z"
    },
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

#### Get User Profile

```http
GET /api/users/profile
Authorization: Bearer <token>
```

Response:
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "1234567890",
      "name": "John Doe",
      "phone": "6281234567890",
      "createdAt": "2024-01-01T00:00:00.000Z"
    }
  }
}
```

#### Update User Profile

```http
PUT /api/users/profile
Authorization: Bearer <token>
Content-Type: application/json

{
  "name": "John Doe Updated"
}
```

Response:
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "1234567890",
      "name": "John Doe Updated",
      "phone": "6281234567890",
      "updatedAt": "2024-01-01T00:00:00.000Z"
    }
  }
}
```

### Child Endpoints

#### Register Child

```http
POST /api/children
Authorization: Bearer <token>
Content-Type: application/json

{
  "name": "Jane Doe",
  "birthDate": "2020-01-01",
  "gender": "female"
}
```

Response:
```json
{
  "success": true,
  "data": {
    "child": {
      "id": "1234567890",
      "name": "Jane Doe",
      "birthDate": "2020-01-01",
      "gender": "female",
      "createdAt": "2024-01-01T00:00:00.000Z"
    }
  }
}
```

#### Get Child Profile

```http
GET /api/children/:childId
Authorization: Bearer <token>
```

Response:
```json
{
  "success": true,
  "data": {
    "child": {
      "id": "1234567890",
      "name": "Jane Doe",
      "birthDate": "2020-01-01",
      "gender": "female",
      "createdAt": "2024-01-01T00:00:00.000Z"
    }
  }
}
```

#### Update Child Profile

```http
PUT /api/children/:childId
Authorization: Bearer <token>
Content-Type: application/json

{
  "name": "Jane Doe Updated"
}
```

Response:
```json
{
  "success": true,
  "data": {
    "child": {
      "id": "1234567890",
      "name": "Jane Doe Updated",
      "birthDate": "2020-01-01",
      "gender": "female",
      "updatedAt": "2024-01-01T00:00:00.000Z"
    }
  }
}
```

### Questionnaire Endpoints

#### Submit Questionnaire

```http
POST /api/questionnaires
Authorization: Bearer <token>
Content-Type: application/json

{
  "childId": "1234567890",
  "answers": [
    {
      "questionId": "1",
      "answer": "yes"
    },
    {
      "questionId": "2",
      "answer": "no"
    }
  ]
}
```

Response:
```json
{
  "success": true,
  "data": {
    "questionnaire": {
      "id": "1234567890",
      "childId": "1234567890",
      "answers": [
        {
          "questionId": "1",
          "answer": "yes"
        },
        {
          "questionId": "2",
          "answer": "no"
        }
      ],
      "result": "normal",
      "createdAt": "2024-01-01T00:00:00.000Z"
    }
  }
}
```

#### Get Questionnaire Results

```http
GET /api/questionnaires/:questionnaireId
Authorization: Bearer <token>
```

Response:
```json
{
  "success": true,
  "data": {
    "questionnaire": {
      "id": "1234567890",
      "childId": "1234567890",
      "answers": [
        {
          "questionId": "1",
          "answer": "yes"
        },
        {
          "questionId": "2",
          "answer": "no"
        }
      ],
      "result": "normal",
      "createdAt": "2024-01-01T00:00:00.000Z"
    }
  }
}
```

### Message Endpoints

#### Send Message

```http
POST /api/messages
Authorization: Bearer <token>
Content-Type: application/json

{
  "phone": "6281234567890",
  "message": "Hello, how are you?"
}
```

Response:
```json
{
  "success": true,
  "data": {
    "message": {
      "id": "1234567890",
      "phone": "6281234567890",
      "message": "Hello, how are you?",
      "status": "sent",
      "createdAt": "2024-01-01T00:00:00.000Z"
    }
  }
}
```

#### Get Message History

```http
GET /api/messages
Authorization: Bearer <token>
```

Response:
```json
{
  "success": true,
  "data": {
    "messages": [
      {
        "id": "1234567890",
        "phone": "6281234567890",
        "message": "Hello, how are you?",
        "status": "sent",
        "createdAt": "2024-01-01T00:00:00.000Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 1
    }
  }
}
```

### Session Endpoints

#### Update Session

```http
PUT /api/sessions
Authorization: Bearer <token>
Content-Type: application/json

{
  "state": "questionnaire",
  "data": {
    "childId": "1234567890"
  }
}
```

Response:
```json
{
  "success": true,
  "data": {
    "session": {
      "id": "1234567890",
      "userId": "1234567890",
      "state": "questionnaire",
      "data": {
        "childId": "1234567890"
      },
      "updatedAt": "2024-01-01T00:00:00.000Z"
    }
  }
}
```

#### Get Session

```http
GET /api/sessions
Authorization: Bearer <token>
```

Response:
```json
{
  "success": true,
  "data": {
    "session": {
      "id": "1234567890",
      "userId": "1234567890",
      "state": "questionnaire",
      "data": {
        "childId": "1234567890"
      },
      "createdAt": "2024-01-01T00:00:00.000Z",
      "updatedAt": "2024-01-01T00:00:00.000Z"
    }
  }
}
```

## Request/Response

### Request Format

```javascript
// Headers
{
  'Content-Type': 'application/json',
  'Authorization': 'Bearer <token>',
  'X-API-Key': '<api-key>',
  'X-WhatsApp-Phone': '<phone>'
}

// Body
{
  "field1": "value1",
  "field2": "value2"
}

// Query Parameters
?page=1&limit=10&sort=createdAt&order=desc

// Path Parameters
/api/resource/:id
```

### Response Format

```javascript
// Success Response
{
  "success": true,
  "data": {
    "field1": "value1",
    "field2": "value2"
  }
}

// Error Response
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Error message",
    "details": {
      "field1": "Error detail 1",
      "field2": "Error detail 2"
    }
  }
}

// Pagination Response
{
  "success": true,
  "data": {
    "items": [
      {
        "field1": "value1",
        "field2": "value2"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 100
    }
  }
}
```

## Error Handling

### Error Codes

```javascript
// Authentication Errors
AUTH_INVALID_TOKEN: 'Invalid authentication token',
AUTH_EXPIRED_TOKEN: 'Authentication token expired',
AUTH_INVALID_API_KEY: 'Invalid API key',
AUTH_INVALID_PHONE: 'Invalid phone number',

// Validation Errors
VAL_INVALID_INPUT: 'Invalid input data',
VAL_REQUIRED_FIELD: 'Required field missing',
VAL_INVALID_FORMAT: 'Invalid data format',
VAL_INVALID_VALUE: 'Invalid field value',

// Resource Errors
RES_NOT_FOUND: 'Resource not found',
RES_ALREADY_EXISTS: 'Resource already exists',
RES_ACCESS_DENIED: 'Access denied to resource',
RES_INVALID_OPERATION: 'Invalid operation on resource',

// System Errors
SYS_INTERNAL_ERROR: 'Internal server error',
SYS_SERVICE_UNAVAILABLE: 'Service unavailable',
SYS_DATABASE_ERROR: 'Database error',
SYS_EXTERNAL_ERROR: 'External service error'
```

### Error Response Format

```javascript
// Validation Error
{
  "success": false,
  "error": {
    "code": "VAL_INVALID_INPUT",
    "message": "Invalid input data",
    "details": {
      "name": "Name is required",
      "phone": "Invalid phone number format"
    }
  }
}

// Resource Error
{
  "success": false,
  "error": {
    "code": "RES_NOT_FOUND",
    "message": "Resource not found",
    "details": {
      "resource": "user",
      "id": "1234567890"
    }
  }
}

// System Error
{
  "success": false,
  "error": {
    "code": "SYS_INTERNAL_ERROR",
    "message": "Internal server error",
    "details": {
      "requestId": "req_1234567890"
    }
  }
}
```

## Rate Limiting

### Rate Limit Headers

```javascript
// Response Headers
{
  'X-RateLimit-Limit': '100',
  'X-RateLimit-Remaining': '99',
  'X-RateLimit-Reset': '1609459200'
}
```

### Rate Limit Response

```javascript
// Rate Limit Exceeded
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded",
    "details": {
      "limit": 100,
      "remaining": 0,
      "reset": 1609459200
    }
  }
}
```

## Best Practices

1. **Authentication**
   - Use JWT for user authentication
   - Use API keys for service authentication
   - Validate phone numbers for WhatsApp
   - Secure sensitive data

2. **Request Handling**
   - Validate input data
   - Sanitize user input
   - Handle file uploads
   - Process requests asynchronously

3. **Response Format**
   - Use consistent response format
   - Include success flag
   - Provide error details
   - Support pagination

4. **Error Handling**
   - Use standard error codes
   - Provide detailed error messages
   - Log error details
   - Handle errors gracefully

5. **Rate Limiting**
   - Set appropriate limits
   - Include rate limit headers
   - Handle limit exceeded
   - Monitor usage

6. **Security**
   - Use HTTPS
   - Implement CORS
   - Set security headers
   - Validate requests

7. **Performance**
   - Cache responses
   - Compress data
   - Optimize queries
   - Monitor performance 