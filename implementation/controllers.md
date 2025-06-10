# Controllers

## 📋 Overview

Dokumentasi ini menjelaskan implementasi detail dari controller-controller yang digunakan dalam Chatbot Identifikasi Stunting, termasuk logika bisnis, validasi, dan penanganan error.

## 👤 User Controller Implementation

### Register User

```javascript
// controllers/userController.js
const { validateUserInput } = require('../utils/validators');
const { handleError } = require('../utils/errorHandler');
const userService = require('../services/userService');

const registerUser = async (req, res) => {
  try {
    const { name, phone, address, birthDate } = req.body;
    
    // Validate input
    const validation = validateUserInput(req.body);
    if (!validation.isValid) {
      return res.status(400).json({
        status: 'error',
        code: 'VALIDATION_ERROR',
        message: validation.errors
      });
    }
    
    // Check if user exists
    const existingUser = await userService.getUserByPhone(phone);
    if (existingUser) {
      return res.status(409).json({
        status: 'error',
        code: 'USER_EXISTS',
        message: 'User already exists'
      });
    }
    
    // Create user
    const user = await userService.createUser({
      name,
      phone,
      address,
      birthDate
    });
    
    res.status(201).json({
      status: 'success',
      data: user
    });
  } catch (error) {
    handleError(error, res);
  }
};
```

### Get User Profile

```javascript
const getUserProfile = async (req, res) => {
  try {
    const { phone } = req.params;
    
    // Validate phone number
    if (!phone || !phone.match(/^62\d{9,12}$/)) {
      return res.status(400).json({
        status: 'error',
        code: 'INVALID_PHONE',
        message: 'Invalid phone number format'
      });
    }
    
    const user = await userService.getUserByPhone(phone);
    if (!user) {
      return res.status(404).json({
        status: 'error',
        code: 'USER_NOT_FOUND',
        message: 'User not found'
      });
    }
    
    // Get user's children
    const children = await userService.getUserChildren(user.id);
    
    res.json({
      status: 'success',
      data: {
        ...user,
        children
      }
    });
  } catch (error) {
    handleError(error, res);
  }
};
```

## 👶 Child Controller Implementation

### Register Child

```javascript
// controllers/childController.js
const { validateChildInput } = require('../utils/validators');
const { handleError } = require('../utils/errorHandler');
const childService = require('../services/childService');

const registerChild = async (req, res) => {
  try {
    const { userId, name, birthDate, gender, height, weight, headCircumference } = req.body;
    
    // Validate input
    const validation = validateChildInput(req.body);
    if (!validation.isValid) {
      return res.status(400).json({
        status: 'error',
        code: 'VALIDATION_ERROR',
        message: validation.errors
      });
    }
    
    // Check if user exists
    const user = await userService.getUserById(userId);
    if (!user) {
      return res.status(404).json({
        status: 'error',
        code: 'USER_NOT_FOUND',
        message: 'User not found'
      });
    }
    
    // Create child
    const child = await childService.createChild({
      userId,
      name,
      birthDate,
      gender,
      height,
      weight,
      headCircumference
    });
    
    res.status(201).json({
      status: 'success',
      data: child
    });
  } catch (error) {
    handleError(error, res);
  }
};
```

### Get Child Data

```javascript
const getChildData = async (req, res) => {
  try {
    const { childId } = req.params;
    
    // Validate child ID
    if (!childId || !childId.match(/^[0-9a-fA-F]{24}$/)) {
      return res.status(400).json({
        status: 'error',
        code: 'INVALID_ID',
        message: 'Invalid child ID format'
      });
    }
    
    const child = await childService.getChildById(childId);
    if (!child) {
      return res.status(404).json({
        status: 'error',
        code: 'CHILD_NOT_FOUND',
        message: 'Child not found'
      });
    }
    
    // Get child's questionnaires
    const questionnaires = await childService.getChildQuestionnaires(childId);
    
    res.json({
      status: 'success',
      data: {
        ...child,
        questionnaires
      }
    });
  } catch (error) {
    handleError(error, res);
  }
};
```

## 📝 Questionnaire Controller Implementation

### Submit Questionnaire

