# Ringkasan Materi: Exploit – Failure to Initialize (Studi Kasus Parity Wallet)

## Pendahuluan

Setelah melihat contoh sederhana di Remix, kita sekarang membahas **studi kasus nyata** yang terkenal di ekosistem Web3: **Parity Wallet Hack**. Insiden ini terjadi karena kegagalan inisialisasi dan mengakibatkan **lock-up dana senilai jutaan dolar**.

---

## Kronologi Kejadian

### Latar Belakang

**Parity Wallet** adalah salah satu dompet multi-sig paling populer di Ethereum pada masanya. Kontraknya menggunakan pola upgradeable dan memerlukan inisialisasi setelah deployment.

### Kerentanan

Kontrak Parity Wallet memiliki fungsi `initWallet` yang seharusnya dipanggil **setelah deployment** untuk mengatur:
- Daftar pemilik (`_owners`)
- Jumlah tanda tangan yang diperlukan (`_required`)

Namun, karena fungsi ini **tidak dipanggil segera setelah deployment** (atau karena tidak ada pemeriksaan bahwa kontrak sudah diinisialisasi), kontrak tetap dalam keadaan **tidak terinisialisasi**.

---

## Eksploitasi

### Transaksi yang Menyebabkan Bug

[**Transaksi di Etherscan**](https://etherscan.io/tx/0x05f71e1b2cb4f03e547739db15d080fd30c989eda04d37ce6264c5686e0722c9)

![failure-to-initialize-case-study2](/security-section-6/22-failure-to-initialize-case-study/failure-to-initialize-case-study2.png)

### Apa yang Dilakukan Penyerang?

1. Penyerang melihat kontrak Parity Wallet yang **belum diinisialisasi**.
2. Mereka memanggil fungsi `initWallet` dengan parameter:
   - `_owners` = alamat mereka sendiri
   - `_required` = 0 (tidak diperlukan tanda tangan dari orang lain)

![failure-to-initialize-case-study3](/security-section-6/22-failure-to-initialize-case-study/failure-to-initialize-case-study3.png)

3. Hasilnya:
   - Kontrak sekarang memiliki **satu pemilik** (penyerang) dan **tidak memerlukan tanda tangan** untuk melakukan transaksi.
   - Penyerang dapat mengontrol kontrak sepenuhnya.

### Dampak

- Penyerang **tidak mencuri dana** (ironisnya), tetapi mereka "membrick" kontrak – mengunci dana pengguna yang sudah disetor ke dalam kontrak.
- Karena kontrak sekarang memiliki pemilik yang sah (penyerang) dan tidak ada mekanisme untuk mengubahnya, dana menjadi **terkunci selamanya**.
- **Chaos** terjadi di ekosistem – pengguna kehilangan akses ke dana mereka.
- Tim Parity dan komunitas terpaksa melakukan **hard fork** atau upaya lain untuk memulihkan dana (yang akhirnya tidak sepenuhnya berhasil).

> *"The post above lives in infamy in the Web3 ecosystem."*

---

## Mengapa Ini Terjadi?

| Faktor | Penjelasan |
|--------|------------|
| **Fungsi initializer tidak dipanggil** | Tim Parity lupa atau gagal memanggil `initWallet` setelah deployment. |
| **Tidak ada pemeriksaan inisialisasi** | Fungsi `initWallet` tidak memiliki modifier `initializer` atau pemeriksaan `initialized`. |
| **Siapa pun bisa memanggil `initWallet`** | Karena kontrak belum diinisialisasi, fungsi ini masih terbuka untuk publik. |

---

## Pelajaran untuk Auditor

1. **Selalu periksa inisialisasi** – Pastikan setiap kontrak upgradeable memiliki fungsi initializer yang dipanggil segera setelah deployment (idealnya dalam script deploy).

2. **Gunakan modifier `initializer`** – OpenZeppelin menyediakan `initializer` modifier yang mencegah inisialisasi ulang dan memastikan hanya dipanggil sekali.

3. **Tambahkan pemeriksaan di fungsi kritis** – Fungsi-fungsi yang mengubah state seharusnya memiliki `require(initialized, "Not initialized")` atau modifier serupa.

4. **Audit deployment script** – Periksa apakah deploy script benar-benar memanggil initializer.

---

## Praktik Terbaik untuk Mencegah

| Metode | Penjelasan |
|--------|------------|
| **Panggil initializer di deploy script** | Hindari transaksi manual – sertakan pemanggilan `initialize` dalam script deployment yang sama. |
| **Gunakan modifier `initializer`** | OpenZeppelin `Initializable` menyediakan `initializer` modifier untuk mencegah inisialisasi ulang. |
| **Tambahkan `onlyInitialized` modifier** | Pada fungsi-fungsi kritis, tambahkan pemeriksaan bahwa kontrak sudah diinisialisasi. |
| **Dokumentasikan dengan jelas** | Beri tahu tim bahwa initializer **harus** dipanggil setelah deployment. |

---

## Ringkasan

- **Parity Wallet hack** adalah contoh klasik dari **failure to initialize**.
- Kerentanan terjadi karena fungsi `initWallet` tidak dilindungi dan tidak dipanggil setelah deployment.
- Akibatnya, penyerang dapat mengatur diri sebagai pemilik dan mengunci dana pengguna.
- Ini menunjukkan bahwa **kesalahan kecil dalam inisialisasi dapat memiliki konsekuensi besar**.