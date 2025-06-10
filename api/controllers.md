# 🎮 API Controllers

<div align="center">

![API Controllers](https://img.shields.io/badge/API-Controllers-blue)
![Status](https://img.shields.io/badge/Status-Active-green)

</div>

## 📋 Overview

Dokumentasi ini menjelaskan controller-controller yang digunakan dalam API Chatbot Identifikasi Stunting, termasuk endpoint, request handling, dan response format.

## 👤 User Controller

### Register User
```javascript
// controllers/userController.js
const registerUser = async (req, res) => {
  try {
    const { name, phone, address, birthDate } = req.body;
    
    // Validate input
    const validation = validateUserInput(req.body);
    if (!validation.isValid) {
      return res.status(400).json({
        status: 'error',
        message: validation.errors
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
    
    const user = await userService.getUserByPhone(phone);
    if (!user) {
      return res.status(404).json({
        status: 'error',
        message: 'User not found'
      });
    }
    
    res.json({
      status: 'success',
      data: user
    });
  } catch (error) {
    handleError(error, res);
  }
};
```

## 👶 Child Controller

### Register Child
```javascript
// controllers/childController.js
const registerChild = async (req, res) => {
  try {
    const { userId, name, birthDate, gender, height, weight, headCircumference } = req.body;
    
    // Validate input
    const validation = validateChildInput(req.body);
    if (!validation.isValid) {
      return res.status(400).json({
        status: 'error',
        message: validation.errors
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
    
    const child = await childService.getChildById(childId);
    if (!child) {
      return res.status(404).json({
        status: 'error',
        message: 'Child not found'
      });
    }
    
    res.json({
      status: 'success',
      data: child
    });
  } catch (error) {
    handleError(error, res);
  }
};
```

## 📝 Questionnaire Controller

### Submit Questionnaire
```javascript
// controllers/questionnaireController.js
const submitQuestionnaire = async (req, res) => {
  try {
    const { childId, type, answers } = req.body;
    
    // Validate input
    const validation = validateQuestionnaireInput(req.body);
    if (!validation.isValid) {
      return res.status(400).json({
        status: 'error',
        message: validation.errors
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

## 💬 Message Controller

### Send Message
```javascript
// controllers/messageController.js
const sendMessage = async (req, res) => {
  try {
    const { phone, message, type } = req.body;
    
    // Validate input
    const validation = validateMessageInput(req.body);
    if (!validation.isValid) {
      return res.status(400).json({
        status: 'error',
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

## 🔄 Session Controller

### Create Session
```javascript
// controllers/sessionController.js
const createSession = async (req, res) => {
  try {
    const { userId, type } = req.body;
    
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

## 📚 Dokumentasi Terkait

- [API Overview](README.md)
- [Configuration](configuration.md)
- [Models](models.md)
- [Services](services.md)

---

<div align="center">

### 🔗 Navigasi

[⬅️ Kembali ke Configuration](configuration.md) | [Lanjut ke Models ➡️](models.md)

</div> 