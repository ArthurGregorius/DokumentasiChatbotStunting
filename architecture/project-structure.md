# Project Structure

## Overview

Chatbot Identifikasi Stunting mengikuti struktur proyek yang terorganisir dengan baik untuk memudahkan pengembangan dan pemeliharaan. Berikut adalah penjelasan detail tentang struktur direktori dan file-file utama.

## Root Directory

```
chatbot-stunting/
├── .vscode/                 # VS Code configuration
├── auth_info_baileys/      # WhatsApp authentication data
├── config/                 # Configuration files
├── docs/                   # Documentation
├── src/                    # Source code
├── .gitignore             # Git ignore rules
├── AppScript.gs           # Google Apps Script code
├── README.md              # Project documentation
├── dockerfile             # Docker configuration
├── index.js              # Application entry point
├── package.json          # Project dependencies
└── testApi.js            # API testing
```

## Source Code Structure

### src/
```
src/
├── constants/            # Constant values and messages
│   ├── messages.js      # Message templates
│   └── questions.js     # Questionnaire data
├── controllers/         # Business logic
│   ├── registrationController.js
│   ├── childRegistrationController.js
│   └── questionnaireController.js
├── handlers/           # Message handlers
│   ├── messageHandler.js
│   ├── menuHandler.js
│   └── nextProcessHandler.js
├── models/            # Data models
│   └── userState.js
├── services/         # External services
│   └── apiService.js
└── utils/           # Utility functions
    └── helpers.js
```

## Key Components

### 1. Entry Point (index.js)
```javascript
// Main application entry point
const { DisconnectReason, useMultiFileAuthState } = require('@whiskeysockets/baileys');
const makeWASocket = require('@whiskeysockets/baileys').default;
const messageHandler = require('./src/handlers/messageHandler');
const UserState = require('./src/models/userState');
```

### 2. Message Handler (src/handlers/messageHandler.js)
```javascript
// Handles incoming messages
async function handleMessage(sock, m, userStateManager) {
  // Message processing logic
}
```

### 3. User State Model (src/models/userState.js)
```javascript
// Manages user session state
class UserState {
  // State management methods
}
```

### 4. API Service (src/services/apiService.js)
```javascript
// Handles external API communication
class ApiService {
  // API interaction methods
}
```

## Configuration Files

### 1. package.json
```json
{
  "name": "ChatbotSerpihan",
  "version": "1.0.0",
  "dependencies": {
    "@whiskeysockets/baileys": "^6.7.16",
    "axios": "^1.8.3",
    "nodemon": "^3.1.9",
    "qrcode-terminal": "^0.12.0"
  }
}
```

### 2. dockerfile
```dockerfile
# Docker configuration for deployment
FROM node:14
WORKDIR /app
COPY . .
RUN npm install
CMD ["npm", "start"]
```

## Google Apps Script Integration

### AppScript.gs
```javascript
// Google Apps Script for data handling
function doGet(e) {
  // Handle GET requests
}

function doPost(e) {
  // Handle POST requests
}
```

## Documentation Structure

### docs/
```
docs/
├── getting-started/
│   ├── prerequisites.md
│   ├── installation.md
│   └── configuration.md
├── features/
│   ├── user-registration.md
│   ├── child-data-management.md
│   └── questionnaire-system.md
└── architecture/
    ├── system-overview.md
    ├── technical-stack.md
    └── project-structure.md
```

## Development Guidelines

### Code Organization
1. **Modular Structure**
   - Separate concerns
   - Clear responsibilities
   - Easy maintenance

2. **File Naming**
   - Consistent naming
   - Clear purpose
   - Proper extensions

3. **Directory Structure**
   - Logical grouping
   - Easy navigation
   - Scalable design

### Best Practices
1. **Code Style**
   - Consistent formatting
   - Clear comments
   - Proper documentation

2. **Error Handling**
   - Try-catch blocks
   - Error logging
   - User feedback

3. **Security**
   - Input validation
   - Data encryption
   - Access control

## Testing Structure

### Test Files
```
tests/
├── unit/
│   ├── controllers/
│   ├── services/
│   └── utils/
├── integration/
│   ├── api/
│   └── handlers/
└── e2e/
    └── flows/
```

## Deployment Structure

### Production
```
production/
├── config/
├── logs/
├── data/
└── backups/
```

### Staging
```
staging/
├── config/
├── logs/
└── data/
```

## Maintenance

### Backup Structure
```
backups/
├── daily/
├── weekly/
└── monthly/
```

### Log Structure
```
logs/
├── app/
├── error/
└── access/
``` 