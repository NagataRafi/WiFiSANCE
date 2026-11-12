# Architecture Overview

## Arsitektur inti WiFiSENSE

Sistem WiFiSENSE dirancang untuk memanfaatkan sinyal Wi-Fi dari dua ESP32 sebagai komponen utama. Arsitektur yang ditekankan dalam PRD adalah:

```text
ESP32 TX
   |
   | Sinyal Wi-Fi / paket
   v
Area Pengamatan
   |
   v
ESP32 RX
   |
   v
CSI Acquisition & Logger
   |
   v
CSI Parser
   |
   v
Preprocessing / DSP
   |
   v
Feature Extraction
   |
   v
Machine Learning Inference
   |
   v
Backend REST API
   |
   +------ Database / storage
   |
   v
Mobile App / Dashboard
```

## Komponen utama

### 1. ESP32 TX

ESP32 transmitter bertugas memancarkan sinyal Wi-Fi sesuai konfigurasi yang benar-benar diuji pada board yang digunakan. Peran ini tetap menjadi satu-satunya perangkat transmisi utama dalam arsitektur inti.

### 2. ESP32 RX

ESP32 receiver bertugas menangkap paket Wi-Fi dan mengakses data CSI yang dihasilkan oleh perangkat penerima. Data ini kemudian dicatat bersama metadata seperti waktu, sesi, dan konfigurasi pengujian.

### 3. CSI Acquisition & Logger

Modul ini berfungsi untuk menyimpan sampel CSI beserta metadata yang relevan. Pada tahap awal, format output harus tetap konsisten dengan sumber data mentah dan tidak diubah tanpa dokumentasi.

### 4. Parser dan validasi

Data CSI mentah diparsing dan divalidasi sebelum menuju preprocessing. Tujuan dari validasi ini adalah memastikan bahwa data masuk ke pipeline memiliki struktur minimal yang dapat diolah dan didokumentasikan.

### 5. DSP dan preprocessing

Setelah data valid, pipeline akan membersihkan dan memperhalus sinyal dengan teknik filtering, normalisasi, dan segmentasi yang sesuai kebutuhan. Rincian spesifik tetap harus ditentukan dari hasil eksperimen dan tidak boleh diandaikan.

### 6. Feature extraction

Fitur-fitur dari CSI akan diekstraksi untuk menjadi representasi yang lebih baik bagi model. Fitur yang dipilih bergantung pada hasil eksperimen, kebutuhan label, dan kesesuaian data.

### 7. Machine Learning inference

Model akan belajar dari data berlabel jumlah manusia dan menghasilkan estimasi. Target utama model adalah jumlah manusia, bukan hanya deteksi keberadaan.

### 8. Backend dan aplikasi mobile

Hasil estimasi dikirim ke backend agar dapat disimpan dan dipakai untuk dashboard aplikasi mobile. Tujuan utama pada fase awal adalah menampilkan estimasi terbaru dan riwayat yang sederhana.

## Batasan penting

- HC-SR04 tidak termasuk dalam arsitektur inti.
- Komputer/laptop dapat berperan sebagai logging, preprocessing, training, backend, dan pengembangan aplikasi.
- Integrasi cloud atau deployment tidak dianggap wajib pada MVP.
- Rencana arsitektur ini masih memerlukan validasi hardware dan konfigurasi firmware. Segala keputusan yang belum dikonfirmasi ditandai `TBD`.
