# Mitigasi Weak Randomness: Solusi Angka Acak yang Aman

## Pendahuluan

Bergantung pada data on-chain untuk menghasilkan angka acak sangat bermasalah karena sifat blockchain yang deterministik. Untuk mengatasi hal ini, kita harus menggunakan metode yang menghasilkan atau memverifikasi keacakan di luar chain (off-chain).

---

## 1. Chainlink VRF (Verifiable Random Function)

**Chainlink VRF** adalah standar industri untuk mendapatkan angka acak yang adil dan terverifikasi di Smart Contract tanpa mengorbankan keamanan.

### Cara Kerja:
1.  **Request**: Kontrak meminta angka acak ke oracle Chainlink.
2.  **Generation**: Oracle menghasilkan angka acak bersama dengan **bukti kriptografis** tentang bagaimana angka tersebut ditentukan.
3.  **Verification**: Bukti tersebut dipublikasikan dan **diverifikasi on-chain** oleh kontrak pintar khusus sebelum aplikasi dapat menggunakannya.

### Keuntungan Utama:
-   **Anti-Manipulasi**: Hasil tidak dapat dirusak oleh operator oracle, penambang (miners), pengguna, bahkan pengembang kontrak itu sendiri.
-   **Transparansi**: Siapa pun dapat memverifikasi bahwa angka tersebut benar-benar acak melalui bukti kriptografis yang tersedia di blockchain.

---

## 2. Skema Commit-Reveal (Commit-Reveal Scheme)

Jika tidak ingin menggunakan oracle eksternal, kita bisa menggunakan skema kriptografi yang melibatkan partisipasi pengguna dalam dua fase.

### Fase 1: Commit (Komitmen)
- Pengguna memilih jawaban atau angka rahasia.
- Pengguna mengirimkan **hash** dari jawaban tersebut bersama dengan **seed** acak (garam/salt).
- Kontrak menyimpan hash ini di blockchain. Karena berupa hash, jawaban asli tetap rahasia.

### Fase 2: Reveal (Pengungkapan)
- Setelah batas waktu tertentu, pengguna mengungkapkan jawaban asli dan nilai seed mereka.
- Kontrak menghitung ulang hash dari data yang baru dikirim dan membandingkannya dengan hash yang disimpan pada fase Commit.
- Jika cocok, jawaban diterima sebagai valid.

### Keuntungan:
-   Tidak bergantung pada pihak ketiga (Oracle).
-   Mencegah pemain lain melihat jawaban pemain sebelumnya sebelum semua orang memberikan komitmen.

---

## Ringkasan Perbandingan

| Fitur | Chainlink VRF | Commit-Reveal |
| :--- | :--- | :--- |
| **Sumber Keacakan** | Oracle Eksternal | Partisipasi Pengguna |
| **Keamanan** | Sangat Tinggi (Kriptografis) | Tergantung pada jumlah partisipan |
| **Kompleksitas** | Integrasi API Oracle | Logika kontrak dua fase |
| **Biaya Gas** | Membutuhkan pembayaran LINK | Dua kali transaksi (Commit & Reveal) |

---

## Kesimpulan

Untuk protokol yang melibatkan nilai finansial besar (seperti lotre PuppyRaffle atau pencetakan NFT langka), sangat disarankan untuk menggunakan **Chainlink VRF**. Namun, untuk sistem voting sederhana atau game, **Commit-Reveal** bisa menjadi alternatif yang baik tanpa ketergantungan pada pihak ketiga.

---
*Referensi:*
- *[Chainlink VRF Documentation](https://docs.chain.link/vrf)*
- *[Commit-Reveal Scheme in Solidity (Medium)](https://medium.com/coinmonks/commit-reveal-scheme-in-solidity-c06eba4091bb)*
