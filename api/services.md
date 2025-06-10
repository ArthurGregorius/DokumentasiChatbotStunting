# 🛠️ API Services

<div align="center">

![API Services](https://img.shields.io/badge/API-Services-blue)
![Status](https://img.shields.io/badge/Status-Active-green)

</div>

## 📋 Overview

Dokumentasi ini menjelaskan service-service yang digunakan dalam API Chatbot Identifikasi Stunting, termasuk business logic, integrasi dengan layanan eksternal, dan penanganan data.

## 👤 User Service

### Create User
```javascript
// services/userService.js
const createUser = async (userData) => {
  try {
    // Validate input
    const validation = validateUserInput(userData);
    if (!validation.isValid) {
      throw new ValidationError(validation.errors);
    }
    
    // Check if user exists
    const existingUser = await User.findOne({ phone: userData.phone });
    if (existingUser) {
      throw new ConflictError('User already exists');
    }
    
    // Create user
    const user = new User(userData);
    await user.save();
    
    return user;
  } catch (error) {
    throw handleServiceError(error);
  }
};
```

### Get User
```javascript
const getUserByPhone = async (phone) => {
  try {
    const user = await User.findOne({ phone });
    if (!user) {
      throw new NotFoundError('User not found');
    }
    
    return user;
  } catch (error) {
    throw handleServiceError(error);
  }
};
```

## 👶 Child Service

### Create Child
```javascript
// services/childService.js
const createChild = async (childData) => {
  try {
    // Validate input
    const validation = validateChildInput(childData);
    if (!validation.isValid) {
      throw new ValidationError(validation.errors);
    }
    
    // Check if user exists
    const user = await User.findById(childData.userId);
    if (!user) {
      throw new NotFoundError('User not found');
    }
    
    // Create child
    const child = new Child(childData);
    await child.save();
    
    return child;
  } catch (error) {
    throw handleServiceError(error);
  }
};
```

### Get Child
```javascript
const getChildById = async (childId) => {
  try {
    const child = await Child.findById(childId);
    if (!child) {
      throw new NotFoundError('Child not found');
    }
    
    return child;
  } catch (error) {
    throw handleServiceError(error);
  }
};
```

## 📝 Questionnaire Service

### Submit Questionnaire
```javascript
// services/questionnaireService.js
const submitQuestionnaire = async (questionnaireData) => {
  try {
    // Validate input
    const validation = validateQuestionnaireInput(questionnaireData);
    if (!validation.isValid) {
      throw new ValidationError(validation.errors);
    }
    
    // Check if child exists
    const child = await Child.findById(questionnaireData.childId);
    if (!child) {
      throw new NotFoundError('Child not found');
    }
    
    // Create questionnaire
    const questionnaire = new Questionnaire(questionnaireData);
    
    // Calculate score and risk level
    await questionnaire.calculateScore();
    await questionnaire.determineRiskLevel();
    
    await questionnaire.save();
    
    return questionnaire;
  } catch (error) {
    throw handleServiceError(error);
  }
};
```

### Get Results
```javascript
const getResultsByChildId = async (childId) => {
  try {
    const questionnaires = await Questionnaire.find({ childId })
      .sort({ createdAt: -1 });
    
    if (!questionnaires.length) {
      throw new NotFoundError('No questionnaires found');
    }
    
    return questionnaires;
  } catch (error) {
    throw handleServiceError(error);
  }
};
```

## 💬 Message Service

### Send Message
```javascript
// services/messageService.js
const sendMessage = async (messageData) => {
  try {
    // Validate input
    const validation = validateMessageInput(messageData);
    if (!validation.isValid) {
      throw new ValidationError(validation.errors);
    }
    
    // Create message
    const message = new Message({
      phone: messageData.phone,
      type: messageData.type,
      content: messageData.message,
      direction: 'outgoing'
    });
    
    await message.save();
    
    // Send via WhatsApp
    try {
      await whatsappService.sendMessage(messageData.phone, messageData.message);
      await message.updateStatus('delivered');
    } catch (error) {
      await message.updateStatus('failed');
      throw error;
    }
    
    return message;
  } catch (error) {
    throw handleServiceError(error);
  }
};
```

### Get Message History
```javascript
const getMessageHistory = async (phone, options) => {
  try {
    const messages = await Message.find({ phone })
      .sort({ createdAt: -1 })
      .skip(options.offset)
      .limit(options.limit);
    
    return messages;
  } catch (error) {
    throw handleServiceError(error);
  }
};
```

## 🔄 Session Service

### Create Session
```javascript
// services/sessionService.js
const createSession = async (sessionData) => {
  try {
    // Validate input
    const validation = validateSessionInput(sessionData);
    if (!validation.isValid) {
      throw new ValidationError(validation.errors);
    }
    
    // Check if user exists
    const user = await User.findById(sessionData.userId);
    if (!user) {
      throw new NotFoundError('User not found');
    }
    
    // Create session
    const session = new Session({
      userId: sessionData.userId,
      type: sessionData.type,
      state: 'initial',
      expiresAt: new Date(Date.now() + 24 * 60 * 60 * 1000)
    });
    
    await session.save();
    
    return session;
  } catch (error) {
    throw handleServiceError(error);
  }
};
```

### End Session
```javascript
const endSession = async (sessionId) => {
  try {
    const session = await Session.findById(sessionId);
    if (!session) {
      throw new NotFoundError('Session not found');
    }
    
    session.state = 'ended';
    await session.save();
    
    return session;
  } catch (error) {
    throw handleServiceError(error);
  }
};
```

## 📱 WhatsApp Service

### Send Message
```javascript
// services/whatsappService.js
const sendMessage = async (phone, message) => {
  try {
    // Validate input
    if (!phone || !message) {
      throw new ValidationError('Phone and message are required');
    }
    
    // Send message via WhatsApp API
    const response = await axios.post(
      `${config.whatsapp.apiUrl}/messages`,
      {
        phone,
        message
      },
      {
        headers: {
          'Authorization': `Bearer ${config.whatsapp.apiKey}`
        }
      }
    );
    
    return response.data;
  } catch (error) {
    throw handleServiceError(error);
  }
};
```

### Handle Webhook
```javascript
const handleWebhook = async (webhookData) => {
  try {
    // Validate webhook
    if (!webhookData.verifyToken || webhookData.verifyToken !== config.whatsapp.webhookToken) {
      throw new ValidationError('Invalid webhook token');
    }
    
    // Process message
    const message = new Message({
      phone: webhookData.phone,
      type: webhookData.type,
      content: webhookData.message,
      direction: 'incoming'
    });
    
    await message.save();
    
    // Process message based on type
    await processMessage(message);
    
    return message;
  } catch (error) {
    throw handleServiceError(error);
  }
};
```

## 📚 Dokumentasi Terkait

- [API Overview](README.md)
- [Controllers](controllers.md)
- [Models](models.md)
- [Configuration](configuration.md)

---

<div align="center">

### 🔗 Navigasi

[⬅️ Kembali ke Models](models.md) | [Lanjut ke Routes ➡️](routes.md)

</div> 