# Validasi Akhir Dataset CSI dan Kesiapan ML

## 1. Ruang lingkup

Audit akhir ini memeriksa 15 CSV WiCount berdasarkan urutan baris asli dan meninjau matriks fitur 50 sampel yang sudah tersedia. DSP dan ekstraksi fitur tidak diulang. Tidak ada baris atau sampel yang dihapus, tidak ada model yang dilatih, dan matriks fitur baseline maupun arsip dataset tidak ditimpa.

Notebook yang dapat dijalankan ulang:

- `analysis/notebooks/csi_final_validation.ipynb`

## 2. Timestamp dan sequence number

Untuk setiap recording, timestamp dan sequence number dibaca dalam urutan baris CSV. Perhitungan delta memakai `timestamp.diff()` dan `seq.diff()` tanpa mengurutkan timestamp.

| Pemeriksaan | Hasil seluruh recording |
|---|---:|
| Recording diperiksa | 15 |
| Baris CSI dibaca | 78.618 |
| Timestamp kosong | 0 |
| Timestamp gagal parsing | 0 |
| Baris dengan timestamp berulang | 8.900 |
| `timestamp.diff() == 0` | 8.900 |
| `timestamp.diff() < 0` | 0 |
| `timestamp.diff() > 0` | 69.703 |
| Delta timestamp yang tidak tersedia pada awal recording | 15 |
| Sequence number kosong/gagal parsing | 0 |
| Sequence number duplikat | 0 |
| `seq.diff() == 0` atau negatif | 0 |
| Delta sequence number selain `+1` | 0 |
| `seq.diff() == +1` | 78.603 |

Timestamp berulang terdapat pada 11 dari 15 recording. Setiap delta timestamp nol yang berulang tetap berpasangan dengan kenaikan sequence number; tidak ada sequence number duplikat atau lompatan non-unit dalam data yang diperiksa. Ini mendukung kesimpulan bahwa baris-baris dengan timestamp sama bukan duplikasi paket yang dapat dibuktikan dari timestamp saja. Hal ini tidak menjelaskan penyebab timestamp berulang atau menjamin semantik sequence number di luar nilai yang tercatat.

### Gap waktu

Untuk menandai gap secara deskriptif, sebuah delta positif ditandai bila lebih dari tiga kali median delta positif di recording yang sama. Dengan ambang heuristik tersebut, terdapat 32 gap. Median delta positif per recording berada pada rentang 11–16 ms; delta positif terbesar dalam satu recording adalah 2.013 ms. Ambang ini bukan batas kualitas atau laju sampling yang ditentukan dataset.

Tidak ada delta sequence yang hilang atau tidak berurutan pada recording-recording yang diperiksa. Karena itu, gap timestamp tersebut sendiri tidak membuktikan paket hilang; sebaliknya, sequence yang berurutan juga tidak membuktikan bahwa tidak pernah terjadi kehilangan di luar semantik field `seq` yang tersedia.

Tidak ada timestamp, sequence, atau sampel yang dihapus atau diperbaiki. Ringkasan per recording tersedia di `analysis/processed/features/recording_timestamp_audit.csv`.

## 3. Feature redundancy dan rekomendasi reduksi

Matriks baseline `analysis/processed/features/feature_matrix_50.csv` tetap memiliki 3.122 window dan 999 kolom: 992 fitur numerik serta tujuh metadata (`window_id`, `recording_id`, `occupancy`, `window_size`, `start_index`, `end_index`, `row_count`).

| Pemeriksaan | Hasil |
|---|---:|
| Pasangan dengan `|Pearson r| >= 0.90` | 3.892 |
| Pasangan dalam subcarrier yang sama | 1.362 |
| Pasangan antar-subcarrier | 2.529 |
| Pasangan yang melibatkan fitur agregat lintas-subcarrier | 1 |
| Pasangan kolom yang nilainya identik pada seluruh window | 52 |
| Fitur konstan (`variance <= 1e-12`) | 0 |
| Fitur variance `< 0.01` dalam skala mentah | 52 |

Ke-52 pasangan kolom identik adalah `sub_i_std` dan `sub_i_temporal_std`, satu pasangan untuk setiap subcarrier. Nilai mentah tetap dipertahankan pada baseline. Ke-52 fitur `sub_i_first_diff_mean` memiliki variance di bawah 0.01; ambang variance ini bergantung skala dan hanya dipakai untuk menandai fitur, bukan sebagai alasan otomatis untuk menghapusnya.

Selain itu, `cross_subcarrier_var_amp` secara aljabar adalah kuadrat `cross_subcarrier_std_amp` untuk definisi `ddof` yang sama. Pemeriksaan numerik pada matriks menunjukkan galat absolut maksimum terhadap relasi tersebut sebesar `1.42e-14`.

### Kandidat compact set (bukan hasil seleksi model)

File rekomendasi terpisah memiliki satu baris untuk masing-masing 992 fitur, beserta tipe, subcarrier, variance, korelasi maksimum, jumlah pasangan berkorelasi tinggi, rekomendasi, dan alasannya:

- `analysis/processed/features/feature_reduction_recommendation.csv`

Kandidat awal berisi 211 fitur: untuk setiap subcarrier, `mean`, `std`, `zero_crossing_rate`, dan `first_diff_std` (208 fitur); serta `cross_subcarrier_mean_amp`, `cross_subcarrier_std_amp`, dan `cross_subcarrier_corr_median` (3 fitur).

