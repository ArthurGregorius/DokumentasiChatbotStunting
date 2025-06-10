# 💻 Development: Local Setup

<div align="center">

![Development](https://img.shields.io/badge/Development-Local-blue)
![Status](https://img.shields.io/badge/Status-Active-green)

</div>

## 📋 Overview

Dokumentasi ini menjelaskan langkah-langkah setup environment development lokal untuk Chatbot Identifikasi Stunting, termasuk instalasi dependencies, konfigurasi database, dan tools development.

## 🛠️ Prerequisites

### 1. Required Software
```bash
# Node.js & npm
node -v  # Should be v14.x or higher
npm -v   # Should be v6.x or higher

# Git
git --version  # Should be v2.x or higher

# Docker & Docker Compose
docker --version      # Should be v20.x or higher
docker-compose --version  # Should be v2.x or higher

# MongoDB
mongod --version  # Should be v4.4 or higher

# Redis
redis-cli --version  # Should be v6.x or higher
```

### 2. Development Tools
```bash
# VS Code Extensions
- ESLint
- Prettier
- Docker
- MongoDB for VS Code
- Redis for VS Code
- GitLens
```

## 🚀 Setup Steps

### 1. Clone Repository
```bash
# Clone repository
git clone https://github.com/your-org/chatbot-stunting.git
cd chatbot-stunting

# Create and switch to development branch
git checkout -b development
```

### 2. Install Dependencies
```bash
# Install npm dependencies
npm install

# Install development dependencies
npm install --save-dev \
    @types/node \
    @types/express \
    @types/mongoose \
    @types/jest \
    typescript \
    ts-node \
    nodemon \
    jest \
    supertest \
    eslint \
    prettier
```

### 3. Environment Setup
```bash
# Create development environment file
cp .env.example .env.development

# Update environment variables
nano .env.development
```

### 4. Database Setup
```bash
# Start MongoDB
mongod --dbpath /data/db

# Start Redis
redis-server

# Create database and collections
mongo
use chatbot-stunting
db.createCollection('users')
db.createCollection('children')
db.createCollection('questionnaires')
db.createCollection('messages')
db.createCollection('sessions')
```

## 🔧 Development Tools

### 1. VS Code Configuration
```json
// .vscode/settings.json
{
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "typescript.tsdk": "node_modules/typescript/lib",
  "typescript.enablePromptUseWorkspaceTsdk": true,
  "eslint.validate": [
    "javascript",
    "typescript"
  ]
}
```

### 2. ESLint Configuration
```javascript
// .eslintrc.js
module.exports = {
  parser: '@typescript-eslint/parser',
  extends: [
    'plugin:@typescript-eslint/recommended',
    'prettier'
  ],
  parserOptions: {
    ecmaVersion: 2020,
    sourceType: 'module'
  },
  rules: {
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/no-explicit-any': 'warn',
    '@typescript-eslint/no-unused-vars': ['error', { 'argsIgnorePattern': '^_' }]
  }
};
```

### 3. Prettier Configuration
```json
// .prettierrc
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2
}
```

## 🧪 Testing Setup

### 1. Jest Configuration
```javascript
// jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/src'],
  testMatch: ['**/__tests__/**/*.ts', '**/?(*.)+(spec|test).ts'],
  transform: {
    '^.+\\.ts$': 'ts-jest'
  },
  coverageDirectory: 'coverage',
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.d.ts'
  ]
};
```

### 2. Test Database Setup
```bash
# Create test database
mongo
use chatbot-stunting-test
db.createCollection('users')
db.createCollection('children')
db.createCollection('questionnaires')
db.createCollection('messages')
db.createCollection('sessions')
```

## 📝 Development Workflow

### 1. Start Development Server
```bash
# Start development server with hot reload
npm run dev

# Start development server with debug
npm run dev:debug
```

### 2. Run Tests
```bash
# Run all tests
npm test

# Run tests with coverage
npm run test:coverage

# Run specific test file
npm test -- src/__tests__/user.test.ts
```

### 3. Code Quality
```bash
# Lint code
npm run lint

# Format code
npm run format

# Check types
npm run type-check
```

## 🔍 Debugging

### 1. VS Code Debug Configuration
```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Program",
      "skipFiles": ["<node_internals>/**"],
      "program": "${workspaceFolder}/src/index.ts",
      "preLaunchTask": "tsc: build - tsconfig.json",
      "outFiles": ["${workspaceFolder}/dist/**/*.js"]
    }
  ]
}
```

### 2. Logging
```javascript
// src/utils/logger.ts
import winston from 'winston';

const logger = winston.createLogger({
  level: process.env.NODE_ENV === 'development' ? 'debug' : 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.Console({
      format: winston.format.simple()
    })
  ]
});

export default logger;
```

## 📚 Dokumentasi Terkait

- [Testing Guidelines](testing.md)
- [Code Style Guide](code-style.md)
- [Production Guidelines](../deployment/production.md)

---

<div align="center">

### 🔗 Navigasi

[⬅️ Kembali ke Production Guidelines](../deployment/production.md) | [Lanjut ke Testing Guidelines ➡️](testing.md)

</div> 