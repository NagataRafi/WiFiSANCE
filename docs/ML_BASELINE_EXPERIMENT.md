# ML Baseline Experiment — WiCount

Tanggal eksekusi: 2026-11-13

## 1. Tujuan

Menguji tiga model klasifikasi baseline dan dua skenario fitur pada feature matrix WiCount yang sudah tersedia. Validasi dilakukan dengan Leave-One-Recording-Out terbatas pada kelas okupansi 1 dan 2. Eksperimen ini merupakan pilot pada dataset eksternal WiCount, bukan hasil pengujian hardware ESP32 WiFiSENSE dan bukan validasi empat kelas.

Notebook: `analysis/notebooks/ml_baseline_experiment.ipynb`.

## 2. Sumber dan karakteristik dataset

Input model berasal dari `analysis/processed/features/feature_matrix_50.csv`, bukan dengan menghitung ulang DSP atau feature extraction.

- Bentuk matriks: 3.122 window x 999 kolom.
- Fitur numerik: 992.
- Metadata yang tidak digunakan sebagai input: `window_id`, `recording_id`, `occupancy`, `window_size`, `start_index`, `end_index`, `row_count`.
- Recording per kelas: class 0 = 1, class 1 = 7, class 2 = 6, class 3 = 1.
- Window per kelas: class 0 = 207, class 1 = 1.452, class 2 = 1.268, class 3 = 195.
- Kelas 0 dan 3 adalah singleton recording; hasil evaluasi tidak memberikan true validation samples untuk keduanya.

Dataset WiCount adalah data referensi eksternal. Hasilnya tidak membuktikan dukungan, performa, atau akurasi pada ESP32 milik WiFiSENSE.

## 3. Strategi group validation

Notebook membuat 13 fold:

- 7 fold menahan satu recording kelas 1 secara bergantian.
- 6 fold menahan satu recording kelas 2 secara bergantian.
- Recording kelas 0 dan 3 selalu masuk training.
- Setiap fold memiliki 14 recording training dan 1 recording validation.
- Training berisi 2.898–2.932 window; validation berisi 190–224 window.
- Semua window dari recording yang ditahan menjadi validation. Tidak ada `recording_id` yang muncul sekaligus di training dan validation.

Dengan demikian, setiap model dan skenario fitur menghasilkan out-of-fold (OOF) prediksi untuk 13 recording dan 2.720 window, semuanya dari kelas 1 atau 2. Kelas 0/3 tidak memiliki evaluasi independen. Confusion matrix tetap menunjukkan empat label, dengan support true validation kelas 0 dan 3 sama dengan nol.

## 4. Fitur yang digunakan

### Full

Semua 992 kolom numerik pada feature matrix.

### Compact candidate

211 kolom dengan `recommendation_tier == candidate_core` dari `analysis/processed/features/feature_reduction_recommendation.csv`. Notebook memverifikasi jumlah, nama, keberadaan kolom, dan memisahkan metadata sebelum fit.

Daftar compact candidate dibekukan sebelum eksperimen ini dan tidak diseleksi berdasarkan skor atau label validation fold. Namun, rekomendasi sebelumnya berasal dari audit eksploratif terhadap seluruh feature matrix. Karena itu, perbandingan compact merupakan benchmark pilot dengan candidate set yang sudah ditetapkan, bukan evaluasi nested feature selection. Untuk estimasi seleksi fitur yang sepenuhnya bebas dari informasi validation, candidate set perlu ditentukan ulang pada training fold saja.

## 5. Konfigurasi model dan preprocessing

Random state tetap 42. Tidak dilakukan hyperparameter tuning luas.

| Model | Konfigurasi | Preprocessing |
|---|---|---|
| Logistic Regression | `C=1.0`, `solver=lbfgs`, `max_iter=2000`, `class_weight=balanced`, `random_state=42` | `StandardScaler` di dalam pipeline fold |
| Random Forest | `n_estimators=100`, `max_features=sqrt`, `class_weight=balanced_subsample`, `random_state=42`, `n_jobs=1` | Tidak memakai scaler |
| RBF-SVM | `C=1.0`, `gamma=scale`, `class_weight=balanced` | `StandardScaler` di dalam pipeline fold |

Class weights dihitung oleh estimator hanya dari label training pada fold bersangkutan. Pipeline scaler dibuat dan di-fit ulang pada training fold saja; notebook memeriksa `n_samples_seen_` sebelum memprediksi validation. Validation tidak digunakan untuk fit scaler, model, PCA, atau pemilihan fitur.

Lingkungan: Python 3.14.7, scikit-learn 1.9.1, NumPy 2.5.2, pandas 3.0.5. Versi scikit-learn dicatat di `analysis/requirements.txt` dan `analysis/outputs/ml_baseline/run_manifest.json`.

## 6. Hasil pooled out-of-fold

