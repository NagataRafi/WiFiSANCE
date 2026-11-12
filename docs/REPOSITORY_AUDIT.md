# Repository Audit

Dokumen ini berfungsi sebagai catatan audit awal terhadap repository referensi yang berpotensi dipelajari untuk modul terkait. Pada tahap awal, repository tersebut masih dianggap referensi, bukan sebagai sumber yang sudah diintegrasikan ke proyek.

| Repository | Modul yang relevan | Lisensi | Kompatibilitas | Status verifikasi | Catatan |
|---|---|---|---|---|---|
| https://github.com/espressif/esp-csi | Firmware ESP32, CSI acquisition, ESP-IDF | TBD | TBD | Not verified | Repository referensi resmi untuk CSI pada platform ESP32. Perlu audit terhadap API, dukungan board, dan dokumentasi. |
| https://github.com/everythingfreee/ESP32-WiFi-CSI-Human-Sensing | CSI parsing, preprocessing, feature extraction, human sensing | TBD | TBD | Not verified | Repo ini relevan untuk pemahaman pipeline CSI dan estimasi okupansi. Perlu pengecekan lisensi dan adaptasi kode. |
| https://github.com/venkatasasikiranreddykotapati/wifi-csi-occupancy | Dataset, feature engineering, ML baseline | TBD | TBD | Not verified | Relevan pada dataset dan evaluasi occupancy. Tidak dianggap sudah diintegrasikan. |
| https://github.com/yasarkaleem/wifi-sense | Backend/dashboard integration reference | TBD | TBD | Not verified | Relevan untuk pola backend dan tampilan data. Harus dicek pendekatan dan dependensi yang dipakai. |
| https://github.com/cvwifi-lab/WiVi32-A_People_Counting_Tool | Human counting, signal processing, comparison study | TBD | TBD | Not verified | Repo ini dapat dipakai sebagai referensi metodologi, namun tidak otomatis diadopsi. |

## Catatan proses audit

Sebelum mengadopsi kode dari repo eksternal, tim perlu memastikan:

- lisensi dapat diterima dan atribusi jelas;
- struktur kode sesuai kebutuhan modul;
- perubahan yang dibutuhkan terdokumentasi;
- kompatibilitas dengan hardware yang digunakan dipastikan;
- hasil eksperimen tidak diklaim sebelum diverifikasi.

Semua verifikasi referensi akan dilakukan di fase berikutnya dan tidak dinyatakan selesai pada inisialisasi repository ini.
