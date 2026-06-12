# Ringkasan Materi: Phase 2 – Reconnaissance (Membaca Dokumentasi ThunderLoan)

## Pendahuluan

Setelah menyelesaikan scoping, langkah berikutnya adalah **reconnaissance** – membaca dokumentasi lebih lanjut untuk menggali konteks protokol ThunderLoan.

---

## Informasi dari README

### Latar Belakang Protokol

ThunderLoan adalah:

> *"A flash loan protocol based on [Aave](https://aave.com/) and [Compound](https://compound.finance/)."*

Jika tidak familiar dengan Aave, disarankan menonton video penjelasan singkat dari Whiteboard Crypto: [Aave Explained](https://www.youtube.com/watch?v=dTCwssZ116A). Pemahaman tentang Aave dan Compound akan sangat membantu karena ThunderLoan mengadopsi konsep serupa.

---

## Tujuan Protokol (About Section)

Dokumentasi menyebutkan dua tujuan utama ThunderLoan:

1. **Memberi pengguna cara untuk membuat flash loan.**
2. **Memberi liquidity providers cara menghasilkan uang dari modal mereka.**

---

## Definisi Istilah Penting

### Liquidity Provider (Penyedia Likuiditas)

- **Definisi:** Seseorang yang menyetor dana ke dalam protokol untuk mendapatkan bunga.
- **Pertanyaan penting:** Dari mana bunga berasal?
  - Di TSwap, bunga berasal dari biaya swap (swap fees).
  - Di ThunderLoan, liquidity providers menyetor aset ke ThunderLoan dan mendapatkan **AssetToken** sebagai imbalan. AssetToken ini bertambah nilainya seiring waktu tergantung seberapa sering orang mengambil flash loan.

### Flash Loan

- **Definisi:** Pinjaman yang hanya berlangsung **dalam satu transaksi**.
- **Cara kerja:** Pengguna dapat meminjam aset dalam jumlah berapa pun dari protokol, selama **mengembalikan jumlah yang sama + biaya dalam transaksi yang sama**. Jika gagal membayar, seluruh transaksi akan **revert** dan pinjaman dibatalkan.
- **Karakteristik:** Tidak memerlukan jaminan (collateral) karena pengembalian dipaksakan dalam satu transaksi atomik.

---

## Detail Tambahan dari ThunderLoan

- Pengguna juga harus membayar **biaya kecil (fee)** ke protokol, yang besarnya tergantung pada jumlah pinjaman.
- Untuk menghitung fee, ThunderLoan menggunakan **oracle harga on-chain TSwap** (yang sudah kita audit sebelumnya).

> **Kesimpulan sementara:** Biaya inilah yang kemungkinan menjadi sumber bunga bagi liquidity providers.

---

## Langkah Selanjutnya

- Memahami flash loan secara lebih mendalam – bagaimana mereka bekerja dalam protokol pinjam-meminjam.
- Mulai membuat **diagram protokol** untuk memvisualisasikan alur data dan interaksi antara komponen ThunderLoan.
- Mempelajari oracle dan bagaimana ThunderLoan mengintegrasikan harga dari TSwap.