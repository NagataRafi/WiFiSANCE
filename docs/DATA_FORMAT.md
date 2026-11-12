# Data Format Specification

Dokumen ini menjelaskan format internal data CSI yang akan dipakai oleh WiFiSENSE. Karena tidak ada contoh data nyata yang sudah diverifikasi pada tahap awal, banyak detail spesifik masih dinyatakan sebagai `TBD`.

## Prinsip format data

- Data mentah harus disimpan terpisah dari data hasil preprocessing.
- Tiap file CSI harus melengkapi metadata sesi dan waktu akuisisi.
- Format internal harus konsisten dan tidak berubah tanpa dokumentasi.
- Struktur data harus mengikuti kebutuhan pipeline dari ESP32 RX ke model ML.

## Struktur data yang disarankan

```text
session_id
timestamp
device_role
device_model
sample_index
raw_csi_payload
metadata:
  - room/location
  - tx_device
  - rx_device
  - firmware_version
  - configuration
  - experiment_note
```

## Field yang masih TBD

Karena validasi firmware dan contoh data CSI belum tersedia, field berikut tetap belum final:

- jumlah subcarrier yang benar-benar dipakai;
- sampel per frame atau per paket;
- skema representasi kompleks CSI (real/imaginary, magnitude, phase, atau kombinasi);
- sampling rate yang dipakai dalam eksperimen;
- format file raw (CSV, JSON, binary, atau log text);
- struktur array atau tabel untuk tiap packet/frame.

## Rekomendasi pengelolaan dataset

1. Simpan CSI mentah di `data/raw/`.
2. Simpan hasil preprocessing di `data/processed/`.
3. Simpan dataset berlabel di `data/labeled/`.
4. Catat metadata yang relevan pada setiap file atau manifest sesi.
5. Hindari menimpa data mentah dengan hasil transformasi.

## Catatan penting

Tujuan dari format ini bukan untuk mengklaim struktur final dari device yang belum teruji. Dokumen ini berfungsi sebagai spesifikasi awal yang bisa diperbarui setelah data real dan perangkat valid terpasang.
