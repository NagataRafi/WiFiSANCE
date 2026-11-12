# Product Requirements Document (PRD)
# WiFiSENSE — Estimasi Jumlah Manusia Berbasis Wi-Fi CSI

- **Status:** Draft / fondasi pengembangan
- **Versi:** 0.1.0
- **Tanggal:** 12 November 2026
- **Jenis proyek:** Project-Based Learning (PBL) — AI, IoT, dan Aplikasi Mobile
- **Nama repository:** `WiFiSENSE`

---

## 1. Ringkasan Proyek

WiFiSENSE adalah sistem estimasi jumlah manusia di suatu area menggunakan perubahan karakteristik sinyal Wi-Fi yang direkam sebagai **Channel State Information (CSI)**. Sistem inti menggunakan dua board ESP32: satu sebagai transmitter (TX) dan satu sebagai receiver (RX).

Data CSI dari receiver diproses melalui pipeline Digital Signal Processing (DSP), kemudian digunakan oleh model Machine Learning untuk memperkirakan jumlah manusia. Hasil estimasi dikirim ke backend, disimpan, dan ditampilkan melalui aplikasi mobile/dashboard.

**Sensor HC-SR04 bukan bagian dari arsitektur utama.** Sensor tersebut hanya dapat dipertimbangkan sebagai metode alternatif atau pembanding dalam eksperimen terpisah. Pengembangan utama harus tetap dapat berjalan dengan dua ESP32 saja.

## 2. Latar Belakang dan Permasalahan

Metode penghitungan manusia konvensional dapat membutuhkan kamera atau sensor khusus. WiFiSENSE mengeksplorasi pemanfaatan CSI dari sinyal Wi-Fi untuk memperkirakan okupansi ruangan tanpa menjadikan kamera sebagai sensor utama.

Tantangan teknis yang harus dijawab:
1. Apakah dua ESP32 yang tersedia dapat menghasilkan data CSI secara konsisten?
2. Bagaimana data CSI mentah dibersihkan dan diubah menjadi fitur?
3. Apakah fitur CSI dapat digunakan untuk membedakan jumlah manusia?
4. Seberapa baik model bekerja pada variasi posisi, aktivitas, dan kondisi ruangan?
5. Bagaimana hasil estimasi disalurkan ke backend dan aplikasi?

## 3. Tujuan

1. Membangun komunikasi Wi-Fi antara dua ESP32, dengan peran TX dan RX.
2. Mengakuisisi dan menyimpan data CSI dari ESP32 receiver.
3. Membuat pipeline parsing, preprocessing, filtering, dan ekstraksi fitur CSI.
4. Mengumpulkan dataset CSI dengan label jumlah manusia yang diketahui (*ground truth*).
5. Melatih dan mengevaluasi model Machine Learning untuk estimasi jumlah manusia.
6. Menyediakan backend/API dan aplikasi mobile untuk menampilkan hasil estimasi.
7. Menguji kinerja sistem secara terukur dan mendokumentasikan keterbatasannya.

## 4. Ruang Lingkup

### 4.1 Termasuk dalam MVP

- Dua ESP32 fisik:
  - ESP32 TX: memancarkan paket/sinyal Wi-Fi sesuai konfigurasi.
  - ESP32 RX: menerima paket dan mengambil CSI.
- Komunikasi langsung tanpa router eksternal, jika didukung oleh konfigurasi firmware yang dipilih (misalnya SoftAP–Station).
- Akuisisi CSI, logging, dan dokumentasi format data.
- Pipeline Python untuk parsing, preprocessing, DSP, dan ekstraksi fitur.
- Dataset berlabel jumlah orang.
- Model Machine Learning baseline untuk estimasi jumlah orang.
- Evaluasi menggunakan metrik klasifikasi dan/atau regresi yang sesuai.
- Backend/API dan penyimpanan hasil estimasi.
- Aplikasi mobile/dashboard untuk menampilkan jumlah estimasi dan riwayat.
- Pengujian end-to-end dan dokumentasi.

### 4.2 Di luar MVP / opsional

- HC-SR04 sebagai metode alternatif atau pembanding.
- Integrasi kamera sebagai sensor operasional.
- Cloud deployment berbayar.
- Sistem multi-ruangan atau multi-pasang ESP32.
- Klaim akurasi universal untuk semua ruangan.
- Fitur yang belum didukung bukti eksperimen.

## 5. Pengguna dan Skenario Penggunaan

**Pengguna sasaran:** penguji/dosen, anggota tim pengembang, dan operator yang memantau okupansi area uji.

