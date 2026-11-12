# AGENTS.md

## Instruksi permanen untuk AI coding agent

1. Baca `PRD.md` sebelum mengerjakan fitur atau perubahan apapun.
2. Pertahankan arsitektur utama sistem: dua ESP32 sebagai transmitter (TX) dan receiver (RX).
3. Sensor HC-SR04 bukan komponen wajib dalam sistem inti. Jika digunakan, itu harus diperlakukan sebagai eksperimen alternatif atau pembanding yang terpisah.
4. Jangan mengubah format data CSI tanpa dokumentasi dan validasi yang terdokumentasi.
5. Jangan mengarang hasil eksperimen, dukungan hardware, akurasi model, atau data hasil pengujian.
6. Hindari dependensi baru tanpa alasan yang jelas dan tanpa persetujuan.
7. Gunakan perubahan sekecil mungkin untuk mencapai tujuan.
8. Perbarui `changelog.md` setiap kali ada perubahan pada struktur atau dokumentasi.
9. Perbarui `roadmap.md` jika status pekerjaan berubah.
10. Jangan menghapus pekerjaan yang sudah ada.
11. Jangan melakukan commit atau push tanpa perintah pengguna.
12. Fokus pada struktur folder dan dokumentasi fondasi sesuai PRD pada tahap awal.
13. Jika informasi hardware, toolchain, atau eksperimen belum diketahui, gunakan `TODO` atau `TBD` dan jangan menebak.
14. Tetap jujur dalam melaporkan status, batasan, dan hasil yang belum teruji.

## Prinsip kerja

- Selalu mulai dari PRD dan dokumentasi yang ada.
- Prioritaskan reproduktifitas, traceability, dan modularity.
- Jaga data mentah dan hasil preprocessing tetap terpisah.
- Hindari commit besar atau artifact yang belum disetujui.
- Pastikan setiap perubahan aman, kecil, dan terdokumentasi.
