# API Testing

## Overview

Dokumentasi ini menjelaskan strategi dan implementasi testing untuk API chatbot, termasuk unit testing, integration testing, dan end-to-end testing.

## Test Environment

### Setup

1. **Environment Variables**
   ```bash
   # .env.test
   NODE_ENV=test
   API_KEY=test_api_key
   GOOGLE_APPS_SCRIPT_DEPLOYMENT_ID=test_deployment_id
   ```

2. **Test Database**
   ```javascript
   // test/setup.js
   const { MongoMemoryServer } = require('mongodb-memory-server');
   
   let mongod;
   
   beforeAll(async () => {
     mongod = await MongoMemoryServer.create();
     const uri = mongod.getUri();
     await mongoose.connect(uri);
   });
   
   afterAll(async () => {
     await mongoose.disconnect();
     await mongod.stop();
   });
   ```

## Unit Testing

### 1. Service Tests

```javascript
describe('WhatsApp Service', () => {
  let whatsappService;
  
  beforeEach(() => {
    whatsappService = new WhatsAppService();
  });
  
  test('should connect successfully', async () => {
    const sock = await whatsappService.connect();
    expect(sock).toBeDefined();
  });
  
  test('should send message successfully', async () => {
    const result = await whatsappService.sendMessage(
      '6281234567890',
      'Test message'
    );
    expect(result).toBe(true);
  });
  
  test('should handle connection error', async () => {
    await expect(whatsappService.connect())
      .rejects
      .toThrow('Connection failed');
  });
});
```

### 2. Model Tests

```javascript
describe('User Model', () => {
  test('should create user successfully', async () => {
    const user = new User({
      phoneNumber: '6281234567890',
      name: 'Test User',
      role: 'user'
    });
    
    await user.save();
    
    expect(user._id).toBeDefined();
    expect(user.phoneNumber).toBe('6281234567890');
  });
  
  test('should validate phone number', async () => {
    const user = new User({
      phoneNumber: 'invalid',
      name: 'Test User'
    });
    
    await expect(user.save())
      .rejects
      .toThrow('Invalid phone number');
  });
  
  test('should find user by phone number', async () => {
    const user = await User.findByPhone('6281234567890');
    expect(user).toBeDefined();
  });
});
```

## Integration Testing

### 1. API Endpoint Tests

```javascript
describe('User API', () => {
  let app;
  let token;
  
  beforeAll(async () => {
    app = await setupTestApp();
    token = await generateTestToken();
  });
  
  test('should register user', async () => {
    const response = await request(app)
      .post('/api/users/register')
      .send({
        phoneNumber: '6281234567890',
        name: 'Test User'
      });
    
    expect(response.status).toBe(201);
    expect(response.body.success).toBe(true);
    expect(response.body.data.user).toBeDefined();
  });
  
  test('should get user profile', async () => {
    const response = await request(app)
      .get('/api/users/profile')
      .set('Authorization', `Bearer ${token}`);
    
    expect(response.status).toBe(200);
    expect(response.body.success).toBe(true);
    expect(response.body.data).toBeDefined();
  });
});
```

### 2. Database Integration Tests

```javascript
describe('Database Integration', () => {
  let db;
  
  beforeAll(async () => {
    db = await connectTestDatabase();
  });
  
  afterAll(async () => {
    await db.close();
  });
  
  test('should save and retrieve data', async () => {
    const user = new User({
      phoneNumber: '6281234567890',
      name: 'Test User'
    });
    
    await user.save();
    
    const found = await User.findById(user._id);
    expect(found).toBeDefined();
    expect(found.phoneNumber).toBe('6281234567890');
  });
  
  test('should handle database errors', async () => {
    const invalidUser = new User({});
    
    await expect(invalidUser.save())
      .rejects
      .toThrow();
  });
});
```

## End-to-End Testing

### 1. User Flow Tests

```javascript
describe('User Registration Flow', () => {
  let app;
  
  beforeAll(async () => {
    app = await setupTestApp();
  });
  
  test('should complete registration process', async () => {
    // Register user
    const registerResponse = await request(app)
      .post('/api/users/register')
      .send({
        phoneNumber: '6281234567890',
        name: 'Test User'
      });
    
    expect(registerResponse.status).toBe(201);
    const { token } = registerResponse.body.data;
    
    // Register child
    const childResponse = await request(app)
      .post('/api/children')
      .set('Authorization', `Bearer ${token}`)
      .send({
        name: 'Test Child',
        dateOfBirth: '2020-01-01',
        gender: 'male'
      });
    
    expect(childResponse.status).toBe(201);
    
    // Submit questionnaire
    const questionnaireResponse = await request(app)
      .post('/api/questionnaires')
      .set('Authorization', `Bearer ${token}`)
      .send({
        childId: childResponse.body.data._id,
        answers: [
          { questionId: '1', answer: 'yes' },
          { questionId: '2', answer: 'no' }
        ]
      });
    
    expect(questionnaireResponse.status).toBe(201);
  });
});
```

### 2. WhatsApp Integration Tests