**Skenario utama:**
1. Operator menyalakan ESP32 TX dan RX.
2. Receiver mengumpulkan CSI dan mengirim/menyimpan data ke komputer pemroses.
3. Pipeline memvalidasi dan memproses data CSI.
4. Model menghasilkan estimasi jumlah manusia.
5. Backend menerima hasil, menyimpan timestamp dan nilai estimasi.
6. Aplikasi menampilkan nilai estimasi dan riwayat.

## 6. Arsitektur Sistem

```text
ESP32 TX
   |
   | Sinyal/paket Wi-Fi
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
   +------ Database
   |
   v
Mobile App / Dashboard
```

**Catatan arsitektur:**
- ESP32 TX dan RX adalah satu-satunya hardware wajib pada sistem inti.
- Komputer/laptop dapat digunakan untuk logging, preprocessing, training, backend, dan pengembangan aplikasi.
- Tentukan mekanisme komunikasi ESP32 berdasarkan kemampuan board dan firmware yang benar-benar diuji. Jangan menganggap semua tipe ESP32 memiliki dukungan CSI atau konfigurasi yang identik.
- Jangan mengunci format data sebelum firmware dan contoh keluaran CSI diverifikasi.
- Pisahkan data mentah, data hasil preprocessing, fitur, prediksi, dan label ground truth.

## 7. Persyaratan Fungsional

| ID | Persyaratan | Prioritas |
|---|---|---|
| FR-01 | Sistem dapat menginisialisasi ESP32 TX dan RX. | Must |
| FR-02 | TX dapat mengirim paket Wi-Fi dan RX dapat menerima paket. | Must |
| FR-03 | RX dapat mengaktifkan CSI dan menghasilkan data CSI. | Must |
| FR-04 | Data CSI dapat direkam dengan metadata/timestamp yang konsisten. | Must |
| FR-05 | Pipeline dapat membaca dan memvalidasi format data CSI. | Must |
| FR-06 | Pipeline dapat melakukan preprocessing dan filtering yang terdokumentasi. | Must |
| FR-07 | Sistem dapat mengekstraksi fitur dari data CSI. | Must |
| FR-08 | Dataset dapat diberi label jumlah manusia berdasarkan ground truth. | Must |
| FR-09 | Model dapat dilatih dan menghasilkan prediksi jumlah manusia. | Must |
| FR-10 | Model dapat dievaluasi dan hasilnya disimpan. | Must |
| FR-11 | Backend menerima dan menyimpan hasil estimasi. | Should |
| FR-12 | Aplikasi menampilkan estimasi terbaru dan riwayat. | Should |
| FR-13 | Sistem mencatat error atau status proses penting. | Should |

## 8. Persyaratan Non-Fungsional

- **Reproducibility:** versi firmware, konfigurasi, dataset, parameter preprocessing, dan model harus dicatat.
- **Traceability:** setiap prediksi dapat ditelusuri ke waktu pengambilan dan versi model.
- **Data integrity:** data mentah tidak boleh ditimpa oleh hasil preprocessing.
- **Modularity:** firmware, parser, DSP, ML, backend, dan mobile dipisahkan.
- **Resource awareness:** pengembangan mempertimbangkan laptop Windows dengan RAM 8 GB.
- **Maintainability:** gunakan struktur sederhana dan hindari dependensi runtime yang tidak diperlukan.
- **Honest evaluation:** hasil pengujian harus dilaporkan apa adanya, termasuk keterbatasan dan kegagalan.

## 9. Dataset dan Eksperimen

Dataset harus dikumpulkan dengan label jumlah orang yang diketahui. Minimal setiap sesi mencatat:
- jumlah orang aktual;
- ID sesi dan waktu pengambilan;
- lokasi/ruangan dan posisi perangkat;
- kondisi aktivitas (diam/bergerak, jika diuji);
- konfigurasi TX/RX dan firmware;
- nama file data CSI mentah.

Rancangan kelas jumlah orang ditetapkan setelah uji awal. Contoh kelas awal: 0, 1, 2, 3, dan 4 orang. Jangan mengklaim kemampuan 0–7 orang sebelum kelas tersebut benar-benar dikumpulkan dan diuji.

Pisahkan data training, validation, dan testing berdasarkan sesi atau subjek bila memungkinkan. Hindari kebocoran data akibat membagi potongan dari sesi yang sama ke train dan test.

Metrik yang dipertimbangkan:
- klasifikasi: accuracy, precision, recall, F1-score, confusion matrix;
- estimasi jumlah: MAE dan RMSE;
- sistem: latency, packet/data loss yang teramati, stabilitas, dan keberhasilan akuisisi.

## 10. Strategi Integrasi Repository Referensi

Repository eksternal digunakan sebagai referensi per modul, bukan digabungkan secara langsung tanpa pemeriksaan.