Accuracy dan macro-F1 dihitung hanya untuk true validation classes 1 dan 2. Precision/recall kelas 0 dan 3 tidak dilaporkan karena tidak ada true validation support untuk kelas tersebut.

| Fitur | Model | OOF accuracy | Macro-F1 (1,2) | Precision 1 | Recall 1 | Precision 2 | Recall 2 | Mean akurasi per recording |
|---|---|---:|---:|---:|---:|---:|---:|---:|
| Full | Logistic Regression | 0.7989 | 0.8291 | 0.8635 | 0.8753 | 0.8852 | 0.7114 | 0.8011 |
| Full | Random Forest | 0.8551 | 0.8582 | 0.8540 | 0.9022 | 0.8804 | 0.8013 | 0.8566 |
| Full | RBF-SVM | 0.8327 | 0.8587 | 0.8809 | 0.9167 | 0.9220 | 0.7366 | 0.8346 |
| Compact | Logistic Regression | 0.7886 | 0.8253 | 0.8712 | 0.8760 | 0.8917 | 0.6885 | 0.7907 |
| Compact | Random Forest | **0.8596** | **0.8637** | 0.8681 | 0.8884 | 0.8733 | **0.8265** | 0.8609 |
| Compact | RBF-SVM | 0.8173 | 0.8496 | 0.8843 | 0.9056 | 0.9172 | 0.7161 | 0.8190 |

Compact Random Forest mendapat skor pooled tertinggi pada eksperimen ini, tetapi selisih terhadap full Random Forest kecil: accuracy +0.0045 dan macro-F1 +0.0055. Compact tidak unggul pada Logistic Regression atau RBF-SVM. Hasil ini tidak cukup untuk menyatakan compact feature set secara umum lebih baik.

Rata-rata waktu fit per fold pada eksekusi ini: full Logistic Regression 4.64 s, full Random Forest 6.28 s, full RBF-SVM 3.56 s, compact Logistic Regression 1.71 s, compact Random Forest 3.98 s, dan compact RBF-SVM 0.64 s. Waktu ini bergantung pada lingkungan eksekusi; rincian fit dan prediction per fold tersimpan di `fold_metrics.csv`.

### Hasil accuracy setiap fold

Tabel berikut memperlihatkan accuracy untuk setiap recording yang ditahan dan seluruh enam kombinasi fitur/model. Precision, recall, F1 kelas yang didukung, support, waktu fit, dan waktu prediksi setiap kombinasi fold tersedia lengkap di `analysis/outputs/ml_baseline/fold_metrics.csv`.

| Kelas | Recording ditahan (waktu) | Window | Full LR | Full RF | Full SVM | Compact LR | Compact RF | Compact SVM |
|---:|---|---:|---:|---:|---:|---:|---:|---:|
| 1 | 19:27:21.930 | 190 | 0.9789 | 0.9632 | 0.9789 | 0.9632 | 0.9474 | 0.9474 |
| 1 | 19:31:17.706 | 206 | 0.7039 | 0.8738 | 0.7573 | 0.6990 | 0.8592 | 0.7184 |
| 1 | 19:34:42.783 | 208 | 0.9567 | 0.9183 | 0.9663 | 0.9567 | 0.9135 | 0.9519 |
| 1 | 19:37:36.861 | 205 | 0.9707 | 0.9659 | 0.9854 | 0.9756 | 0.9707 | 0.9902 |
| 1 | 19:40:12.110 | 215 | 0.8000 | 0.9674 | 0.9302 | 0.7907 | 0.9116 | 0.9209 |
| 1 | 19:42:28.267 | 224 | 0.8750 | 0.8170 | 0.8929 | 0.8884 | 0.8125 | 0.9062 |
| 1 | 19:45:07.501 | 204 | 0.8529 | 0.8186 | 0.9118 | 0.8676 | 0.8137 | 0.9069 |
| 2 | 19:47:14.669 | 214 | 0.3551 | 0.4533 | 0.4579 | 0.3645 | 0.4860 | 0.4346 |
| 2 | 19:49:37.940 | 211 | 0.4976 | 0.5640 | 0.5450 | 0.4882 | 0.6777 | 0.5355 |
| 2 | 19:54:01.269 | 204 | 0.8578 | 0.8971 | 0.8824 | 0.8039 | 0.9020 | 0.8480 |
| 2 | 19:56:22.903 | 217 | 0.8664 | 0.9770 | 0.8802 | 0.8203 | 0.9585 | 0.8571 |
| 2 | 20:02:00.329 | 207 | 0.9227 | 0.9710 | 0.9082 | 0.8889 | 0.9662 | 0.9082 |
| 2 | 20:04:39.406 | 215 | 0.7767 | 0.9488 | 0.7535 | 0.7721 | 0.9721 | 0.7209 |