```javascript
describe('WhatsApp Integration', () => {
  let whatsappService;
  
  beforeAll(async () => {
    whatsappService = new WhatsAppService();
    await whatsappService.connect();
  });
  
  test('should send and receive messages', async () => {
    // Send message
    const sendResult = await whatsappService.sendMessage(
      '6281234567890',
      'Test message'
    );
    expect(sendResult).toBe(true);
    
    // Wait for response
    const response = await new Promise(resolve => {
      whatsappService.on('message', msg => {
        if (msg.from === '6281234567890') {
          resolve(msg);
        }
      });
    });
    
    expect(response).toBeDefined();
    expect(response.content).toBeDefined();
  });
});
```

## Performance Testing

### 1. Load Testing

```javascript
describe('API Performance', () => {
  test('should handle concurrent requests', async () => {
    const requests = Array(100).fill().map(() =>
      request(app)
        .get('/api/health')
        .set('Authorization', `Bearer ${token}`)
    );
    
    const start = Date.now();
    const responses = await Promise.all(requests);
    const duration = Date.now() - start;
    
    expect(responses.every(r => r.status === 200)).toBe(true);
    expect(duration).toBeLessThan(5000); // 5 seconds
  });
  
  test('should handle database load', async () => {
    const users = Array(1000).fill().map((_, i) => ({
      phoneNumber: `6281234567${i}`,
      name: `Test User ${i}`
    }));
    
    const start = Date.now();
    await User.insertMany(users);
    const duration = Date.now() - start;
    
    expect(duration).toBeLessThan(10000); // 10 seconds
  });
});
```

### 2. Stress Testing

```javascript
describe('API Stress Test', () => {
  test('should handle high load', async () => {
    const requests = Array(1000).fill().map(() =>
      request(app)
        .post('/api/messages')
        .set('Authorization', `Bearer ${token}`)
        .send({
          content: 'Test message',
          type: 'text'
        })
    );
    
    const start = Date.now();
    const results = await Promise.allSettled(requests);
    const duration = Date.now() - start;
    
    const success = results.filter(r => r.status === 'fulfilled').length;
    const failed = results.filter(r => r.status === 'rejected').length;
    
    expect(success).toBeGreaterThan(900); // 90% success rate
    expect(duration).toBeLessThan(30000); // 30 seconds
  });
});
```

## Test Coverage

### 1. Coverage Configuration

```javascript
module.exports = {
  collectCoverage: true,
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov', 'html'],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  },
  collectCoverageFrom: [
    'src/**/*.js',
    '!src/config/**',
    '!src/tests/**'
  ]
};
```

### 2. Coverage Reports

```javascript
describe('Coverage Report', () => {
  test('should maintain coverage thresholds', () => {
    const coverage = require('../coverage/coverage-summary.json');
    
    expect(coverage.total.branches.pct).toBeGreaterThanOrEqual(80);
    expect(coverage.total.functions.pct).toBeGreaterThanOrEqual(80);
    expect(coverage.total.lines.pct).toBeGreaterThanOrEqual(80);
    expect(coverage.total.statements.pct).toBeGreaterThanOrEqual(80);
  });
});
```

## Test Data

### 1. Test Fixtures

```javascript
const testData = {
  users: [
    {
      phoneNumber: '6281234567890',
      name: 'Test User 1',
      role: 'user'
    },
    {
      phoneNumber: '6281234567891',
      name: 'Test User 2',
      role: 'admin'
    }
  ],
  
  children: [
    {
      name: 'Test Child 1',
      dateOfBirth: '2020-01-01',
      gender: 'male',
      height: 100,
      weight: 15
    },
    {
      name: 'Test Child 2',
      dateOfBirth: '2021-01-01',
      gender: 'female',
      height: 90,
      weight: 12
    }
  ],
  
  questionnaires: [
    {
      answers: [
        { questionId: '1', answer: 'yes' },
        { questionId: '2', answer: 'no' }
      ]
    },
    {
      answers: [
        { questionId: '1', answer: 'no' },
        { questionId: '2', answer: 'yes' }
      ]
    }
  ]
};
```

### 2. Test Mocks

```javascript
const mocks = {
  whatsapp: {
    sendMessage: jest.fn().mockResolvedValue(true),
    handleIncomingMessage: jest.fn().mockResolvedValue(true)
  },
  
  sheets: {
    appendRow: jest.fn().mockResolvedValue(true),
    getData: jest.fn().mockResolvedValue([])
  },
  
  email: {
    sendEmail: jest.fn().mockResolvedValue(true),
    sendBulkEmail: jest.fn().mockResolvedValue({ success: 1, failed: 0 })
  }
};
```

## Best Practices

1. **Test Organization**
   - Group related tests
   - Use descriptive names
   - Follow AAA pattern
   - Clean up after tests

2. **Test Data Management**
   - Use fixtures
   - Implement mocks
   - Clean up data
   - Isolate tests

3. **Test Coverage**
   - Set thresholds
   - Monitor coverage
   - Review reports
   - Update tests

4. **Performance Testing**
   - Load testing
   - Stress testing
   - Monitor metrics
   - Set benchmarks

5. **Continuous Integration**
   - Automated testing
   - Coverage reporting
   - Test environment
   - Deployment checks

## Maintenance

### Regular Tasks

1. **Daily**
   - Run test suite
   - Check test coverage
   - Review failed tests

2. **Weekly**
   - Update test data
   - Review test performance
   - Clean up old tests

3. **Monthly**
   - Update dependencies
   - Review test strategy
   - Update documentation 