Urutan yang disarankan:
1. **ESP-IDF / Espressif ESP-CSI:** referensi resmi untuk firmware dan akuisisi CSI.
2. **ESP32-WiFi-CSI-Human-Sensing:** referensi parser, preprocessing, dan ekstraksi fitur.
3. **wifi-csi-occupancy:** referensi metode fitur, training, dan evaluasi okupansi.
4. **wifi-sense:** referensi pola integrasi backend dan dashboard.

Sebelum mengadopsi kode:
- periksa lisensi dan atribusi;
- catat URL repository, commit/tag, file yang digunakan, dan perubahan;
- periksa tipe chip ESP32 dan versi ESP-IDF/Arduino;
- bandingkan format CSI, timestamp, jumlah subcarrier, sampling, dan satuan;
- buat adapter bila format berbeda;
- jalankan pengujian kecil sebelum integrasi;
- jangan menyalin kode atau dataset yang akses/lisensinya tidak jelas.

## 11. Struktur Folder Repository

Agent VS Code wajib membuat struktur berikut. Buat `README.md`, `PRD.md`, `roadmap.md`, `changelog.md`, dan `AGENTS.md` di root. Buat file placeholder hanya jika berguna; jangan membuat file kode kosong yang seolah-olah sudah berfungsi.

```text
WiFiSENSE/
├── AGENTS.md
├── README.md
├── PRD.md
├── roadmap.md
├── changelog.md
├── .gitignore
│
├── docs/
│   ├── ARCHITECTURE.md
│   ├── REPOSITORY_AUDIT.md
│   ├── DATA_FORMAT.md
│   ├── DSP_PIPELINE.md
│   ├── AI_MODEL.md
│   └── TESTING_PLAN.md
│
├── firmware/
│   ├── transmitter/
│   │   └── README.md
│   └── receiver/
│       └── README.md
│
├── data/
│   ├── raw/
│   │   └── .gitkeep
│   ├── processed/
│   │   └── .gitkeep
│   ├── labeled/
│   │   └── .gitkeep
│   └── README.md
│
├── processing/
│   ├── parser/
│   │   └── README.md
│   ├── preprocessing/
│   │   └── README.md
│   └── feature_extraction/
│       └── README.md
│
├── models/
│   └── README.md
│
├── backend/
│   └── README.md
│
├── mobile/
│   └── README.md
│
├── tests/
│   └── README.md
│
└── references/
    └── REPOSITORIES.md
```

### Aturan struktur

- `firmware/transmitter/`: kode khusus ESP32 TX.
- `firmware/receiver/`: kode khusus ESP32 RX dan CSI acquisition.
- `data/raw/`: data asli, tidak diedit.
- `data/processed/`: data setelah preprocessing.
- `data/labeled/`: dataset dengan label ground truth.
- `processing/parser/`: pembacaan dan parsing format CSI.
- `processing/preprocessing/`: validasi, cleaning, filtering, normalisasi bila diperlukan.
- `processing/feature_extraction/`: ekstraksi fitur.
- `models/`: training, evaluasi, dan artefak model; jangan commit file model besar tanpa keputusan tim.
- `backend/`: API dan penyimpanan hasil.
- `mobile/`: aplikasi mobile/dashboard.
- `tests/`: pengujian unit dan integrasi.
- `references/`: daftar repository eksternal, lisensi, dan catatan adaptasi.

Folder `data/` harus diatur dalam `.gitignore` agar file dataset mentah/besar tidak otomatis masuk Git. Pertahankan `.gitkeep` agar struktur folder tetap ada. Jangan mengabaikan file README atau metadata dataset yang memang perlu dilacak.

## 12. Roadmap 3 Minggu

Tanggal pelaksanaan aktual harus dikonfirmasi tim. Roadmap berikut menggunakan pembagian relatif tiga minggu sejak proyek dimulai.

### Minggu 1 — Hardware dan CSI Proof of Concept
- Identifikasi tipe/varian kedua board ESP32.
- Pilih toolchain dan versi firmware.
- Jalankan TX dan RX.
- Buktikan CSI berhasil diperoleh.
- Simpan contoh data mentah dan dokumentasikan formatnya.
- Buat audit repository referensi.

**Exit criteria:** receiver menghasilkan file/log CSI yang dapat dibaca dan konfigurasi dapat diulang.

### Minggu 2 — DSP, Dataset, dan Model Baseline
- Implementasikan parser dan validasi data.
- Implementasikan preprocessing dan ekstraksi fitur.
- Kumpulkan dataset dengan ground truth.
- Latih model baseline.
- Evaluasi dengan pembagian data yang mencegah leakage.

**Exit criteria:** pipeline dari data CSI ke prediksi baseline berjalan dan metrik evaluasi tersedia.