Fold accuracy bervariasi tajam antar-recording. Untuk seluruh enam konfigurasi, recording kelas 2 `19:47:14.669` merupakan fold paling sulit dengan accuracy 0.3551–0.4860. Skor kelas 1 umumnya lebih tinggi, tetapi juga bervariasi. Ini menunjukkan metrik pooled menutupi heterogenitas recording.

## 7. Confusion matrix dan analisis kesalahan

Confusion matrix empat kelas dibuat dari pooled OOF prediction. Baris true class 0 dan 3 memiliki support nol karena kedua recording singleton selalu berada di training. Prediksi model ke kolom 0 atau 3 tetap terlihat, tetapi tidak dapat ditafsirkan sebagai recall/precision tervalidasi untuk kelas tersebut.

Pada compact Random Forest (konfigurasi dengan pooled score tertinggi):

- True kelas 1: 1.290 benar, 152 salah sebagai kelas 2, dan 10 sebagai kelas 0.
- True kelas 2: 1.048 benar, 196 salah sebagai kelas 1, dan 24 sebagai kelas 3.

Pertukaran kelas 1↔2 menjadi pola kesalahan utama. Sebagian prediksi kelas 2 jatuh pada kelas 3, padahal tidak ada true validation class 3; ini hanya kesalahan terhadap label pada data evaluasi kelas 1/2 dan bukan pengukuran kemampuan untuk kelas 3. Model tidak boleh dianggap tervalidasi untuk kelas 0 atau 3.

## 8. Artefak eksperimen

- `analysis/notebooks/ml_baseline_experiment.ipynb`
- `analysis/outputs/ml_baseline/fold_metrics.csv`
- `analysis/outputs/ml_baseline/model_comparison.csv`
- `analysis/outputs/ml_baseline/oof_predictions.csv`
- `analysis/outputs/ml_baseline/confusion_matrix.png`
- `analysis/outputs/ml_baseline/feature_comparison.png`
- `analysis/outputs/ml_baseline/fold_plan.csv`
- `analysis/outputs/ml_baseline/error_analysis.csv`
- `analysis/outputs/ml_baseline/recording_oof_metrics.csv`
- `analysis/outputs/ml_baseline/run_manifest.json`

## 9. Keterbatasan dan interpretasi

1. Tidak ada evaluasi independen untuk kelas 0 dan 3; macro-F1/precision/recall hanya mencakup kelas 1 dan 2.
2. Kelas 0/3 tetap ada di seluruh training fold, sehingga model dapat memprediksi keduanya tetapi performa kelas itu tidak dapat diuji.
3. Hanya tersedia 7 recording kelas 1 dan 6 recording kelas 2; 13 fold mengukur variasi recording terbatas, bukan generalisasi ke ruangan, hari, hardware, atau kondisi baru.
4. Label direkam dalam blok waktu yang berurutan pada dataset WiCount. Perbedaan sesi/waktu dapat menjadi faktor pengganggu dan tidak dapat dipisahkan dari okupansi menggunakan data ini saja.
5. Candidate compact berasal dari rekomendasi QA terdahulu. Walau tidak dipilih berdasarkan skor validation atau target, rekomendasi itu dibuat dari audit seluruh feature matrix; benchmark compact bukan nested feature-selection evaluation.
6. Overlap window membuat pembagian acak window berisiko leakage; skema ini mencegahnya dengan menahan seluruh recording.
7. Durasi fit/predict hanya pengukuran pada lingkungan ini dan dapat berubah menurut hardware/beban sistem. Konfigurasi serta versi paket dicatat di run manifest.
8. Dataset merupakan WiCount eksternal, bukan hasil akuisisi dua ESP32 WiFiSENSE. Tidak ada deployment, firmware, backend, atau mobile yang diuji.

## 10. Rekomendasi pengumpulan data berikutnya

- Kumpulkan beberapa recording independen untuk setiap okupansi, terutama kelas 0 dan 3.
- Interleave atau acak urutan kelas saat perekaman agar label tidak berimpit dengan urutan waktu.
- Catat sesi, tanggal, ruangan, posisi TX/RX, aktivitas, konfigurasi, dan versi firmware.
- Setelah data baru tersedia, tetapkan group-based train/validation/test berdasarkan recording atau sesi. Gunakan training-fold-only fitting untuk scaler, seleksi fitur, PCA, dan transformasi lain.
- Bandingkan compact dan full melalui fold yang sama, lalu pertimbangkan feature selection nested bila candidate set dipelajari dari data.
- Validasi terpisah pada data yang direkam oleh perangkat ESP32 TX/RX WiFiSENSE sebelum membuat klaim tentang sistem proyek.

Eksperimen ini adalah baseline terukur pada cakupan terbatas; tidak ada klaim akurasi universal atau generalisasi empat kelas.
