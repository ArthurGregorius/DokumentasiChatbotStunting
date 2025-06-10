# API Integration

## Overview

Dokumentasi ini menjelaskan integrasi API chatbot dengan layanan eksternal, termasuk WhatsApp, Google Sheets, dan layanan lainnya.

## WhatsApp Integration

### 1. Connection Setup

```javascript
const whatsappIntegration = {
  async connect() {
    try {
      const sock = makeWASocket({
        printQRInTerminal: true,
        auth: state,
        browser: ['Chatbot Stunting', 'Chrome', '1.0.0']
      });
      
      // Handle connection updates
      sock.ev.on('connection.update', async (update) => {
        const { connection, lastDisconnect } = update;
        
        if (connection === 'close') {
          const shouldReconnect = lastDisconnect?.error?.output?.statusCode !== 401;
          
          if (shouldReconnect) {
            await this.connect();
          }
        }
      });
      
      // Handle incoming messages
      sock.ev.on('messages.upsert', async (m) => {
        if (m.type === 'notify') {
          for (const msg of m.messages) {
            await this.handleIncomingMessage(msg);
          }
        }
      });
      
      return sock;
    } catch (error) {
      logger.error('WhatsApp connection error:', error);
      throw error;
    }
  }
};
```

### 2. Message Handling

```javascript
const whatsappIntegration = {
  async sendMessage(phoneNumber, content) {
    try {
      if (!this.sock) {
        await this.connect();
      }
      
      const message = {
        text: content
      };
      
      await this.sock.sendMessage(phoneNumber, message);
      
      return true;
    } catch (error) {
      logger.error('Send message error:', error);
      throw error;
    }
  },
  
  async handleIncomingMessage(msg) {
    try {
      const { from, message } = msg;
      
      // Create message record
      const messageRecord = new Message({
        userId: await this.getUserIdByPhone(from),
        type: this.getMessageType(message),
        content: this.getMessageContent(message),
        direction: 'incoming'
      });
      
      await messageRecord.save();
      
      // Process message
      await this.processMessage(messageRecord);
    } catch (error) {
      logger.error('Handle incoming message error:', error);
      throw error;
    }
  }
};
```

## Google Sheets Integration

### 1. API Setup

```javascript
const sheetsIntegration = {
  async setup() {
    try {
      const auth = new google.auth.GoogleAuth({
        keyFile: process.env.GOOGLE_APPLICATION_CREDENTIALS,
        scopes: ['https://www.googleapis.com/auth/spreadsheets']
      });
      
      const client = await auth.getClient();
      const sheets = google.sheets({ version: 'v4', auth: client });
      
      return sheets;
    } catch (error) {
      logger.error('Google Sheets setup error:', error);
      throw error;
    }
  }
};
```

### 2. Data Operations

```javascript
const sheetsIntegration = {
  async appendRow(sheetId, range, values) {
    try {
      const sheets = await this.setup();
      
      const response = await sheets.spreadsheets.values.append({
        spreadsheetId: sheetId,
        range,
        valueInputOption: 'USER_ENTERED',
        resource: {
          values: [values]
        }
      });
      
      return response.data;
    } catch (error) {
      logger.error('Append row error:', error);
      throw error;
    }
  },
  
  async getData(sheetId, range) {
    try {
      const sheets = await this.setup();
      
      const response = await sheets.spreadsheets.values.get({
        spreadsheetId: sheetId,
        range
      });
      
      return response.data.values;
    } catch (error) {
      logger.error('Get data error:', error);
      throw error;
    }
  }
};
```

## Email Integration

### 1. SMTP Setup

```javascript
const emailIntegration = {
  async setup() {
    try {
      const transporter = nodemailer.createTransport({
        host: process.env.SMTP_HOST,
        port: process.env.SMTP_PORT,
        secure: process.env.SMTP_SECURE === 'true',
        auth: {
          user: process.env.SMTP_USER,
          pass: process.env.SMTP_PASS
        }
      });
      
      return transporter;
    } catch (error) {
      logger.error('SMTP setup error:', error);
      throw error;
    }
  }
};
```

