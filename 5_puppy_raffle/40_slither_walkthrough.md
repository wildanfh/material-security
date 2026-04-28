# Panduan Analisis Statis: Slither Walkthrough

Materi ini membahas cara menggunakan **Slither** untuk mendeteksi kerentanan secara otomatis dan cara menginterpretasikan hasilnya pada protokol Puppy Raffle.

### 1. Menjalankan Slither
Gunakan perintah berikut di root direktori proyek:
```bash
slither .
```
Tips: Gunakan `--exclude-dependencies` untuk mengabaikan peringatan yang muncul dari library eksternal (seperti OpenZeppelin) agar fokus pada kode utama.

---

### 2. Temuan High Severity
*   **Arbitrary Send ETH:** Slither mendeteksi pengiriman ETH ke alamat yang bisa diubah (`feeAddress`). 
    *   *Validasi:* Dalam PuppyRaffle, ini dianggap disengaja karena hanya `owner` yang bisa mengubahnya.
*   **Weak PRNG:** Deteksi penggunaan `block.timestamp` dan `block.difficulty` untuk keacakan.
    *   *Status:* **Valid.** Ini mengonfirmasi temuan manual kita sebelumnya.

### 3. Temuan Medium Severity
*   **Divide before Multiply:** Ditemukan pada library Base64. Melakukan pembagian sebelum perkalian dapat menyebabkan hilangnya presisi secara signifikan.
*   **Incorrect Equality:** Slither menandai `address(this).balance == totalFees`.
    *   *Status:* **Valid.** Ini adalah kerentanan "Mishandling of ETH" yang dapat menyebabkan DoS.
*   **Reentrancy:** Mendeteksi pemanggilan eksternal sebelum perubahan state pada fungsi `refund`.

### 4. Temuan Low & Informational
*   **Missing Zero Address Check:** Slither menyarankan pengecekan `address(0)` saat menyetel `feeAddress` di constructor atau fungsi ubah alamat. Ini adalah praktik *input validation* yang baik.
*   **Event Reentrancy:** Meskipun tidak mencuri dana, reentrancy pada event dapat memanipulasi urutan log yang diandalkan oleh aplikasi off-chain (seperti front-end atau indexer).
*   **Floating Pragma:** Penggunaan `^0.7.6` dianggap tidak aman karena tidak mengunci versi compiler yang spesifik.
*   **Dead Code:** Mengonfirmasi bahwa fungsi `_isActivePlayer` tidak pernah digunakan.

### 5. Temuan Optimasi Gas
Slither sangat ahli dalam menemukan penghematan gas:
*   **Cached Array Length:** Loop yang menggunakan `players.length` berkali-kali sangat boros gas karena membaca storage berulang kali. Seharusnya panjang array disimpan di variabel lokal (memory) sebelum loop.
*   **Constant & Immutable Candidates:** Variabel seperti gambar URI dan `raffleDuration` tidak pernah berubah setelah disetel, sehingga harus ditandai sebagai `constant` atau `immutable`.

---

### 6. Cara Mengelola Laporan Slither
Jika sebuah temuan dianggap bukan masalah (*false positive*) atau sudah dicatat, kita bisa menyembunyikan peringatan tersebut agar laporan tetap bersih:

**A. Menggunakan Komentar (In-code):**
```javascript
// slither-disable-next-line detector-name
(bool success,) = feeAddress.call{value: feesToWithdraw}("");
```

**B. Menggunakan File Konfigurasi:**
Buat file `slither.config.json` untuk mengecualikan detektor tertentu secara global (misal: `solc-version`).

### Kesimpulan
Slither adalah "jaring pengaman" yang luar biasa. Ia menemukan hal-hal yang mungkin terlewat saat manual review, terutama optimasi gas dan pola-pola teknis seperti *cached array length*. Namun, auditor tetap harus memvalidasi setiap temuan untuk menentukan dampak sebenarnya bagi protokol.