```javascript
// controllers/questionnaireController.js
const { validateQuestionnaireInput } = require('../utils/validators');
const { handleError } = require('../utils/errorHandler');
const questionnaireService = require('../services/questionnaireService');

const submitQuestionnaire = async (req, res) => {
  try {
    const { childId, type, answers } = req.body;
    
    // Validate input
    const validation = validateQuestionnaireInput(req.body);
    if (!validation.isValid) {
      return res.status(400).json({
        status: 'error',
        code: 'VALIDATION_ERROR',
        message: validation.errors
      });
    }
    
    // Check if child exists
    const child = await childService.getChildById(childId);
    if (!child) {
      return res.status(404).json({
        status: 'error',
        code: 'CHILD_NOT_FOUND',
        message: 'Child not found'
      });
    }
    
    // Submit questionnaire
    const result = await questionnaireService.submitQuestionnaire({
      childId,
      type,
      answers
    });
    
    res.status(201).json({
      status: 'success',
      data: result
    });
  } catch (error) {
    handleError(error, res);
  }
};
```

### Get Questionnaire Results

```javascript
const getQuestionnaireResults = async (req, res) => {
  try {
    const { childId } = req.params;
    
    // Validate child ID
    if (!childId || !childId.match(/^[0-9a-fA-F]{24}$/)) {
      return res.status(400).json({
        status: 'error',
        code: 'INVALID_ID',
        message: 'Invalid child ID format'
      });
    }
    
    const results = await questionnaireService.getResultsByChildId(childId);
    
    res.json({
      status: 'success',
      data: results
    });
  } catch (error) {
    handleError(error, res);
  }
};
```

## 💬 Message Controller Implementation

### Send Message

```javascript
// controllers/messageController.js
const { validateMessageInput } = require('../utils/validators');
const { handleError } = require('../utils/errorHandler');
const messageService = require('../services/messageService');

const sendMessage = async (req, res) => {
  try {
    const { phone, message, type } = req.body;
    
    // Validate input
    const validation = validateMessageInput(req.body);
    if (!validation.isValid) {
      return res.status(400).json({
        status: 'error',
        code: 'VALIDATION_ERROR',
        message: validation.errors
      });
    }
    
    // Send message
    const result = await messageService.sendMessage({
      phone,
      message,
      type
    });
    
    res.json({
      status: 'success',
      data: result
    });
  } catch (error) {
    handleError(error, res);
  }
};
```

### Get Message History

```javascript
const getMessageHistory = async (req, res) => {
  try {
    const { phone } = req.params;
    const { limit = 50, offset = 0 } = req.query;
    
    // Validate phone number
    if (!phone || !phone.match(/^62\d{9,12}$/)) {
      return res.status(400).json({
        status: 'error',
        code: 'INVALID_PHONE',
        message: 'Invalid phone number format'
      });
    }
    
    const messages = await messageService.getMessageHistory(phone, {
      limit: parseInt(limit),
      offset: parseInt(offset)
    });
    
    res.json({
      status: 'success',
      data: messages
    });
  } catch (error) {
    handleError(error, res);
  }
};
```

## 🔄 Session Controller Implementation

### Create Session

```javascript
// controllers/sessionController.js
const { validateSessionInput } = require('../utils/validators');
const { handleError } = require('../utils/errorHandler');
const sessionService = require('../services/sessionService');

const createSession = async (req, res) => {
  try {
    const { userId, type } = req.body;
    
    // Validate input
    const validation = validateSessionInput(req.body);
    if (!validation.isValid) {
      return res.status(400).json({
        status: 'error',
        code: 'VALIDATION_ERROR',
        message: validation.errors
      });
    }
    
    // Check if user exists
    const user = await userService.getUserById(userId);
    if (!user) {
      return res.status(404).json({
        status: 'error',
        code: 'USER_NOT_FOUND',
        message: 'User not found'
      });
    }
    
    // Create session
    const session = await sessionService.createSession({
      userId,
      type
    });
    
    res.status(201).json({
      status: 'success',
      data: session
    });
  } catch (error) {
    handleError(error, res);
  }
};
```

### End Session

```javascript
const endSession = async (req, res) => {
  try {
    const { sessionId } = req.params;
    
    // Validate session ID
    if (!sessionId || !sessionId.match(/^[0-9a-fA-F]{24}$/)) {
      return res.status(400).json({
        status: 'error',
        code: 'INVALID_ID',
        message: 'Invalid session ID format'
      });
    }
    
    const session = await sessionService.endSession(sessionId);
    
    res.json({
      status: 'success',
      data: session
    });
  } catch (error) {
    handleError(error, res);
  }
};
```
