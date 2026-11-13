# Roadmap WiFiSENSE

Dokumen ini merangkum roadmap tiga minggu yang sesuai dengan PRD. Status menggunakan kategori: `Not Started`, `In Progress`, dan `Completed`.

## Minggu 1: Hardware dan CSI Proof of Concept

| Item kerja | Status | Milestone | Acceptance criteria |
|---|---|---|---|
| Identifikasi tipe/varian ESP32 TX dan RX | In Progress | Verifikasi hardware dasar | Model board dan kapasitas komunikasi tercatat |
| Pemilihan toolchain dan konfigurasi firmware | In Progress | Toolchain yang akan dipakai disetujui tim | Toolchain ditulis di dokumentasi dan status siap diuji |
| Menjalankan TX dan RX | Not Started | Komunikasi dasar berhasil | Data log/status komunikasi tersedia |
| Proof of concept CSI | Not Started | CSI dapat diambil dari receiver | File/log CSI dapat dibaca dan didokumentasikan |
| Audit repository referensi | In Progress | Repository yang relevan dicatat | Daftar repository, modul, lisensi, dan catatan ketersediaan terdokumentasi |
| Simpan contoh data mentah dan format awal | Not Started | Format awal raw data dibuat | Data mentah tersimpan beserta metadata dasar |

### Milestone Minggu 1

- Verifikasi dua ESP32 sebagai komponen inti sistem.
- Dokumen toolchain awal dan konfigurasi komunikasi jenis `TBD` bila belum tersedia.
- CSI berhasil dihasilkan dan dijaga agar tidak dipaksakan sebelum validasi.

### Acceptance Criteria Minggu 1

- Receiver mampu menghasilkan log atau file CSI yang dapat dibaca.
- Struktur data mentah tercatat dan dipisahkan dari hasil preprocessing.
- Status hardware dan keterbatasan terdokumentasi secara jujur.

## Minggu 2: DSP, Dataset, dan AI Model

| Item kerja | Status | Milestone | Acceptance criteria |
|---|---|---|---|
| Parser CSI | Not Started | Parser dapat membaca format data yang valid | File parsing dapat diproses dengan validasi sederhana |
| Validasi data | Not Started | Data tidak valid teridentifikasi | Pencatatan error dan anomaly tersedia |
| Preprocessing dan filtering | Not Started | Data bersih siap diproses | Parameter preprocessing terdokumentasi |
| Ekstraksi fitur | Not Started | Fitur CSI berhasil dibuat | Fitur dapat disimpan dan diakses untuk training |
| Pengumpulan dataset berlabel | Not Started | Dataset ground truth tersedia | Setiap sesi mencatat jumlah orang aktual, waktu, dan konfigurasi |
| Integrasi dan audit dataset referensi WiCount | Completed | Dataset publik tersedia untuk eksplorasi awal | Arsip LFS terverifikasi, schema/label/keterbatasan dicatat; bukan pengganti dataset ESP32 WiFiSENSE |
| Visualisasi CSI awal dari WiCount | Completed | Notebook eksplorasi dapat dijalankan | Notebook menggunakan CSV asli untuk 2D dan 3D; gambar output tersimpan |
| Analisis mendalam seluruh rekaman WiCount | Completed | Statistik sesi/kelas dan kualitas CSI terdokumentasi | Seluruh 15 CSV dianalisis, visualisasi lintas sesi dibuat, risiko leakage dicatat; tanpa training model |
| Model baseline | Not Started | Baseline model siap untuk prediksi | Metrik evaluasi awal tersedia |

### Milestone Minggu 2

- Pipeline dari data CSI menuju fitur dan prediksi baseline berfungsi secara berulang.
- Dataset mencakup label jumlah manusia yang terdokumentasi.
- Pembagian train/validation/test dibangun dengan kehati-hatian agar tidak terjadinya leakage.

### Acceptance Criteria Minggu 2

- Parser dan preprocessing dapat dijalankan ulang dengan output yang terdokumentasi.
- Dataset berlabel mengandung metadata session, waktu, dan konfigurasi.
- Model baseline menghasilkan prediksi dan metrik evaluasi awal.

## Minggu 3: Integrasi, Testing, dan Pelaporan

| Item kerja | Status | Milestone | Acceptance criteria |
|---|---|---|---|
| Integrasi inference dengan backend | Not Started | Hasil prediksi dapat diterima backend | Endpoint atau alur kerja dasar tersedia |
| Aplikasi mobile/dashboard dasar | Not Started | Hasil estimasi ditampilkan | Antarmuka sederhana tersedia |
| Pengujian end-to-end | Not Started | Proses dari CSI ke hasil ditest | Hasil test terdokumentasi |
| Evaluasi error dan latency | Not Started | Keterbatasan sistem dipantau | Grafik/log latency dan error tersedia |
| Dokumentasi akhir dan laporan | Not Started | Laporan proyek dibuat | Keterbatasan dan bukti dokumentasi jelas |

### Milestone Minggu 3

- Hasil estimasi dapat ditampilkan dalam alur end-to-end sederhana.
- Pengujian dilakukan dan keterbatasan disampaikan jujur.
- Dokumentasi akhir sudah siap untuk dipresentasikan.

### Acceptance Criteria Minggu 3

- End-to-end sistem dijalankan atau status kegagalan terdokumentasi dengan bukti.
- Hasil evaluasi, error, dan keterbatasan ditulis dengan transparan.
- Struktur repository, dokumentasi, dan laporan selesai dengan konsisten.

## Ringkasan Status Proyek

- Minggu 1: `In Progress`
- Minggu 2: `In Progress`
- Minggu 3: `Not Started`

Dataset WiCount adalah data referensi eksternal. Pengumpulan dataset khusus dari perangkat ESP32 TX/RX WiFiSENSE dan pengembangan model belum selesai. Semua status dapat berubah sesuai keputusan teknis, eksperimen, dan validasi nyata.
