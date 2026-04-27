# Recon Continued: Analisis Fungsi Administratif & Dead Code

Materi ini melanjutkan proses peninjauan kode (code review) pada fungsi-fungsi yang tersisa di protokol Puppy Raffle.

### 1. Fungsi `changeFeeAddress`
Fungsi ini digunakan untuk mengubah alamat penerima biaya (fees).
*   **Akses Kontrol:** Menggunakan modifier `onlyOwner` dari OpenZeppelin. Auditor harus memastikan modifier ini terimplementasi dengan benar (dan memang benar).
*   **Urutan Eksekusi:** Fungsi ini mengikuti pola yang benar: mengubah state (`feeAddress = newFeeAddress`) baru kemudian memancarkan event (`emit FeeAddressChanged`).
*   **Catatan Auditor:** Selalu periksa apakah fungsi lain mengikuti urutan ini (CEI - Checks, Effects, Interactions).

### 2. Temuan Dead Code: `_isActivePlayer`
Auditor menemukan fungsi internal `_isActivePlayer()` yang bertujuan mengecek apakah `msg.sender` ada dalam array `players`.
*   **Masalah:** Setelah diperiksa secara menyeluruh, fungsi ini **tidak dipanggil atau digunakan di mana pun** dalam protokol.
*   **Dampak:** Membuang gas saat deployment dan menambah kompleksitas kode tanpa alasan.
*   **Severity:** `Informational` atau `Gas`. Ini adalah temuan valid yang menunjukkan kurangnya pembersihan kode (code cleanup).

### 3. Metadata & NFT: `_baseURI` dan `tokenURI`
*   **`_baseURI`:** Fungsi sederhana yang mengembalikan prefix untuk metadata on-chain. Auditor menyarankan ini bisa dijadikan variabel `constant` atau `immutable` untuk menghemat gas.
*   **`tokenURI`:** Fungsi ini menangani pembuatan metadata NFT (SVG/Base64).
    *   **Hal yang perlu diperiksa:** Kelangkaan (rarity) token, pemetaan `rarityToUri`, dan integritas encoding Base64.
    *   **Tantangan Auditor:** "Bagaimana cara merusak fungsi view ini?". Misalnya, apakah ada kemungkinan pembagian dengan nol atau array out-of-bounds saat menentukan rarity?

### 4. Menjawab Pertanyaan dari Catatan Sebelumnya
Tahap ini adalah saat yang tepat untuk memverifikasi asumsi dan pertanyaan yang terkumpul di awal audit:
*   **Versi Solidity:** "Apakah custom error didukung di Solidity 0.7.6?". Jawabannya: **Tidak** (Custom error baru ada di 0.8.4+). Maka penggunaan `require` dengan string adalah hal yang wajar untuk versi ini.
*   **Edge Cases:** Apa yang terjadi jika `players.length == 0`? Apakah event tetap dipancarkan?
*   **Redundansi:** Apakah ada kode yang bisa disederhanakan untuk menghemat gas?

### 5. Kesimpulan Tahap Pertama (First Pass)
Satu kali pembacaan kode (one pass) biasanya tidak cukup. Auditor harus melakukan beberapa kali iterasi:
1.  **Pass 1:** Memahami alur dan menandai hal mencurigakan.
2.  **Pass 2:** Menjawab pertanyaan teknis dan mencari exploit secara spesifik.
3.  **Pass 3:** Verifikasi perbaikan dan penulisan laporan.

Jika masih banyak pertanyaan yang belum terjawab, itu adalah sinyal bahwa auditor harus menggali lebih dalam (go deeper).
