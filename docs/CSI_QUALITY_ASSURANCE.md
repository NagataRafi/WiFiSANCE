# CSI Quality Assurance

## 1. Ringkasan pekerjaan

Dokumen ini mencatat audit kualitas pada data CSI WiCount yang digunakan sebagai baseline analisis di repository WiFiSENSE. Tahap ini berfokus pada validasi fitur DSP, kualitas timestamp, redundansi fitur, koreksi PCA, dan keamanan pembagian data untuk eksperimen Machine Learning di masa depan.

Pekerjaan ini tidak melatih model, tidak menambah data sintetis, dan tidak mengubah dataset asli. Semua hasil yang dilaporkan di bawah ini didasarkan pada eksekusi nyata terhadap `analysis/dataset/WiCount/WiCount.zip` dan matriks fitur `analysis/processed/features/feature_matrix_50.csv`.

## 2. Dataset dan baseline fitur

- Dataset sumber: `analysis/dataset/WiCount/WiCount.zip`
- Rekaman yang diproses: 15 file CSV
- Total baris CSI: 78,618
- Total kelas (berdasarkan folder data): kelas 0 = 1 rekaman, kelas 1 = 7 rekaman, kelas 2 = 6 rekaman, kelas 3 = 1 rekaman
- Baseline fitur yang diaudit: `feature_matrix_50.csv`
- Dimensi baseline: 3,122 window x 999 kolom
- Metadata yang dipertahankan: `window_id`, `recording_id`, `occupancy`, `window_size`, `start_index`, `end_index`, `row_count`
- Fitur numerik: 992 kolom
- Tidak ada NaN atau infinity yang tertinggal pada matriks fitur baseline setelah penanganan `fillna(0.0)` dan `replace([inf, -inf], np.nan)` di tahap pemrosesan sebelumnya.

## 3. Audit definisi fitur

### 3.1 Statistical features

Definisi yang digunakan sesuai dengan eksperimen sebelumnya:

- Mean: rata-rata amplitudo pada tiap subcarrier dalam jendela.
- Median: nilai tengah amplitude dalam jendela.
- Standard deviation: `std` populasi/ sample-like `ddof=1` untuk setiap subcarrier.
- Variance: `var` dengan `ddof=1`.
- Minimum dan maximum: nilai extrem pada tiap subcarrier.
- Range: `max - min`.
- Q1, Q3, IQR: kuartil pertama, kuartil ketiga, dan selisihnya.
- Skewness: momen ketiga yang dinormalisasi dengan standar deviasi kubik.
- Kurtosis: momen keempat yang dinormalisasi lalu dikurangi 3 (excess kurtosis).

### 3.2 Temporal features

- Mean Absolute Deviation (MAD): `mean(abs(x - mean(x)))` pada tiap subcarrier dalam jendela.
- Temporal standard deviation: pada fitur yang dihasilkan saat ini, `temporal_std` sama persis dengan `std` untuk tiap subcarrier. Ini adalah redundansi yang benar-benar ada, bukan kebetulan.
- RMS: `sqrt(mean(x^2))`.
- Energy: `sum(x^2)`.
- Zero Crossing Rate (ZCR): `mean((x_t - mean) * (x_{t+1} - mean) < 0)`. Artinya, ZCR dihitung relatif terhadap mean jendela (titik referensi mean), bukan terhadap nol absolut. Definisi ini jelas dan konsisten dengan kode yang dipakai.
- First Difference Mean: rata-rata dari selisih antar-sampel `diff = x[t+1] - x[t]`.
- First Difference Standard Deviation: `std(diff, ddof=1)`.

### 3.3 Cross-subcarrier features

- Mean amplitude antar-subcarrier: rata-rata nilai mean tiap subcarrier.
- Standard deviation antar-subcarrier: `std(sub_mean, ddof=1)`.
- Variance antar-subcarrier: `var(sub_mean, ddof=1)`.
- Median korelasi antar-subcarrier: median dari elemen atas segitiga matriks korelasi subcarrier dalam jendela.

### 3.4 Pemeriksaan khusus

