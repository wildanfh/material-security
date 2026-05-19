# Manual Review of TSwap – Pentingnya Pemeriksaan Manual setelah Fuzzing

## Pendahuluan

Setelah menghabiskan waktu yang cukup lama untuk menyiapkan dan menjalankan *stateful fuzzing suite* pada TSwap, kita menemukan sebuah bug tersembunyi (transfer token ekstra setiap 10 swap). Namun, penting untuk diingat bahwa **fuzzing tidak menggantikan manual review**. Keduanya saling melengkapi.

---

## Mengapa Manual Review Tetap Diperlukan?

| Aspek | Fuzzing | Manual Review |
|-------|---------|---------------|
| **Pendekatan** | Otomatis, acak, berdasarkan invariant | Sistematis, kontekstual, membaca kode baris per baris |
| **Kelebihan** | Menemukan edge case dan interaksi tak terduga | Memahami logika bisnis, arsitektur, dan asumsi desain |
| **Kekurangan** | Hanya sebaik invariant yang didefinisikan; bisa kehilangan bug logika tingkat tinggi | Membutuhkan waktu dan keahlian; bisa terlewat karena kompleksitas |
| **Contoh temuan** | Pelanggaran invariant akibat transfer ekstra | Dokumentasi kurang, potensi reentrancy, kontrol akses, dll. |

---

## Langkah Selanjutnya

1. **Beristirahat sejenak** – materi manual review akan cukup intensif.
2. **Membaca `TSwapPool.sol` secara teliti** – cari potensi kerentanan lain yang mungkin tidak terdeteksi fuzzing, seperti:
   - *Access control* pada fungsi-fungsi kritis.
   - *Reentrancy* (meskipun sudah menggunakan `safeTransfer`, tetap perlu diperiksa).
   - *Integer overflow/underflow* (walaupun Solidity 0.8+ sudah aman, casting tetap perlu diperhatikan).
   - *Deadline* dan *slippage protection* yang mungkin tidak diimplementasikan dengan benar.
   - *Compatibility* dengan token aneh (Weird ERC20) selain fee‑on‑transfer.
3. **Mencatat semua temuan** – baik dari fuzzing maupun manual review.
4. **Menyusun laporan audit** – menggabungkan hasil otomatis dan manual.

---

## Kesimpulan

- **Fuzzing adalah alat yang sangat kuat**, tetapi tidak sempurna.
- **Manual review tetap menjadi tulang punggung security audit** – karena manusia memahami konteks bisnis dan niat di balik kode.
- Kombinasi keduanya memberikan hasil terbaik: fuzzing untuk menemukan pelanggaran invariant dan edge case, manual review untuk memastikan tidak ada kerentanan logika yang terlewat.

> *"Our manual review is never replaced and is an invaluable part of the process."*