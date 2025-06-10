# 💻 Development Guide

<div align="center">

![Development](https://img.shields.io/badge/Development-Guide-blue)
![Status](https://img.shields.io/badge/Status-Active-green)

</div>

## 📋 Overview

Panduan ini menjelaskan cara mengembangkan dan berkontribusi pada proyek Chatbot Identifikasi Stunting.

## 🚀 Setup Development Environment

### Prerequisites
- Node.js v18.x atau lebih baru
- npm atau yarn
- Git
- WhatsApp Business API access
- Google Apps Script project

### Installation Steps
1. Clone repository
```bash
git clone https://github.com/your-username/chatbot-stunting.git
cd chatbot-stunting
```

2. Install dependencies
```bash
npm install
```

3. Setup environment variables
```bash
cp .env.example .env
# Edit .env dengan konfigurasi yang sesuai
```

4. Setup Google Apps Script
- Buat project baru di Google Apps Script
- Copy script dari `AppScript.gs`
- Deploy sebagai web app
- Update `DEPLOYMENT_ID` di `.env`

## 🧪 Testing

### Unit Testing
```bash
npm run test
```

### Integration Testing
```bash
npm run test:integration
```

### E2E Testing
```bash
npm run test:e2e
```

## 📝 Code Style Guide

### JavaScript/Node.js
- Gunakan ESLint dan Prettier
- Ikuti style guide dari Airbnb
- Gunakan async/await untuk asynchronous code
- Dokumentasikan fungsi dengan JSDoc

### Contoh Kode
```javascript
/**
 * Register new user
 * @param {Object} userData - User data
 * @returns {Promise<Object>} Registered user
 */
async function registerUser(userData) {
  try {
    const user = await userService.create(userData);
    return user;
  } catch (error) {
    logger.error('Failed to register user:', error);
    throw error;
  }
}
```

## 🔄 Git Workflow

1. **Branch Naming**
   - feature/feature-name
   - bugfix/bug-description
   - hotfix/issue-description

2. **Commit Messages**
   - feat: add new feature
   - fix: fix bug
   - docs: update documentation
   - style: format code
   - refactor: refactor code
   - test: add tests
   - chore: update dependencies

3. **Pull Request Process**
   - Buat branch baru
   - Implementasi perubahan
   - Jalankan tests
   - Update dokumentasi
   - Buat pull request
   - Review dan merge

## 🛠️ Development Tools

### VS Code Extensions
- ESLint
- Prettier
- GitLens
- Node.js Extension Pack

### Chrome Extensions
- Postman
- Redux DevTools
- React Developer Tools

## 📚 Dokumentasi Terkait

- [Local Setup](local-setup.md)
- [Testing Guide](testing.md)
- [Code Style](code-style.md)

---

<div align="center">

### 🔗 Navigasi

[⬅️ Kembali ke Technical Implementation](../implementation/README.md) | [Lanjut ke Deployment ➡️](../deployment/README.md)

</div> 