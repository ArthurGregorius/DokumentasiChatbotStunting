# Models

## 📋 Overview

Dokumentasi ini menjelaskan model-model data yang digunakan dalam API Chatbot Identifikasi Stunting, termasuk struktur data, validasi, dan relasi antar model.

## 👤 User Model

### Schema Definition

```javascript
// models/User.js
const userSchema = new Schema({
  name: {
    type: String,
    required: true,
    trim: true
  },
  phone: {
    type: String,
    required: true,
    unique: true,
    trim: true
  },
  address: {
    type: String,
    required: true,
    trim: true
  },
  birthDate: {
    type: Date,
    required: true
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
```

### Methods

```javascript
userSchema.methods = {
  async updateProfile(data) {
    Object.assign(this, data);
    this.updatedAt = Date.now();
    return this.save();
  },
  
  async getChildren() {
    return Child.find({ userId: this._id });
  }
};
```

## 👶 Child Model

### Schema Definition

```javascript
// models/Child.js
const childSchema = new Schema({
  userId: {
    type: Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  name: {
    type: String,
    required: true,
    trim: true
  },
  birthDate: {
    type: Date,
    required: true
  },
  gender: {
    type: String,
    enum: ['male', 'female'],
    required: true
  },
  height: {
    type: Number,
    required: true
  },
  weight: {
    type: Number,
    required: true
  },
  headCircumference: {
    type: Number,
    required: true
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
```

### Methods

```javascript
childSchema.methods = {
  async updateMeasurements(data) {
    Object.assign(this, data);
    this.updatedAt = Date.now();
    return this.save();
  },
  
  async getQuestionnaires() {
    return Questionnaire.find({ childId: this._id });
  }
};
```

## 📝 Questionnaire Model

### Schema Definition

```javascript
// models/Questionnaire.js
const questionnaireSchema = new Schema({
  childId: {
    type: Schema.Types.ObjectId,
    ref: 'Child',
    required: true
  },
  type: {
    type: String,
    enum: ['initial', 'followup', 'monitoring'],
    required: true
  },
  answers: [{
    questionId: {
      type: String,
      required: true
    },
    answer: {
      type: String,
      required: true
    }
  }],
  score: {
    type: Number,
    required: true
  },
  riskLevel: {
    type: String,
    enum: ['low', 'medium', 'high'],
    required: true
  },
  recommendations: [{
    type: String
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
  }
};
```

## 💬 Message Model

### Schema Definition

```javascript
// models/Message.js
const messageSchema = new Schema({
  phone: {
    type: String,
    required: true,
    trim: true
  },
  type: {
    type: String,
    enum: ['text', 'image', 'document'],
    required: true
  },
  content: {
    type: String,
    required: true
  },
  direction: {
    type: String,
    enum: ['incoming', 'outgoing'],
    required: true
  },
  status: {
    type: String,
    enum: ['sent', 'delivered', 'read', 'failed'],
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
    return Message.find({ phone: this.phone })
      .sort({ createdAt: -1 })
      .skip(offset)
      .limit(limit);
  }
};
```

## 🔄 Session Model

### Schema Definition

```javascript
// models/Session.js
const sessionSchema = new Schema({
  userId: {
    type: Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  type: {
    type: String,
    enum: ['registration', 'questionnaire', 'consultation'],
    required: true
  },
  state: {
    type: String,
    required: true
  },
  data: {
    type: Map,
    of: Schema.Types.Mixed
  },
  expiresAt: {
    type: Date,
    required: true
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
  }
};
```
