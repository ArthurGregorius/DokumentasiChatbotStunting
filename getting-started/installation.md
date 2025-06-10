# Installation

Berikut adalah panduan langkah demi langkah untuk menginstal Chatbot Identifikasi Stunting:

## 1. Clone Repository

```bash
git clone https://github.com/ArthurGregorius/ChatbotStunting.git
cd ChatbotStunting
```

## 2. Install Dependencies

```bash
npm install
```

## 3. Konfigurasi Environment

1. Buat file `.env` di root direktori:

```bash
cp .env.example .env
```

2. Edit file `.env` dengan konfigurasi yang sesuai:

```env
# WhatsApp Configuration
WHATSAPP_NUMBER=your_whatsapp_number
WHATSAPP_API_KEY=your_api_key

# API Configuration
API_BASE_URL=your_api_base_url
API_KEY=your_api_key

# Database Configuration
DB_HOST=your_db_host
DB_USER=your_db_user
DB_PASSWORD=your_db_password
DB_NAME=your_db_name

# Security
JWT_SECRET=your_jwt_secret
```

## 4. Setup Database

1. Buat database baru:

```sql
CREATE DATABASE chatbot_stunting;
```

2. Import skema database:

```bash
mysql -u your_username -p chatbot_stunting < database/schema.sql
```

## 5. Setup WhatsApp

1. Pastikan nomor WhatsApp yang digunakan sudah terdaftar
2. Scan QR code yang muncul saat menjalankan aplikasi
3. Tunggu hingga status koneksi berubah menjadi "connected"

## 6. Verifikasi Instalasi

1. Jalankan aplikasi:

```bash
npm start
```

2. Periksa log untuk memastikan tidak ada error
3. Kirim pesan "hi" ke nomor WhatsApp bot
4. Pastikan bot merespons dengan benar

## 7. Setup Docker (Opsional)

1. Build Docker image:

```bash
docker build -t chatbot-stunting .
```

2. Jalankan container:

```bash
docker run -d --name chatbot-stunting -p 3000:3000 chatbot-stunting
```

## Troubleshooting

### Masalah Umum

1. **Error Koneksi WhatsApp**
   * Pastikan nomor WhatsApp valid
   * Periksa koneksi internet
   * Restart aplikasi
2. **Error Database**
   * Periksa kredensial database
   * Pastikan database server berjalan
   * Periksa firewall settings
3. **Error API**
   * Verifikasi API key
   * Periksa URL endpoint
   * Pastikan server API berjalan

### Logging

* Log aplikasi tersedia di `logs/app.log`
* Log error tersedia di `logs/error.log`

## Next Steps

Setelah instalasi selesai, Anda dapat:

1. Mengkonfigurasi fitur tambahan
2. Menyesuaikan pesan bot
3. Mengatur monitoring
4. Melakukan backup regular

## Support

Jika mengalami masalah saat instalasi, silakan:

1. Periksa dokumentasi troubleshooting
2. Hubungi tim support
3. Buat issue di repository
