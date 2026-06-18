# Ringkasan Materi: Quick Recap (Arbitrase dan Flash Loan)

## Pendahuluan

Meskipun baru beberapa pelajaran, kita sudah membahas konsep-konsep kompleks. Materi ini adalah rekap singkat dari apa yang telah dipelajari tentang arbitrase dan flash loan.

---

## 1. Arbitrase

![recap1](/security-section-6/9-recap/recap1.png)

**Arbitrase** adalah tindakan koreksi pasar dengan memanfaatkan perbedaan harga aset yang sama di dua bursa berbeda.

### Cara Kerja:
- Beli aset di bursa A dengan harga lebih rendah.
- Jual aset tersebut di bursa B dengan harga lebih tinggi.
- Ambil selisihnya sebagai keuntungan.
- Proses ini juga membantu **menormalkan** harga aset di seluruh pasar.

### Hubungan dengan Flash Loan:
- Arbitrase adalah salah satu **penggunaan utama** flash loan.
- Flash loan memungkinkan siapa pun melakukan arbitrase skala besar tanpa modal sendiri.

---

## 2. Flash Loan

![recap2](/security-section-6/9-recap/recap2.png)

**Flash loan** adalah sistem di mana:
- **Liquidity providers** (penyedia likuiditas) dapat meminjamkan dana **tanpa agunan** (collateral).
- Peminjam membayar **biaya (fee)** kepada protokol.
- **Pinjaman harus dilunasi dalam transaksi yang sama** dengan saat pinjaman diambil.

### Karakteristik Kunci:
- Memungkinkan pengguna biasa memanfaatkan **daya beli yang jauh lebih besar** untuk satu transaksi.
- Jika pinjaman tidak dilunasi dalam transaksi yang sama, **seluruh transaksi dibatalkan (revert)** – seolah-olah tidak pernah terjadi.

### Ilustrasi Proses:
Diagram dalam materi menunjukkan alur:
1. Peminjam meminta flash loan.
2. Protokol mengirimkan dana.
3. Peminjam melakukan aksi (misal: arbitrase).
4. Protokol memeriksa apakah pinjaman + fee telah dikembalikan.
5. Jika ya → transaksi berhasil. Jika tidak → transaksi batal (revert).

---

## Kesimpulan

- **Arbitrase** dan **flash loan** adalah dua konsep yang saling terkait dan menjadi fondasi penting dalam DeFi.
- Flash loan adalah alat yang **demokratis** – memberikan akses setara kepada semua pengguna, tidak hanya yang memiliki modal besar.
- Pemahaman tentang kedua konsep ini sangat penting sebelum melanjutkan ke audit protokol ThunderLoan.