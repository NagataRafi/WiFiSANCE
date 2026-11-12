# WiFiSENSE

WiFiSENSE adalah proyek PBL yang mengeksplorasi penggunaan Channel State Information (CSI) Wi-Fi untuk memperkirakan jumlah manusia di suatu area tanpa bergantung pada kamera sebagai sensor utama. Sistem inti dibangun dari dua ESP32: satu sebagai transmitter (TX) dan satu sebagai receiver (RX), dengan pipeline pemrosesan yang mencakup CSI, DSP, Machine Learning, backend, dan aplikasi mobile.

## Tujuan Proyek

- Menggunakan dua ESP32 sebagai perangkat utama untuk komunikasi Wi-Fi dan akuisisi CSI.
- Mengumpulkan data CSI dari receiver dan memastikan format data terdokumentasi dengan baik.
- Membangun pipeline parsing, preprocessing, filtering, dan ekstraksi fitur CSI.
- Menyusun dataset berlabel jumlah manusia berdasarkan ground truth.
- Menciptakan model pembelajaran mesin baseline untuk estimasi jumlah manusia.
- Menyediakan backend untuk menerima hasil estimasi dan mobile/dashboard untuk visualisasi.

## Teknologi dan Arsitektur Tingkat Tinggi

Arsitektur utama proyek mengikuti alur berikut:

ESP32 TX -> Wi-Fi signal propagation -> ESP32 RX -> CSI acquisition -> DSP pipeline -> Feature extraction -> ML inference -> Backend API -> Mobile/dashboard

Komponen utama dalam repositori ini dipisahkan berdasarkan fungsi:

- Firmware ESP32: sistem komunikasi dan akuisisi CSI.
- Processing: parsing, validasi, preprocessing, dan feature engineering.
- Models: dataset, training, evaluasi, dan artefak model.
- Backend: REST API, log status, dan penyimpanan hasil.
- Mobile: antarmuka pengguna untuk menampilkan estimasi.

## Status Pengembangan

Status proyek saat ini adalah fondasi awal dan struktur repository. Pengembangan aktif belum mencakup firmware produksi, model AI, backend runtime, atau aplikasi mobile final.

Segala keputusan yang belum dikonfirmasi akan tetap ditandai sebagai `TBD`.

## Struktur Direktori

```text
WiFiSANSE/
├── AGENTS.md
├── README.md
├── PRD.md
├── roadmap.md
├── changelog.md
├── .gitignore
├── docs/
│   ├── ARCHITECTURE.md
│   ├── REPOSITORY_AUDIT.md
│   ├── DATA_FORMAT.md
│   ├── DSP_PIPELINE.md
│   ├── AI_MODEL.md
│   └── TESTING_PLAN.md
├── firmware/
│   ├── transmitter/
│   │   └── README.md
│   └── receiver/
│       └── README.md
├── data/
│   ├── raw/
│   │   └── .gitkeep
│   ├── processed/
│   │   └── .gitkeep
│   ├── labeled/
│   │   └── .gitkeep
│   └── README.md
├── processing/
│   ├── parser/
│   │   └── README.md
│   ├── preprocessing/
│   │   └── README.md
│   └── feature_extraction/
│       └── README.md
├── models/
│   └── README.md
├── backend/
│   └── README.md
├── mobile/
│   └── README.md
├── tests/
│   └── README.md
├── references/
│   └── REPOSITORIES.md
└── .gitignore
```

## Cara Memulai Pengembangan

1. Baca `PRD.md` sebagai dokumen kebutuhan utama.
2. Review `AGENTS.md` untuk aturan AI agent dan alur kerja proyek.
3. Gunakan `roadmap.md` untuk memahami urutan pekerjaan 3 minggu.
4. Mulai dari modul `firmware`, `processing`, dan `docs` untuk memvalidasi CSI dan format data.
5. Hindari pembuatan firmware, model, backend, atau aplikasi final sebelum kebutuhan dan eksperimen terdokumentasi.

## Referensi Dokumentasi

- `PRD.md` — persyaratan produk dan batasan proyek.
- `roadmap.md` — jadwal 3 minggu dan milestone.
- `docs/ARCHITECTURE.md` — arsitektur sistem.
- `docs/DATA_FORMAT.md` — spesifikasi data internasional CSI.
- `docs/DSP_PIPELINE.md` — rencana pipeline pemrosesan sinyal.
- `docs/AI_MODEL.md` — rencana dataset dan model.
- `docs/TESTING_PLAN.md` — strategi pengujian.
- `docs/REPOSITORY_AUDIT.md` — audit repository referensi.
- `references/REPOSITORIES.md` — daftar repositori referensi yang perlu diaudit.

## Catatan

Proyek ini bertujuan untuk membangun fondasi yang jujur, terdokumentasi, dan modular. Semua asumsi hardware, konfigurasi firmware, kelas target, dan framework teknis yang belum divalidasi tetap ditandai sebagai `TBD`.