### 2. Email Operations

```javascript
const emailIntegration = {
  async sendEmail(to, subject, content) {
    try {
      const transporter = await this.setup();
      
      const mailOptions = {
        from: process.env.SMTP_FROM,
        to,
        subject,
        html: content
      };
      
      const info = await transporter.sendMail(mailOptions);
      logger.info('Email sent:', info);
      
      return info;
    } catch (error) {
      logger.error('Send email error:', error);
      throw error;
    }
  },
  
  async sendBulkEmail(recipients, subject, content) {
    try {
      const transporter = await this.setup();
      
      const results = await Promise.allSettled(
        recipients.map(recipient =>
          transporter.sendMail({
            from: process.env.SMTP_FROM,
            to: recipient.email,
            subject,
            html: this.personalizeContent(content, recipient)
          })
        )
      );
      
      return {
        success: results.filter(r => r.status === 'fulfilled').length,
        failed: results.filter(r => r.status === 'rejected').length
      };
    } catch (error) {
      logger.error('Send bulk email error:', error);
      throw error;
    }
  }
};
```

## SMS Integration

### 1. SMS Gateway Setup

```javascript
const smsIntegration = {
  async setup() {
    try {
      const client = new Twilio(
        process.env.TWILIO_ACCOUNT_SID,
        process.env.TWILIO_AUTH_TOKEN
      );
      
      return client;
    } catch (error) {
      logger.error('SMS gateway setup error:', error);
      throw error;
    }
  }
};
```

### 2. SMS Operations

```javascript
const smsIntegration = {
  async sendSMS(to, message) {
    try {
      const client = await this.setup();
      
      const result = await client.messages.create({
        body: message,
        to,
        from: process.env.TWILIO_PHONE_NUMBER
      });
      
      return result;
    } catch (error) {
      logger.error('Send SMS error:', error);
      throw error;
    }
  },
  
  async sendBulkSMS(recipients, message) {
    try {
      const client = await this.setup();
      
      const results = await Promise.allSettled(
        recipients.map(recipient =>
          client.messages.create({
            body: this.personalizeMessage(message, recipient),
            to: recipient.phone,
            from: process.env.TWILIO_PHONE_NUMBER
          })
        )
      );
      
      return {
        success: results.filter(r => r.status === 'fulfilled').length,
        failed: results.filter(r => r.status === 'rejected').length
      };
    } catch (error) {
      logger.error('Send bulk SMS error:', error);
      throw error;
    }
  }
};
```

## Payment Integration

### 1. Payment Gateway Setup

```javascript
const paymentIntegration = {
  async setup() {
    try {
      const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);
      return stripe;
    } catch (error) {
      logger.error('Payment gateway setup error:', error);
      throw error;
    }
  }
};
```

### 2. Payment Operations

```javascript
const paymentIntegration = {
  async createPaymentIntent(amount, currency = 'idr') {
    try {
      const stripe = await this.setup();
      
      const paymentIntent = await stripe.paymentIntents.create({
        amount,
        currency,
        payment_method_types: ['card']
      });
      
      return paymentIntent;
    } catch (error) {
      logger.error('Create payment intent error:', error);
      throw error;
    }
  },
  
  async confirmPayment(paymentIntentId) {
    try {
      const stripe = await this.setup();
      
      const paymentIntent = await stripe.paymentIntents.confirm(
        paymentIntentId
      );
      
      return paymentIntent;
    } catch (error) {
      logger.error('Confirm payment error:', error);
      throw error;
    }
  }
};
```

## Best Practices

1. **Integration Setup**
   - Secure credentials
   - Error handling
   - Retry mechanism
   - Circuit breaker

2. **Data Handling**
   - Data validation
   - Data transformation
   - Error recovery
   - Data backup

3. **Performance**
   - Connection pooling
   - Request batching
   - Response caching
   - Timeout handling

4. **Monitoring**
   - Health checks
   - Error tracking
   - Usage metrics
   - Performance monitoring

5. **Security**
   - API key management
   - Data encryption
   - Access control
   - Audit logging 