# AI Model Plan

Dokumen ini menjelaskan rencana pengembangan model untuk estimasi jumlah manusia berdasarkan CSI Wi-Fi.

## 1. Dataset

Dataset yang dibutuhkan adalah data CSI yang dikumpulkan dari ESP32 receiver dengan label jumlah manusia yang diketahui. Setiap sesi eksperimen harus mencatat:

- jumlah orang aktual;
- ID sesi dan timestamp;
- lokasi / ruangan eksperimen;
- konfigurasi TX-RX dan firmware;
- aktivitas pengamat (diam/bergerak, bila dicatat);
- nama file data CSI raw dan label terkait.

Data harus dipisahkan menjadi:

- `data/raw/` — data mentah tak diubah;
- `data/processed/` — hasil preprocessing;
- `data/labeled/` — dataset yang sudah diberi label ground truth.

## 2. Label target

Target model adalah estimasi jumlah manusia. Kelas jumlah orang belum ditetapkan final karena harus dibuktikan dari eksperimen.

Contoh kelas awal yang disebutkan di PRD adalah 0, 1, 2, 3, dan 4 orang; namun kelas tersebut harus tetap dianggap `TBD` sampai dataset yang relevan benar-benar dikumpulkan dan dievaluasi.

## 3. Training strategy

Pada tahap awal, strategi training akan mengikuti pendekatan yang sederhana dan terdokumentasi:

- memeriksa data kualitas dan distribusi label;
- membagi data berdasarkan sesi atau subjek agar tidak terjadi leakage;
- memilih baseline model yang sesuai dengan data CSI dan ukuran dataset;
- mengukur performa dengan metrik yang sesuai.

## 4. Candidate model approach

Model belum dipilih secara final. Kandidat yang kemungkinan dipertimbangkan meliputi:

- model klasifikasi untuk kategori jumlah orang;
- model regresi bila target berupa estimasi jumlah kontinu;
- pendekatan berbasis feature engineering dengan model klasik;
- pendekatan deep learning bila data dan komputasi mendukung.

Semua pendekatan harus disesuaikan dengan hasil eksperimen dan ukuran dataset.

## 5. Inference flow

Alur inferensi yang diharapkan:

1. CSI diterima dari receiver.
2. Data diproses melalui pipeline DSP.
3. Fitur diekstraksi.
4. Model melakukan prediksi jumlah manusia.
5. Hasil prediksi dikirim ke backend untuk disimpan dan ditampilkan.

## 6. Evaluation

Metrik evaluasi yang akan dipertimbangkan:

- accuracy;
- precision;
- recall;
- F1-score;
- confusion matrix;
- MAE;
- RMSE;
- latency dan stabilitas sistem.

Kinerja model harus dilaporkan apa adanya, termasuk kondisi di mana data terbatas atau hasil belum konsisten.

## 7. Keterbatasan yang harus dijaga

- Jangan mengklaim akurasi universal untuk semua ruangan.
- Jangan menganggap target kelas otomatis final.
- Jangan membagi data dari sesi yang sama ke train dan test secara sembarangan.
- Jangan mengarang hasil eksperimen atau performa model yang belum ada.

Semua keputusan model akan diperbarui berdasarkan eksperimen nyata dan dokumentasi yang telah diverifikasi.
