# Questionnaire System

## Overview

Sistem kuesioner dirancang untuk mengumpulkan data komprehensif tentang kondisi anak yang dapat membantu dalam identifikasi risiko stunting. Sistem ini mencakup berbagai jenis kuesioner yang disesuaikan dengan usia dan kebutuhan anak.

## Jenis Kuesioner

1. **Kuesioner Bayi (0-2 tahun)**
   - Perkembangan motorik
   - Pola makan
   - Kesehatan umum

2. **Kuesioner Anak (2-5 tahun)**
   - Perkembangan kognitif
   - Pola makan
   - Aktivitas fisik

3. **Kuesioner Kognitif**
   - Kemampuan bahasa
   - Kemampuan motorik
   - Kemampuan sosial

4. **Kuesioner Pola Asuh**
   - Praktik pengasuhan
   - Lingkungan rumah
   - Akses layanan kesehatan

## Struktur Kuesioner

### Format Pertanyaan

```json
{
  "id": "String",
  "kategori": "String",
  "pertanyaan": "String",
  "tipe_jawaban": "String",
  "opsi_jawaban": ["Array"],
  "skor": "Number",
  "keterangan": "String"
}
```

### Format Jawaban

```json
{
  "id": "String",
  "kuesioner_id": "String",
  "anak_id": "String",
  "jawaban": "String",
  "skor": "Number",
  "tanggal": "Date",
  "created_at": "Timestamp"
}
```

## Alur Pengisian

1. **Inisiasi**
   - Pilih jenis kuesioner
   - Verifikasi data anak
   - Konfirmasi kesiapan

2. **Proses Pengisian**
   - Tampilkan pertanyaan
   - Input jawaban
   - Validasi input

3. **Penyelesaian**
   - Perhitungan skor
   - Analisis hasil
   - Rekomendasi

## Perhitungan Skor

### Formula

```javascript
function calculateScore(answers) {
  // Implementasi perhitungan skor
  // Berdasarkan bobot pertanyaan
}
```

### Kategori Hasil

- Normal: 0-20
- Risiko Rendah: 21-40
- Risiko Sedang: 41-60
- Risiko Tinggi: >60

## Integrasi

### API Endpoints

```http
POST /api/v1/questionnaires
Content-Type: application/json

{
  "anak_id": "123456",
  "jenis_kuesioner": "bayi",
  "jawaban": [
    {
      "pertanyaan_id": "Q001",
      "jawaban": "Ya"
    }
  ]
}
```

### Response

```json
{
  "status": "success",
  "data": {
    "skor_total": 45,
    "kategori": "Risiko Sedang",
    "rekomendasi": [
      "Konsultasi dengan dokter",
      "Perbaikan pola makan"
    ]
  }
}
```

## Validasi

### Kriteria Validasi

1. **Input**
   - Format jawaban valid
   - Kelengkapan data
   - Konsistensi jawaban

2. **Proses**
   - Urutan pertanyaan
   - Waktu pengisian
   - Duplikasi jawaban

## Notifikasi

### Jenis Notifikasi

1. **Pengingat**
   - Jadwal kuesioner
   - Kuesioner belum selesai
   - Update hasil

2. **Alert**
   - Hasil abnormal
   - Rekomendasi tindakan
   - Follow-up

## Keamanan

1. **Akses**
   - Autentikasi pengguna
   - Enkripsi data
   - Audit log

2. **Validasi**
   - Input sanitasi
   - Pengecekan duplikasi
   - Verifikasi data

## Monitoring

### Metrics

- Jumlah kuesioner diisi
- Waktu pengisian
- Distribusi hasil
- Tingkat penyelesaian

### Alerts

- Hasil abnormal
- Keterlambatan pengisian
- Anomali data

## Maintenance

### Backup

- Backup data kuesioner
- Backup hasil analisis
- Backup konfigurasi

### Updates

- Pembaruan pertanyaan
- Penambahan kategori
- Perubahan skoring

## Best Practices

1. **User Experience**
   - Pertanyaan jelas
   - Navigasi mudah
   - Feedback instan

2. **Data Quality**
   - Validasi real-time
   - Konsistensi data
   - Dokumentasi perubahan

3. **Privacy**
   - Enkripsi data
   - Akses terbatas
   - Retention policy 