# Panduan Analisis Statis: Aderyn Walkthrough

Materi ini membahas penggunaan **Aderyn**, alat analisis statis berbasis Rust yang menghasilkan laporan dalam format Markdown yang rapi dan mudah dibaca.

### 1. Menjalankan Aderyn
Gunakan perintah berikut di root direktori proyek:
```bash
aderyn .
```
Setelah dijalankan, Aderyn akan menghasilkan file `report.md` yang berisi ringkasan temuan.

---

### 2. Temuan Medium Severity
*   **Centralization Risk:** Aderyn menandai fungsi yang menggunakan modifier `onlyOwner` (seperti `changeFeeAddress`).
    *   *Analisis:* Ini menunjukkan bahwa protokol bergantung pada satu pihak (pemilik). Jika private key pemilik bocor, penyerang bisa mengubah alamat biaya. Dalam audit privat, hal ini harus dilaporkan sebagai risiko sentralisasi.

### 3. Temuan Low Severity
*   **`abi.encodePacked()` dengan Tipe Dinamis:** Aderyn memperingatkan penggunaan `encodePacked` di dalam `keccak256` jika melibatkan lebih dari satu tipe dinamis (seperti string atau bytes).
    *   *Risiko:* Dapat menyebabkan **Hash Collision**, di mana input yang berbeda menghasilkan hash yang sama. Disarankan menggunakan `abi.encode()` untuk keamanan lebih tinggi.
*   **Floating Pragma:** Sama seperti Slither, Aderyn mendeteksi penggunaan rentang versi Solidity yang terlalu luas (`^0.7.6`).

### 4. Temuan Informational & Gas Optimization
*   **Missing Zero Address Checks:** Mengulangi temuan Slither tentang pentingnya memvalidasi alamat agar tidak bernilai `address(0)`.
*   **Visibility Optimization:** Aderyn menyarankan fungsi `public` yang tidak pernah dipanggil secara internal (seperti `enterRaffle`, `refund`, dan `tokenURI`) untuk diubah menjadi `external`.
    *   *Manfaat:* Menghemat sedikit gas karena parameter fungsi `external` dibaca langsung dari `calldata` bukan disalin ke `memory`.
*   **Magic Numbers:** Menemukan angka-angka mentah dalam loop dan kalkulasi persentase yang seharusnya dijadikan konstanta.
*   **Non-Indexed Events:** Menandai event yang parameter alamatnya tidak menggunakan kata kunci `indexed`.
    *   *Dampak:* Tanpa `indexed`, aplikasi off-chain (seperti The Graph atau front-end) akan kesulitan memfilter data event tertentu secara efisien.

---

### 5. Kelebihan Aderyn
*   **Format Laporan:** Hasilnya sudah berupa Markdown yang siap digunakan dalam laporan audit profesional.
*   **Ketajaman Deteksi:** Aderyn sering kali mendeteksi masalah logika sentralisasi dan praktik terbaik yang mungkin dilewatkan oleh alat lain.

### Kesimpulan
Aderyn adalah pelengkap yang sangat baik untuk Slither. Meskipun banyak temuannya yang tumpang tindih, format outputnya yang "siap pakai" sangat membantu auditor dalam mempercepat proses dokumentasi laporan. Selalu ingat untuk memvalidasi setiap temuan secara manual karena alat statis bisa bersifat terlalu ketat (*over-vigilant*).
