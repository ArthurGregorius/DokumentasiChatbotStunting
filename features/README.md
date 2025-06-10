---
icon: diagram-previous
cover: >-
  https://images.unsplash.com/photo-1563089145-599997674d42?crop=entropy&cs=srgb&fm=jpg&ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHw1fHxhYnN0cmFjdHxlbnwwfHx8fDE3NDk1NDcwNjJ8MA&ixlib=rb-4.1.0&q=85
coverY: 0
---

# Features

## 📋 Overview

Dokumentasi ini menjelaskan fitur-fitur utama dari Chatbot Identifikasi Stunting, termasuk cara kerja dan interaksi antar fitur.

## 🎯 Core Features

### 1. User Management

* Registrasi pengguna
* Manajemen profil
* Autentikasi
* Manajemen sesi

### 2. Child Data Management

* Pencatatan data anak
* Riwayat pertumbuhan
* Pengukuran antropometri
* Analisis perkembangan

### 3. Questionnaire System

* Kuesioner stunting
* Skoring otomatis
* Rekomendasi
* Riwayat penilaian

### 4. WhatsApp Integration

* Chat interaktif
* Notifikasi
* Pengiriman media
* Template pesan

## 🔄 Feature Workflows

### 1. User Registration Flow

```mermaid
sequenceDiagram
    User->>WhatsApp: Kirim pesan registrasi
    WhatsApp->>Backend: Forward pesan
    Backend->>Database: Simpan data user
    Database->>Backend: Konfirmasi
    Backend->>WhatsApp: Kirim konfirmasi
    WhatsApp->>User: Terima konfirmasi
```

### 2. Child Assessment Flow

```mermaid
sequenceDiagram
    User->>WhatsApp: Minta penilaian
    WhatsApp->>Backend: Forward request
    Backend->>Database: Ambil data anak
    Database->>Backend: Kirim data
    Backend->>WhatsApp: Kirim kuesioner
    WhatsApp->>User: Isi kuesioner
```

## 📊 Feature Details

### User Registration

<details>

<summary>🔍 Detail Registrasi User</summary>

**Proses Registrasi**

1. User mengirim pesan "DAFTAR"
2. Bot meminta data:
   * Nama lengkap
   * Nomor telepon
   * Alamat
   * Tanggal lahir
3. Data divalidasi
4. Akun dibuat
5. Konfirmasi dikirim

</details>

### Child Management

<details>

<summary>🔍 Detail Manajemen Anak</summary>

**Fitur Utama**

1. Tambah data anak
2. Update data
3. Lihat riwayat
4. Analisis pertumbuhan
5. Rekomendasi

</details>

### Questionnaire System

<details>

<summary>🔍 Detail Sistem Kuesioner</summary>

**Jenis Kuesioner**

1. Kuesioner Awal
2. Kuesioner Lanjutan
3. Kuesioner Monitoring
4. Kuesioner Evaluasi

</details>

### WhatsApp Integration

<details>

<summary>🔍 Detail Integrasi WhatsApp</summary>

**Fitur Chat**

1. Pesan teks
2. Pesan media
3. Template pesan
4. Quick replies
5. Buttons

</details>

## 🎨 User Interface

### 1. WhatsApp Interface

* Menu interaktif
* Tombol cepat
* Template pesan
* Media sharing

### 2. Admin Interface

* Dashboard
* Manajemen user
* Laporan
* Konfigurasi

## 📱 Mobile Experience

### 1. PWA Features

* Offline support
* Push notifications
* Home screen install
* Responsive design

### 2. Mobile Optimization

* Fast loading
* Touch friendly
* Battery efficient
* Data saving
