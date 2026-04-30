# Apa Itu AMM dan Bagaimana Cara Kerjanya?

## Perbedaan Dasar: Order Book vs AMM

AMM (*Automated Market Maker*) sangat berbeda dengan bursa model *order book* klasik. Untuk memahaminya, kita lihat dulu cara kerja *order book*.

---

## Bursa Model Order Book

**Cara kerja:** Sederhana secara konsep – mencatat pesanan beli dan jual, lalu mencoba mencocokkannya.

![what-is-an-amm1](/security-section-5/5-what-is-an-amm/what-is-an-amm1.png)

**Masalah fatal di ekosistem blockchain: BIAYA**

- Setiap kali pengguna memasang pesanan (beli atau jual) → biaya transaksi (gas).
- Mencocokkan pesanan → biaya transaksi lagi.
- Melacak dan mengelola data pesanan juga punya biaya overhead.

**Akibatnya:** Bursa order book di Ethereum bisa menjadi **lambat dan mahal** dengan cepat.

---

## Automated Market Maker (AMM)

**Cara kerja:** Memanfaatkan **kumpulan likuiditas (liquidity pools)** dengan tujuan mempertahankan rasio antar aset yang diperdagangkan.

![what-is-an-amm2](/security-section-5/5-what-is-an-amm/what-is-an-amm2.png)

**Mekanisme:**
- Ketika pesanan ditempatkan melawan *liquidity pool*, rasio antara dua aset berubah.
- Perubahan rasio ini **menggerakkan harga** pasangan aset untuk transaksi berikutnya.

**Keuntungan di blockchain:**
- Pengguna hanya perlu bertransaksi **langsung dengan liquidity pool**.
- Hanya **satu transaksi** untuk mengeksekusi perdagangan.
- Jauh lebih hemat gas dan komputasi di lingkungan seperti Ethereum.

---

## Ringkasan Perbandingan

| Aspek | Order Book | AMM |
|-------|-----------|-----|
| Mekanisme | Mencocokkan pesanan beli/jual | Bertransaksi langsung dengan pool |
| Biaya di blockchain | Tinggi (banyak transaksi) | Rendah (satu transaksi) |
| Kecepatan | Semakin lambat seiring volume | Konsisten |
| Likuiditas | Bergantung pada market maker | Dari liquidity providers (LP) |