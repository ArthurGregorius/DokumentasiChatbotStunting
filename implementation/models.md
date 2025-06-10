# Models

## 📋 Overview

Dokumentasi ini menjelaskan implementasi detail dari model-model data yang digunakan dalam Chatbot Identifikasi Stunting, termasuk schema, validasi, dan relasi antar model.

## 👤 User Model Implementation

### Schema Definition

```javascript
// models/User.js
const mongoose = require('mongoose');
const { Schema } = mongoose;

const userSchema = new Schema({
  name: {
    type: String,
    required: [true, 'Name is required'],
    trim: true,
    minlength: [3, 'Name must be at least 3 characters long'],
    maxlength: [100, 'Name cannot exceed 100 characters']
  },
  phone: {
    type: String,
    required: [true, 'Phone number is required'],
    unique: true,
    trim: true,
    match: [/^62\d{9,12}$/, 'Please enter a valid Indonesian phone number']
  },
  address: {
    type: String,
    required: [true, 'Address is required'],
    trim: true,
    minlength: [10, 'Address must be at least 10 characters long']
  },
  birthDate: {
    type: Date,
    required: [true, 'Birth date is required'],
    validate: {
      validator: function(v) {
        return v <= new Date();
      },
      message: 'Birth date cannot be in the future'
    }
  },
  createdAt: {
    type: Date,
    default: Date.now
  },
  updatedAt: {
    type: Date,
    default: Date.now
  }
});

// Update timestamps on save
userSchema.pre('save', function(next) {
  this.updatedAt = Date.now();
  next();
});
```

### Methods

```javascript
userSchema.methods = {
  async updateProfile(data) {
    const allowedUpdates = ['name', 'address'];
    Object.keys(data).forEach(key => {
      if (allowedUpdates.includes(key)) {
        this[key] = data[key];
      }
    });
    this.updatedAt = Date.now();
    return this.save();
  },
  
  async getChildren() {
    return this.model('Child').find({ userId: this._id });
  }
};

// Static methods
userSchema.statics = {
  async findByPhone(phone) {
    return this.findOne({ phone });
  }
};
```

## 👶 Child Model Implementation

### Schema Definition

```javascript
// models/Child.js
const mongoose = require('mongoose');
const { Schema } = mongoose;

const childSchema = new Schema({
  userId: {
    type: Schema.Types.ObjectId,
    ref: 'User',
    required: [true, 'User ID is required']
  },
  name: {
    type: String,
    required: [true, 'Name is required'],
    trim: true,
    minlength: [3, 'Name must be at least 3 characters long'],
    maxlength: [100, 'Name cannot exceed 100 characters']
  },
  birthDate: {
    type: Date,
    required: [true, 'Birth date is required'],
    validate: {
      validator: function(v) {
        return v <= new Date();
      },
      message: 'Birth date cannot be in the future'
    }
  },
  gender: {
    type: String,
    required: [true, 'Gender is required'],
    enum: {
      values: ['male', 'female'],
      message: 'Gender must be either male or female'
    }
  },
  height: {
    type: Number,
    required: [true, 'Height is required'],
    min: [30, 'Height must be at least 30 cm'],
    max: [200, 'Height cannot exceed 200 cm']
  },
  weight: {
    type: Number,
    required: [true, 'Weight is required'],
    min: [1, 'Weight must be at least 1 kg'],
    max: [100, 'Weight cannot exceed 100 kg']
  },
  headCircumference: {
    type: Number,
    required: [true, 'Head circumference is required'],
    min: [20, 'Head circumference must be at least 20 cm'],
    max: [60, 'Head circumference cannot exceed 60 cm']
  },
  createdAt: {
    type: Date,
    default: Date.now
  },
  updatedAt: {
    type: Date,
    default: Date.now
  }
});

// Update timestamps on save
childSchema.pre('save', function(next) {
  this.updatedAt = Date.now();
  next();
});
```

### Methods

```javascript
childSchema.methods = {
  async updateMeasurements(data) {
    const allowedUpdates = ['height', 'weight', 'headCircumference'];
    Object.keys(data).forEach(key => {
      if (allowedUpdates.includes(key)) {
        this[key] = data[key];
      }
    });
    this.updatedAt = Date.now();
    return this.save();
  },
  
  async getQuestionnaires() {
    return this.model('Questionnaire').find({ childId: this._id });
  },
  
  calculateZScore() {
    // Implementation of Z-score calculation based on WHO standards
    // This is a simplified example
    const age = this.calculateAge();
    const gender = this.gender;
    
    // Get reference data from WHO standards
    const referenceData = this.getReferenceData(age, gender);
    
    // Calculate Z-score
    const zScore = (this.height - referenceData.median) / referenceData.sd;
    
    return zScore;
  }
};

// Static methods
childSchema.statics = {
  async findByUserId(userId) {
    return this.find({ userId });
  }
};
```

## 📝 Questionnaire Model Implementation

### Schema Definition