File tersebut menandai 52 `temporal_std` sebagai duplikat tepat dari `std`, satu fitur `cross_subcarrier_var_amp` sebagai fitur yang bergantung secara aljabar pada cross-subcarrier standard deviation, dan 728 fitur lain sebagai kandidat ablation opsional di luar compact set awal. Semua baris baseline tetap ada. Compact set ini hanya penyederhanaan kandidat untuk eksperimen ablation kelak, bukan klaim bahwa fitur tersebut paling prediktif atau terbaik.

Detail pasangan korelasi, termasuk nama tipe dan indeks subcarrier, ada di `analysis/outputs/final_validation/high_correlation_pair_details.csv`. Korelasi tinggi antar-subcarrier juga banyak; compact set 211 fitur masih perlu ditinjau lebih lanjut dengan data training saja.

## 4. Kelayakan group-based validation

Recording adalah unit pembagian. Semua window dari satu `recording_id` wajib berada pada kelompok yang sama.

| Occupancy | Recording | Window | Recording train bila satu group ditahan | Recording validation/test bila satu group ditahan |
|---:|---:|---:|---:|---:|
| 0 | 1 | 207 | 1 | 0 |
| 1 | 7 | 1.452 | 6 | 1 |
| 2 | 6 | 1.268 | 5 | 1 |
| 3 | 1 | 195 | 1 | 0 |

Ini adalah ilustrasi kapasitas, bukan split yang sudah dijalankan. Untuk kelas 0 dan 3, satu-satunya recording dapat dipertahankan di training, tetapi tidak ada recording independen untuk validation/test pada kelas tersebut. Menahan recording singleton itu justru menghilangkan kelasnya dari training. Maka stratified group split lengkap untuk empat kelas tidak dapat dibuat tanpa memaksakan evaluasi yang tidak valid.

Kelas 1 dan 2 dapat menyediakan recording holdout, tetapi jumlah sesi tetap terbatas. Satu holdout per kelas bukan bukti generalisasi luas. Pembagian acak baris/window dilarang; window yang overlap dari recording sama dapat berbagi sampel CSI.

Tabel tersedia sebagai `analysis/processed/features/group_split_feasibility.csv`; jumlah window per recording tersimpan di `analysis/outputs/final_validation/window_counts_by_recording.csv`.

## 5. Audit preprocessing dan risiko leakage

- Tujuh kolom metadata—termasuk label, `recording_id`, dan `window_id`—secara eksplisit dikeluarkan dari 992 kolom numerik.
- PCA pada notebook QA menggunakan fitur numerik saja, tetapi distandardisasi dan di-fit terhadap seluruh window. Itu hanya eksplorasi deskriptif, bukan transformasi model yang bebas leakage. PCA terdahulu di notebook DSP juga mengecualikan metadata, tetapi tidak melakukan standardisasi.
- Eksperimen normalisasi yang tersimpan adalah perbandingan deskriptif per recording/per window, bukan transformer yang di-fit pada train dan diterapkan ke validation/test.
- Korelasi dan rekomendasi fitur pada audit ini dihitung terhadap seluruh matriks untuk keperluan QA. Hasilnya tidak boleh langsung dipakai sebagai feature selection dalam evaluasi model.
- Pada eksperimen ML mendatang, imputer, scaler, PCA, ambang/filter berbasis statistik data, serta pemilihan fitur harus dipelajari hanya dari recording training di dalam setiap fold, lalu diterapkan tanpa refit ke validation/test.
- Notebook validasi ini tidak melakukan split atau training. Ia memeriksa metadata, recording ID, dan integritas input.

## 6. Keputusan kesiapan ML

**Belum layak untuk baseline ML empat kelas dengan evaluasi generalisasi yang dapat dipertanggungjawabkan.** Matriks fitur tersedia untuk percobaan teknis terbatas setelah ada persetujuan, tetapi kelas 0 dan 3 tidak memiliki sesi independen untuk evaluasi. Kelas tidak seimbang, jumlah recording kecil, dan urutan label terkait urutan waktu perekaman. Sebuah eksperimen terbatas pada kelas 1 dan 2 atau pemakaian kelas 0/3 hanya saat training tidak boleh dilaporkan sebagai evaluasi empat kelas.

Sebelum klaim generalisasi, kumpulkan beberapa recording independen untuk setiap kelas, interleave atau acak urutan kondisi perekaman, serta catat sesi dan kondisi akuisisi. Setelah itu, tetapkan group split pada recording dan fit seluruh preprocessing hanya pada kelompok training. Tidak ada model yang dilatih dalam validasi ini.

## 7. Artefak dan validasi integritas

Notebook menghasilkan gambar berikut di `analysis/outputs/final_validation/`:

- `timestamp_order_audit.png`
- `sequence_delta_distribution.png`
- `feature_redundancy_by_type.png`
- `feature_variance_rank.png`
- `group_split_feasibility.png`

File tabel audit:

- `analysis/processed/features/recording_timestamp_audit.csv`
- `analysis/processed/features/feature_reduction_recommendation.csv`
- `analysis/processed/features/group_split_feasibility.csv`

Notebook juga menyimpan detail pasangan korelasi dan hitungan window per recording di folder output. Eksekusi end-to-end memeriksa bahwa seluruh 15 recording dan 78.618 baris dibaca, tidak ada sampel yang dihapus, feature matrix berisi seluruh 15 recording, metadata tidak masuk fitur numerik, dan hash SHA-256 arsip, feature matrix baseline, serta `PRD.md` sama sebelum dan sesudah eksekusi.

Perubahan ini tidak melatih Machine Learning, mengubah `PRD.md`, atau melakukan commit/push.