### Minggu 3 — Integrasi, Pengujian, dan Pelaporan
- Integrasikan inference dengan backend.
- Tampilkan hasil melalui aplikasi mobile/dashboard.
- Jalankan pengujian end-to-end.
- Analisis error, latency, dan keterbatasan.
- Rapikan dokumentasi, demonstrasi, dan laporan akhir.

**Exit criteria:** demonstrasi end-to-end berjalan atau, jika ada bagian yang belum selesai, status dan bukti kendalanya terdokumentasi secara jujur.

## 13. Pembagian Tanggung Jawab Tim

Untuk tim tujuh orang:
1. Koordinator dan integrasi sistem.
2. Firmware ESP32 TX.
3. Firmware ESP32 RX dan CSI acquisition.
4. DSP, parser, preprocessing, dan ekstraksi fitur.
5. Dataset, Machine Learning, dan evaluasi.
6. Backend, API, dan database.
7. Aplikasi mobile/dashboard dan integrasi tampilan.

Anggota dapat saling membantu, tetapi setiap modul harus memiliki satu penanggung jawab utama.

## 14. Risiko dan Mitigasi

| Risiko | Mitigasi |
|---|---|
| Varian ESP32 tidak mendukung konfigurasi CSI yang dipilih | Verifikasi model chip dan dukungan firmware sebelum implementasi |
| CSI tidak stabil atau tidak terbaca | Buat PoC minimal dan simpan log diagnostik |
| Format data antar-repository berbeda | Tetapkan format internal dan buat adapter |
| Dataset kurang atau tidak seimbang | Rencanakan sesi pengambilan data dan catat distribusi kelas |
| Model hanya mengenali presence, bukan jumlah | Pastikan label dan target model adalah jumlah orang |
| Data train/test bocor | Pisahkan berdasarkan sesi/subjek |
| Integrasi backend/mobile memakan waktu | Tunda fitur tambahan; prioritaskan API sederhana dan tampilan inti |
| Waktu tiga minggu terbatas | Prioritaskan CSI PoC, dataset, dan model baseline sebelum fitur kosmetik |

## 15. Definition of Done

Proyek dianggap mencapai MVP jika:
- dua ESP32 dapat menjalankan peran TX dan RX;
- receiver menghasilkan data CSI yang tersimpan;
- parser dan preprocessing dapat dijalankan ulang;
- dataset memiliki ground truth yang terdokumentasi;
- model baseline menghasilkan prediksi dan metrik evaluasi;
- hasil estimasi dapat ditampilkan melalui antarmuka sederhana;
- instruksi instalasi dan pengujian tersedia;
- keterbatasan sistem ditulis secara transparan.

Jika target tertentu belum tercapai, dokumentasikan bukti, kendala, dan statusnya. Jangan menandai fitur selesai tanpa pengujian.

## 16. Aturan Kerja untuk AI Coding Agent

1. Baca `AGENTS.md`, `PRD.md`, `roadmap.md`, dan `changelog.md` sebelum mengubah proyek.
2. Pada tahap awal, tugas agent hanya membuat struktur folder dan dokumen fondasi sesuai PRD. Jangan langsung membuat seluruh firmware, model AI, backend, atau aplikasi.
3. Jangan mengarang hasil pengujian, dataset, dukungan hardware, atau akurasi.
4. Jangan menghapus atau menimpa pekerjaan yang sudah ada.
5. Gunakan perubahan sekecil mungkin dan hindari dependensi baru tanpa alasan serta persetujuan.
6. Jika informasi hardware/toolchain belum tersedia, tulis `TODO` atau `TBD` dan ajukan pertanyaan; jangan menebak.
7. Perbarui `changelog.md` untuk perubahan yang dilakukan.
8. Perbarui `roadmap.md` hanya jika status atau urutan pekerjaan benar-benar berubah.
9. Setelah tugas selesai, laporkan file/folder yang dibuat, keputusan yang diambil, hal yang belum diketahui, dan cara memverifikasi hasil.
10. Jangan melakukan commit atau push kecuali pengguna secara eksplisit memintanya.

---

## 17. Keputusan yang Masih Harus Dikonfirmasi

- Model/varian kedua board ESP32.
- Toolchain: ESP-IDF atau Arduino.
- Koneksi langsung TX–RX dan konfigurasi Wi-Fi yang akan dipakai.
- Framework backend.
- Framework aplikasi mobile.
- Rentang kelas jumlah orang yang sanggup dikumpulkan dan diuji.
- Lokasi dan jadwal eksperimen.

Keputusan yang belum dikonfirmasi harus tetap ditandai `TBD`, bukan dianggap sudah final.