```javascript
// models/Questionnaire.js
const mongoose = require('mongoose');
const { Schema } = mongoose;

const questionnaireSchema = new Schema({
  childId: {
    type: Schema.Types.ObjectId,
    ref: 'Child',
    required: [true, 'Child ID is required']
  },
  type: {
    type: String,
    required: [true, 'Questionnaire type is required'],
    enum: {
      values: ['initial', 'followup', 'monitoring'],
      message: 'Invalid questionnaire type'
    }
  },
  answers: [{
    questionId: {
      type: String,
      required: [true, 'Question ID is required']
    },
    answer: {
      type: String,
      required: [true, 'Answer is required']
    }
  }],
  score: {
    type: Number,
    required: [true, 'Score is required'],
    min: [0, 'Score cannot be negative'],
    max: [10, 'Score cannot exceed 10']
  },
  riskLevel: {
    type: String,
    required: [true, 'Risk level is required'],
    enum: {
      values: ['low', 'medium', 'high'],
      message: 'Invalid risk level'
    }
  },
  recommendations: [{
    type: String,
    required: [true, 'Recommendation is required']
  }],
  createdAt: {
    type: Date,
    default: Date.now
  }
});
```

### Methods

```javascript
questionnaireSchema.methods = {
  calculateScore() {
    // Calculate score based on answers
    this.score = this.answers.reduce((total, answer) => {
      return total + this.getQuestionWeight(answer.questionId);
    }, 0);
    return this.save();
  },
  
  determineRiskLevel() {
    // Determine risk level based on score
    if (this.score >= 8) {
      this.riskLevel = 'high';
    } else if (this.score >= 5) {
      this.riskLevel = 'medium';
    } else {
      this.riskLevel = 'low';
    }
    return this.save();
  },
  
  generateRecommendations() {
    // Generate recommendations based on risk level and answers
    this.recommendations = [];
    
    if (this.riskLevel === 'high') {
      this.recommendations.push(
        'Immediate medical consultation required',
        'Regular growth monitoring',
        'Nutritional assessment'
      );
    } else if (this.riskLevel === 'medium') {
      this.recommendations.push(
        'Regular growth monitoring',
        'Nutritional assessment',
        'Follow-up in 3 months'
      );
    } else {
      this.recommendations.push(
        'Regular growth monitoring',
        'Follow-up in 6 months'
      );
    }
    
    return this.save();
  }
};

// Static methods
questionnaireSchema.statics = {
  async findByChildId(childId) {
    return this.find({ childId }).sort({ createdAt: -1 });
  }
};
```

## 💬 Message Model Implementation

### Schema Definition

```javascript
// models/Message.js
const mongoose = require('mongoose');
const { Schema } = mongoose;

const messageSchema = new Schema({
  phone: {
    type: String,
    required: [true, 'Phone number is required'],
    trim: true,
    match: [/^62\d{9,12}$/, 'Please enter a valid Indonesian phone number']
  },
  type: {
    type: String,
    required: [true, 'Message type is required'],
    enum: {
      values: ['text', 'image', 'document'],
      message: 'Invalid message type'
    }
  },
  content: {
    type: String,
    required: [true, 'Message content is required']
  },
  direction: {
    type: String,
    required: [true, 'Message direction is required'],
    enum: {
      values: ['incoming', 'outgoing'],
      message: 'Invalid message direction'
    }
  },
  status: {
    type: String,
    required: [true, 'Message status is required'],
    enum: {
      values: ['sent', 'delivered', 'read', 'failed'],
      message: 'Invalid message status'
    },
    default: 'sent'
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});
```

### Methods

```javascript
messageSchema.methods = {
  async updateStatus(status) {
    this.status = status;
    return this.save();
  },
  
  async getHistory(limit = 50, offset = 0) {
    return this.model('Message').find({ phone: this.phone })
      .sort({ createdAt: -1 })
      .skip(offset)
      .limit(limit);
  }
};

// Static methods
messageSchema.statics = {
  async findByPhone(phone) {
    return this.find({ phone }).sort({ createdAt: -1 });
  }
};
```

## 🔄 Session Model Implementation

### Schema Definition

```javascript
// models/Session.js
const mongoose = require('mongoose');
const { Schema } = mongoose;

const sessionSchema = new Schema({
  userId: {
    type: Schema.Types.ObjectId,
    ref: 'User',
    required: [true, 'User ID is required']
  },
  type: {
    type: String,
    required: [true, 'Session type is required'],
    enum: {
      values: ['registration', 'questionnaire', 'consultation'],
      message: 'Invalid session type'
    }
  },
  state: {
    type: String,
    required: [true, 'Session state is required']
  },
  data: {
    type: Map,
    of: Schema.Types.Mixed
  },
  expiresAt: {
    type: Date,
    required: [true, 'Expiration date is required'],
    validate: {
      validator: function(v) {
        return v > new Date();
      },
      message: 'Expiration date must be in the future'
    }
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});
```

### Methods

```javascript
sessionSchema.methods = {
  async updateState(state, data = {}) {
    this.state = state;
    if (data) {
      Object.entries(data).forEach(([key, value]) => {
        this.data.set(key, value);
      });
    }
    this.expiresAt = new Date(Date.now() + 24 * 60 * 60 * 1000);
    return this.save();
  },
  
  isExpired() {
    return Date.now() > this.expiresAt;
  },
  
  getData(key) {
    return this.data.get(key);
  },
  
  setData(key, value) {
    this.data.set(key, value);
    return this.save();
  }
};

// Static methods
sessionSchema.statics = {
  async findByUserId(userId) {
    return this.find({ userId }).sort({ createdAt: -1 });
  },
  
  async findActiveByUserId(userId) {
    return this.find({
      userId,
      expiresAt: { $gt: new Date() }
    }).sort({ createdAt: -1 });
  }
};
```
