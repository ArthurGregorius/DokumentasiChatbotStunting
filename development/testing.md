# 🧪 Development: Testing Guidelines

<div align="center">

![Development](https://img.shields.io/badge/Development-Testing-blue)
![Status](https://img.shields.io/badge/Status-Active-green)

</div>

## 📋 Overview

Dokumentasi ini menjelaskan panduan testing untuk Chatbot Identifikasi Stunting, termasuk unit testing, integration testing, dan end-to-end testing.

## 🎯 Testing Strategy

### 1. Unit Testing
```typescript
// src/__tests__/unit/user.test.ts
import { UserService } from '../../services/user.service';
import { UserModel } from '../../models/user.model';

describe('UserService', () => {
  let userService: UserService;

  beforeEach(() => {
    userService = new UserService();
  });

  describe('createUser', () => {
    it('should create a new user', async () => {
      const userData = {
        name: 'Test User',
        phone: '+6281234567890',
        address: 'Test Address'
      };

      const user = await userService.createUser(userData);

      expect(user).toBeDefined();
      expect(user.name).toBe(userData.name);
      expect(user.phone).toBe(userData.phone);
    });

    it('should throw error for duplicate phone', async () => {
      const userData = {
        name: 'Test User',
        phone: '+6281234567890',
        address: 'Test Address'
      };

      await userService.createUser(userData);

      await expect(userService.createUser(userData))
        .rejects
        .toThrow('Phone number already registered');
    });
  });
});
```

### 2. Integration Testing
```typescript
// src/__tests__/integration/user.test.ts
import request from 'supertest';
import { app } from '../../app';
import { UserModel } from '../../models/user.model';

describe('User API', () => {
  beforeEach(async () => {
    await UserModel.deleteMany({});
  });

  describe('POST /api/users/register', () => {
    it('should register a new user', async () => {
      const userData = {
        name: 'Test User',
        phone: '+6281234567890',
        address: 'Test Address'
      };

      const response = await request(app)
        .post('/api/users/register')
        .send(userData);

      expect(response.status).toBe(201);
      expect(response.body.data.name).toBe(userData.name);
    });

    it('should validate required fields', async () => {
      const response = await request(app)
        .post('/api/users/register')
        .send({});

      expect(response.status).toBe(400);
      expect(response.body.errors).toBeDefined();
    });
  });
});
```

### 3. End-to-End Testing
```typescript
// src/__tests__/e2e/chatbot.test.ts
import { TestClient } from '../utils/test-client';

describe('Chatbot Flow', () => {
  let client: TestClient;

  beforeAll(async () => {
    client = new TestClient();
    await client.initialize();
  });

  afterAll(async () => {
    await client.cleanup();
  });

  it('should complete registration flow', async () => {
    // Start registration
    await client.sendMessage('Halo');
    expect(await client.getLastMessage()).toContain('Selamat datang');

    // Enter name
    await client.sendMessage('John Doe');
    expect(await client.getLastMessage()).toContain('nomor telepon');

    // Enter phone
    await client.sendMessage('+6281234567890');
    expect(await client.getLastMessage()).toContain('alamat');

    // Enter address
    await client.sendMessage('Test Address');
    expect(await client.getLastMessage()).toContain('registrasi berhasil');
  });
});
```

## 🛠️ Test Setup

### 1. Test Environment
```typescript
// src/__tests__/utils/test-environment.ts
import mongoose from 'mongoose';
import { MongoMemoryServer } from 'mongodb-memory-server';

export class TestEnvironment {
  private mongoServer: MongoMemoryServer;

  async setup() {
    this.mongoServer = await MongoMemoryServer.create();
    const mongoUri = this.mongoServer.getUri();
    await mongoose.connect(mongoUri);
  }

  async teardown() {
    await mongoose.disconnect();
    await this.mongoServer.stop();
  }
}
```

### 2. Test Utilities
```typescript
// src/__tests__/utils/test-utils.ts
import { UserModel } from '../../models/user.model';
import { ChildModel } from '../../models/child.model';

export const createTestUser = async (userData = {}) => {
  return await UserModel.create({
    name: 'Test User',
    phone: '+6281234567890',
    address: 'Test Address',
    ...userData
  });
};

export const createTestChild = async (userId: string, childData = {}) => {
  return await ChildModel.create({
    userId,
    name: 'Test Child',
    birthDate: new Date(),
    gender: 'male',
    ...childData
  });
};
```

## 📊 Test Coverage

### 1. Coverage Configuration
```javascript
// jest.config.js
module.exports = {
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/*.test.{ts,tsx}',
    '!src/**/__tests__/**'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  }
};
```

### 2. Coverage Report
```bash
# Generate coverage report
npm run test:coverage

# View coverage report
open coverage/lcov-report/index.html
```

## 🔄 Continuous Integration

### 1. GitHub Actions
```yaml
# .github/workflows/test.yml
name: Test

on:
  push:
    branches: [ main, development ]
  pull_request:
    branches: [ main, development ]

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      mongodb:
        image: mongo:4.4
        ports:
          - 27017:27017
      redis:
        image: redis:6
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v2
      
      - name: Setup Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '14'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run tests
        run: npm test
        
      - name: Upload coverage
        uses: codecov/codecov-action@v2
```

## 📝 Test Documentation

### 1. Test Cases
```markdown
# Test Cases

## User Registration
1. Valid user data
   - Input: Complete user data
   - Expected: User created successfully
   - Status: ✅

2. Duplicate phone number
   - Input: Existing phone number
   - Expected: Error message
   - Status: ✅

3. Invalid phone format
   - Input: Invalid phone number
   - Expected: Validation error
   - Status: ✅
```

### 2. Test Reports
```bash
# Generate test report
npm run test:report

# View test report
open test-report.html
```

## 📚 Dokumentasi Terkait

- [Local Setup](local-setup.md)
- [Code Style Guide](code-style.md)
- [Production Guidelines](../deployment/production.md)

---

<div align="center">

### 🔗 Navigasi

[⬅️ Kembali ke Local Setup](local-setup.md) | [Lanjut ke Code Style Guide ➡️](code-style.md)

</div> 