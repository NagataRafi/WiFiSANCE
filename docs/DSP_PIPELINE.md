# DSP Pipeline Plan

Pipeline DSP WiFiSENSE dirancang untuk memproses data CSI dari ESP32 receiver sebelum masuk ke model machine learning. Pada tahap awal, semua langkah ini masih bersifat perencanaan dan harus divalidasi dengan data nyata.

## 1. CSI parsing

Pada tahap ini data CSI dibaca dari file atau log raw yang dihasilkan receiver. Tugas utama:

- membaca struktur data raw;
- mengekstrak waktu, sesi, dan metadata;
- memetakan payload CSI ke representasi internal yang konsisten;
- menangani data yang tidak lengkap atau rusak.

## 2. Data validation

Sebelum diproses lebih lanjut, data harus diperiksa untuk menjamin kualitas awal. Validasi dapat mencakup:

- apakah payload hadir dan terisi;
- apakah metadata penting tersedia;
- apakah data sesuai dengan sesi atau perangkat yang benar;
- apakah ada paket yang hilang, duplikat, atau tidak konsisten.

Jika data tidak valid, hasilnya dicatat dan dapat dipisahkan dari dataset utama.

## 3. Preprocessing

Tahap preprocessing bertujuan membersihkan dan menyusun data agar siap untuk feature extraction. Langkah yang kemungkinan dipakai meliputi:

- filtering noise dasar;
- normalisasi amplitudo atau fase bila relevan;
- segmentasi frame atau window data;
- penghapusan artefak dari sesi yang tidak stabil;
- penyusunan struktur data yang konsisten untuk tiap sampel.

Detail metode tetap `TBD` sampai eksperimen dilakukan.

## 4. Filtering

Filter digunakan untuk mengurangi gangguan dan menjaga kualitas sinyal CSI. Kriteria filter harus dipilih berdasarkan hasil eksperimen dan tidak boleh dipaksakan. Beberapa opsi yang mungkin dipertimbangkan:

- low-pass / high-pass filtering;
- detrending;
- smoothing;
- noise suppression sesuai karakteristik CSI;
- penghilangan outlier.

Semua filter yang dipilih harus terdokumentasi beserta alasan dan parameter yang dipakai.

## 5. Feature extraction

Setelah data bersih, fitur dari CSI diekstraksi untuk digunakan dalam model. Fitur yang umum dipertimbangkan dalam konteks CSI human sensing meliputi:

- magnitude dan phase-based descriptors;
- statistik temporal dan frekuensi;
- perubahan karakteristik sinyal antar-waktu;
- fitur kombinasi yang dapat membedakan jumlah manusia.

Fitur final akan ditentukan setelah data empiris tersedia. Dalam tahap awal, fitur tetap bersifat kandidat dan tidak dianggap final.

## 6. Output pipeline

Hasil akhir pipeline diharapkan berupa data siap model, yaitu versi yang telah: valid, dibersihkan, difilter, dan memiliki feature yang siap dipakai untuk training atau inference.

Pada tahap awal, pipeline ini harus tetap dibuat modular dan dipisahkan dari dataset asli. Hal ini sesuai dengan prinsip PRD: data mentah, data preprocessing, dan label harus tetap terpisah.
