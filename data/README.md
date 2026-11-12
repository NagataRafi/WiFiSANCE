# Data Module

Folder ini mengelola semua data proyek WiFiSENSE, dibagi menjadi data mentah, hasil preprocessing, dan dataset berlabel.

## Struktur

- `raw/` — data CSI mentah dan log akuisisi.
- `processed/` — hasil preprocessing dan data transformasi.
- `labeled/` — dataset berlabel jumlah manusia dan metadata terkait.

## Aturan

- Data mentah tidak boleh ditimpa oleh hasil preprocessing.
- Semua dataset harus dibarengi metadata yang terdokumentasi.
- File besar dan artefak data dijaga agar tidak masuk ke Git secara otomatis via `.gitignore`.

## Status

Belum ada dataset eksperimen yang dihasilkan. Struktur folder sudah dibuat dan akan diisi saat akuisisi CSI dimulai.