1. `std` dan `temporal_std` identik untuk semua 52 subcarrier. Nilai maksimum selisih absolut adalah 0.0.
2. Kondisi ini bukan bug yang disembunyikan: fitur tersebut memang diduplikasi secara eksplisit dalam `compute_feature_vector()` sebelumnya.
3. Definisi MAD dan RMS sudah konsisten secara matematis; energy menggunakan jumlah kuadrat amplitudo (`sum(x^2)`), bukan rata-rata kuadrat.
4. Korelasi antar-subcarrier dihitung pada dimensi waktu dalam satu jendela, lalu median dari segitiga atas matriks korelasi diambil sebagai ringkasan lintas-subcarrier.
5. Nilai konstan tidak muncul pada matriks fitur baseline; tidak ada fitur dengan variance mendekati nol yang dapat dihapus tanpa analisis lebih lanjut.
6. Tidak ada NaN atau inf yang belum ditangani pada output matriks fitur final.

## 4. Timestamp quality analysis

Timestamp dianalisis setelah konversi ke objek datetime Python. Hasil aktual:

- Total baris valid: 78,618
- Total timestamp unik: 69,718
- Total duplikasi timestamp: 8,900
- Total interval nol: 8,900
- Total interval negatif: 0
- Distribusi interval nol muncul di 11 dari 15 rekaman; 4 rekaman tidak memiliki interval nol.
- `seq` tetap naik secara unik pada setiap rekaman, tanpa duplikasi atau urutan mundur.

Kesimpulan:

- Masalahnya bukan kehilangan urutan sampel atau pengacakan data.
- Masalahnya adalah nilai timestamp yang berulang, sementara `seq` tetap konsisten.
- Karena dataset tidak menjelaskan sumber duplikasi timestamp, statusnya diperlakukan sebagai keterbatasan data yang belum dijelaskan, bukan bukti bahwa pengukuran invalid.
- Penghapusan row yang memiliki timestamp berulang dapat kehilangan informasi temporal yang sebenarnya relevan; karena itu, pembersihan timestamp tidak dilakukan tanpa bukti penyebab yang jelas.

## 5. Analisis redundansi fitur

Pada baseline `feature_matrix_50.csv`:

- Jumlah fitur numerik: 992
- Jumlah pasangan fitur dengan korelasi absolut >= 0.90: 3,892
- Jumlah fitur konstan: 0
- Jumlah fitur dengan variance <= 1e-12: 0
- Fitur dengan variance terendah: `sub_x_first_diff_mean` untuk subcarrier tertentu, sekitar 0.002–0.004.

Pasangan redundan terkuat yang muncul secara konsisten adalah:

- `sub_i_std` vs `sub_i_temporal_std` untuk semua subcarrier `i`
- Ini menghasilkan korelasi tepat 1.0 dan merupakan contoh redundansi yang sudah diketahui dari definisi yang identik.

Rekomendasi:

- Pertahankan dataset baseline asli sebagai referensi.
- Jangan otomatis menghapus fitur hanya karena korelasinya tinggi.
- Fitur yang sangat terduplikasi seperti `std` dan `temporal_std` perlu ditinjau ulang saat masuk tahap pemodelan. Namun, penghapusan harus didokumentasikan dan disertai evaluasi dampak terhadap interpretasi temporal.

## 6. PCA setelah standardisasi

PCA dilakukan pada fitur numerik saja dan mengecualikan metadata/label. Sebelum PCA, fitur distandardisasi dengan z-score (mean 0, std 1) untuk setiap kolom. Ini setara dengan StandardScaler, tetapi dihitung manual agar tidak bergantung pada `scikit-learn` dan tetap transparan.

Hasil:

- PC1 explained variance ratio: 27.50%
- PC2 explained variance ratio: 12.21%
- Cumulative explained variance PC1+PC2: 39.71%
- PC1+PC2+...+PC5: 61.53%
- Diperlukan 39 komponen untuk mencapai 90% cumulative variance
- Diperlukan 71 komponen untuk mencapai 95% cumulative variance

Interpretasi yang proporsional:

- PCA hanya menunjukkan struktur variasi global antar-window dan bukan bukti klasifikasi okupansi.
- Struktur yang terlihat mungkin merepresentasikan variasi amplitudo dan temporal lebih daripada kelas yang benar-benar dibedakan.
- PCA sebaiknya digunakan untuk eksplorasi, bukan sebagai bukti bahwa model akan berhasil membedakan jumlah manusia.

