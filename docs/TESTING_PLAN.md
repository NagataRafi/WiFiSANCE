# Testing Plan

Dokumen ini merangkum rencana pengujian WiFiSENSE sesuai PRD. Fokus utama adalah untuk membuktikan bahwa setiap modul dapat dipahami, diulang, dan terdokumentasi.

## 1. Pengujian hardware

### Tujuan

Memastikan dua ESP32 dapat berperan sebagai TX dan RX dengan konfigurasi yang tervalidasi.

### Yang diuji

- komunikasi dasar TX–RX;
- stabilitas koneksi bila diperlukan;
- kemampuan receiver untuk menghasilkan data CSI;
- karakteristik board dan toolchain yang digunakan.

### Hasil yang diharapkan

- status komunikasi tercatat;
- data CSI yang valid dapat disimpan dengan metadata dasar;
- keterbatasan hardware terdokumentasi.

## 2. Pengujian akuisisi CSI

### Tujuan

Memastikan CSI dapat direkam dengan format yang konsisten.

### Yang diuji

- keberhasilan perekaman data dari receiver;
- kelengkapan metadata;
- kualitas payload CSI serta integritas data;
- hubungan antara sesi, waktu, dan konfigurasi perangkat.

### Hasil yang diharapkan

- file/log CSI tersedia di `data/raw/`;
- data memiliki descriptor sesi yang jelas;
- file tidak bercampur antara raw, processed, dan labeled.

## 3. Pengujian DSP

### Tujuan

Memastikan parsing, validasi, preprocessing, dan feature extraction berjalan secara berulang.

### Yang diuji

- parser dapat membaca raw CSI dengan benar;
- validasi mendeteksi data gagal atau tidak konsisten;
- preprocessing tidak merusak data mentah;
- feature extraction menghasilkan output yang dapat dilanjutkan ke model.

### Hasil yang diharapkan

- pipeline memproses data dengan output yang terdokumentasi;
- semua langkah tersusun modular dan traceable.

## 4. Pengujian model

### Tujuan

Menilai performa model baseline untuk estimasi jumlah manusia.

### Yang diuji

- kesesuaian dataset dan label;
- pembagian train/validation/test yang aman;
- metrik performa seperti accuracy, precision, recall, F1-score, MAE, RMSE;
- konsistensi hasil antar sesi eksperimen.

### Hasil yang diharapkan

- model menghasilkan prediksi dan metrik evaluasi yang dicatat;
- keterbatasan model terdokumentasi secara jujur.

## 5. Pengujian backend

### Tujuan

Memastikan hasil prediksi dapat diterima, disimpan, dan diakses melalui API sederhana.

### Yang diuji

- format request/response;
- validasi input;
- penyimpanan hasil prediksi;
- status error, log, dan metadata.

## 6. Pengujian end-to-end

### Tujuan

Menilai integrasi komprehensif dari akuisisi CSI hingga hasil visualisasi.

### Yang diuji

- alur data dari TX/RX ke parser;
- pipeline DSP sampai prediksi;
- pengiriman hasil ke backend;
- tampilan pada aplikasi mobile/dashboard.

### Hasil yang diharapkan

- alur end-to-end dijalankan atau dianggap belum siap dengan catatan yang jelas;
- keterbatasan, kegagalan, dan data yang tidak direproduksi tetap dilaporkan.

## 7. Prinsip pengujian

- Jangan mengklaim hasil yang belum diuji.
- Simpan log, error, dan metadata untuk setiap pengujian.
- Pisahkan data latihan dan data evaluasi berdasarkan sesi bila perlu.
- Dokumentasikan semua parameter dan lingkungan eksperimen.

Pada tahap awal, pengujian fokus pada bukti kelayakan dan dokumentasi integritas, bukan pada klaim performa final.
