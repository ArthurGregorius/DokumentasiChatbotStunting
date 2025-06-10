---
icon: lightbulb-exclamation-on
cover: >-
  https://images.unsplash.com/photo-1567095761054-7a02e69e5c43?crop=entropy&cs=srgb&fm=jpg&ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHwyfHxhYnN0cmFjdHxlbnwwfHx8fDE3NDk1NDcwNjJ8MA&ixlib=rb-4.1.0&q=85
coverY: 0
---

# Getting Started

## 📋 Overview

Panduan ini akan membantu Anda memulai dengan Chatbot Identifikasi Stunting. Ikuti langkah-langkah di bawah ini untuk mengatur dan menjalankan aplikasi.

## 📋 Prerequisites

Sebelum memulai, pastikan Anda memiliki:

* Node.js v18.x atau lebih baru
* npm atau yarn
* Git
* WhatsApp Business API access
* Google Apps Script project
* Docker (opsional)

## 🛠️ Installation

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

## ⚙️ Configuration

Setelah instalasi, Anda perlu mengkonfigurasi beberapa komponen:

1. **WhatsApp API**
   * Dapatkan API key dari WhatsApp Business API
   * Konfigurasi webhook URL
   * Setup phone number
2. **Google Apps Script**
   * Buat project baru
   * Deploy script
   * Dapatkan deployment ID
3. **Environment Variables**
   * Konfigurasi database
   * Setup API keys
   * Konfigurasi logging

## 🚀 Quick Start

1. Start aplikasi

```bash
npm start
```

2. Test koneksi WhatsApp

```bash
npm run test:whatsapp
```

3. Verifikasi Google Apps Script

```bash
npm run test:gscript
```

## 📚 Dokumentasi Terkait

* [Prerequisites](prerequisites.md)
* [Installation](installation.md)
* [Configuration](configuration.md)

## 🔍 Troubleshooting

Jika Anda mengalami masalah saat setup, silakan cek:

* [Common Issues](../troubleshooting/common-issues.md)
* [FAQ](../troubleshooting/faq.md)

## 🤝 Support

Untuk bantuan lebih lanjut:

* Buka issue di GitHub
* Hubungi tim support
* Kunjungi forum diskusi
