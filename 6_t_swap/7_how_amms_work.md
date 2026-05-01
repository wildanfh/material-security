# Rekap Cara Kerja AMM

## Ringkasan Order Book (Bursa Konvensional)

Sebelum memahami AMM, kita perlu tahu cara kerja bursa model **order book**:

- Bursa mencatat pesanan beli dan jual dari pengguna.
- Sistem berusaha mencocokkan pesanan beli dengan pesanan jual untuk mengeksekusi transaksi.
- Setiap pesanan dan pencocokan dapat menjadi transaksi terpisah, sehingga biaya gas bisa membengkak.
- Prosesnya bisa lambat dan mahal di lingkungan DeFi.

![How AMMs Work Recap 1](https://updraft.cyfrin.io/security-section-5/7-how-amms-work-recap/how-amms-work-recap1.png)

---

## AMM (Automated Market Maker) sebagai Solusi

AMM hadir untuk mengatasi keterbatasan order book dengan pendekatan berbeda:

- Menggunakan **Liquidity Pools** (kumpulan likuiditas) sebagai lawan transaksi.
- Harga aset ditentukan oleh **rasio** antara dua aset di dalam pool – murni berdasarkan permintaan dan penawaran.
- Pengguna bertransaksi langsung dengan pool, bukan dengan pengguna lain.

![How AMMs Work Recap 2](https://updraft.cyfrin.io/security-section-5/7-how-amms-work-recap/how-amms-work-recap2.png)

---

## Liquidity Providers (Penyedia Likuiditas)

- **Sumber likuiditas** berasal dari pengguna yang menyetor aset ke pool. Mereka disebut Liquidity Providers (LP).
- Sebagai imbalan, LP menerima **LP Token** yang mewakili persentase kontribusi mereka terhadap total pool.

![How AMMs Work Recap 3](https://updraft.cyfrin.io/security-section-5/7-how-amms-work-recap/how-amms-work-recap3.png)

---

## Peran Biaya (Fees)

- Setiap transaksi di DEX dikenakan **biaya** (misalnya 0,3%).
- Biaya ini **ditambahkan ke liquidity pool**.
- Karena LP memiliki klaim proporsional melalui LP Token, nilai klaim mereka ikut naik setiap kali ada transaksi.
- Dengan kata lain, **LP mendapat untung dari setiap transaksi** yang terjadi di pool mereka.

![How AMMs Work Recap 4](https://updraft.cyfrin.io/security-section-5/7-how-amms-work-recap/how-amms-work-recap4.png)

---

## Kesimpulan untuk Security Researcher

Memahami mekanisme AMM, order book, liquidity providers, dan fee adalah bagian dari **mendapatkan konteks** dalam security review. Seorang auditor mungkin perlu mempelajari ini untuk benar-benar memahami ruang lingkup dan tujuan protokol yang sedang diaudit.