## 7. Audit filtering

Filter yang diuji pada data CSI adalah:

- raw amplitude baseline
- moving average: 3, 5, 7, 11
- Savitzky-Golay: 5, 7, 11 dengan order 2
- median filter: 3, 5, 7
- EMA: alpha 0.1, 0.3, 0.5

Hasil kuantitatif dari `filter_comparison.csv` menunjukkan bahwa filter yang tampak paling halus belum tentu paling cocok untuk mempertahankan informasi temporal. Nilai `mae_vs_raw` dan `rms_vs_raw` memberikan indikator bahwa smoothing yang terlalu agresif dapat mengurangi variasi temporal yang mungkin informatif untuk okupansi.

Kesimpulan utama:

- Tidak ada ground truth denoising yang bersih untuk membuktikan filter mana yang terbaik secara definitif.
- Evaluasi filter harus memperhitungkan trade-off antara pengurangan noise dan pemeliharaan sinyal temporal.
- Untuk tahap ML masa depan, filter terbaik harus dipilih bukan hanya berdasarkan plot yang lebih halus, tetapi juga berdasarkan stabilitas dan validasi pada sesi yang terpisah.

## 8. Risiko data leakage dan rancangan pembagian data

Risiko leakage pada tahap eksperimen berikutnya adalah tinggi jika pembagian dilakukan secara acak pada baris atau window.

Aturan yang disarankan:

- `recording_id` harus dipertahankan sebagai unit pembagian.
- Semua window dari satu rekaman tetap berada di satu split.
- Jangan melakukan random row split atau random window split.
- Normalisasi/standarisasi hanya boleh dipelajari dari training set saat eksperimen ML dimulai.

Keterbatasan yang sudah jelas:

- Kelas 0 hanya memiliki 1 rekaman.
- Kelas 3 hanya memiliki 1 rekaman.
- Karena itu, evaluasi yang benar-benar independen untuk kelas 0 dan 3 tidak mungkin dipastikan saat ini.
- Pembagian grup-based dapat dilakukan untuk rekaman yang tersedia, tetapi hasil generalisasi tetap terbatas.

## 9. Rekomendasi untuk tahap Machine Learning

1. Tetap gunakan baseline `feature_matrix_50.csv` sebagai referensi dan simpan salinan asli.
2. Gunakan split berbasis recording/session di tahap berikutnya, bukan split acak.
3. Untuk setiap fold, hanya hitung parameter normalisasi dari data training.
4. Evaluasi filter dan fitur pada tingkat sesi, bukan hanya berdasarkan plot visual.
5. Pertimbangkan fitur yang paling stabil dan kurang redundan, terutama sebelum melakukan reduksi dimensi atau model yang sensitif terhadap multikolinearitas.
6. Tingkatkan jumlah sesi per kelas sebelum mengklaim kemampuan generalisasi yang kuat.

## 10. Keterbatasan dataset

- Kelas 0 dan 3 masing-masing hanya memiliki satu sesi.
- Kelas tidak seimbang: 1, 7, 6, dan 1 rekaman untuk kelas 0,1,2,3.
- Terdapat timestamp berulang yang tidak dijelaskan di dataset.
- Label kelas berkaitan dengan urutan sesi dan waktu, bukan eksperimen yang benar-benar interleaved.
- Tidak ada bukti bahwa perubahan CSI dapat dipandang sebagai sebab-akibat langsung dari jumlah manusia tanpa kontrol eksperimen tambahan.

## 11. Kesimpulan

Audit ini menegaskan bahwa baseline preprocessing dan fitur di WiCount sudah masuk ke tahap validasi yang sehat: ada validasi data, metadata dijaga, fitur dihitung dan tersimpan, serta tingkat redundansi dan risiko leakage telah didokumentasikan. Namun, proyek masih belum siap untuk klaim performa klasifikasi. Tahap berikutnya harus menggunakan data split berbasis sesi, analisis yang lebih hati-hati terhadap temporal features, dan pengumpulan sesi tambahan sebelum ML dilakukan.
