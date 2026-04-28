# Temuan Informational & Gas Optimization

Selain temuan keamanan (vulnerability), auditor juga memberikan rekomendasi mengenai kualitas kode (*clean code*), standar industri, dan efisiensi gas.

### 1. Konvensi Penamaan (Naming Conventions)
*   **Masalah:** Variabel storage dalam `PuppyRaffle` tidak memiliki pembeda visual yang jelas dari variabel lokal.
*   **Rekomendasi:** Menggunakan prefix seperti `s_` untuk variabel storage (misal: `s_players`, `s_entranceFee`).
*   **Manfaat:** Memudahkan auditor dan pengembang lain untuk mengidentifikasi kapan sebuah fungsi melakukan operasi baca/tulis ke storage yang mahal biayanya.

### 2. Floating Pragma
*   **Masalah:** Penggunaan `pragma solidity ^0.7.6;`. Simbol `^` (caret) berarti kontrak dapat dikompilasi dengan versi Solidity yang lebih baru dari 0.7.6.
*   **Risiko:** Versi compiler yang berbeda mungkin memiliki bug atau perilaku yang berbeda, yang bisa memengaruhi integritas kontrak saat di-deploy.
*   **Rekomendasi:** Gunakan versi yang dikunci (locked pragma), misalnya `pragma solidity 0.7.6;`, untuk menjamin konsistensi antara lingkungan pengembangan, pengujian, dan produksi.

### 3. Angka Ajaib (Magic Numbers)
*   **Masalah:** Penggunaan angka mentah tanpa konteks dalam logika kalkulasi.
    ```javascript
    uint256 prizePool = (totalAmountCollected * 80) / 100;
    ```
*   **Risiko:** Kode sulit dibaca dan dipelihara. Tidak jelas apa arti `80` atau `100` bagi orang yang baru membaca kode.
*   **Rekomendasi:** Gunakan konstanta bernama:
    ```javascript
    uint256 public constant PRIZE_POOL_PERCENTAGE = 80;
    uint256 public constant FEE_PERCENTAGE = 20;
    uint256 public constant POOL_PRECISION = 100;
    ```

### 4. Audit Library Eksternal
*   **Langkah Penting:** Selalu periksa versi library (seperti OpenZeppelin) yang digunakan dalam `foundry.toml` atau `package.json`.
*   **Tindakan:** Bandingkan versi tersebut dengan [OpenZeppelin Security Advisories](https://github.com/OpenZeppelin/openzeppelin-contracts/security). Seringkali bug ditemukan pada versi library lama yang masih digunakan oleh protokol.

### 5. Optimasi Gas (Gas Savings)
Meskipun opsional, rekomendasi gas sangat dihargai oleh klien:
*   **`constant` & `immutable`:** Variabel seperti `raffleDuration` yang nilainya hanya diset sekali dan tidak pernah berubah harus dideklarasikan sebagai `immutable`.
    *   **Dampak:** Menghemat ribuan unit gas karena nilai tersebut akan disimpan langsung di dalam bytecode kontrak, bukan di storage.
*   **Penyimpanan Variabel:** Hindari membaca variabel storage berulang kali di dalam loop; lebih baik simpan di variabel lokal (memory/stack).

### 6. Dokumentasi (NatSpec)
Pastikan dokumentasi NatSpec (`/// @notice`, `/// @param`) akurat. Auditor menemukan beberapa NatSpec yang merujuk pada parameter yang tidak ada, yang harus diperbaiki untuk menghindari kebingungan.
