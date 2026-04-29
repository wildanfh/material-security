# Laporan Temuan Audit: Randomness yang Lemah [H-2]

Materi ini membahas temuan kerentanan kritis terkait penggunaan variabel on-chain untuk menghasilkan angka acak, yang sangat mudah diprediksi atau dimanipulasi.

## Judul Temuan
**[H-2] *Weak Randomness* pada `PuppyRaffle::selectWinner` memungkinkan pengguna memengaruhi atau memprediksi pemenang dan kelangkaan NFT.**

## Penilaian Keparahan (Severity Assessment)
*   **Impact (Dampak): High** — Jika seseorang dapat memprediksi pemenang, mekanisme utama protokol (raffle) menjadi tidak berguna.
*   **Likelihood (Kemungkinan): High** — Penyerang memiliki insentif finansial yang besar untuk memastikan mereka menang dan mendapatkan NFT langka.

## Deskripsi (Description)
Penggunaan hashing terhadap `msg.sender`, `block.timestamp`, dan `block.difficulty` untuk menghasilkan angka acak adalah praktik yang tidak aman. Nilai-nilai ini bersifat publik dan dapat diketahui atau dimanipulasi sebelum transaksi dieksekusi.

```javascript
uint256 winnerIndex =
    uint256(keccak256(abi.encodePacked(msg.sender, block.timestamp, block.difficulty))) % players.length;
```

**Catatan Tambahan**: Hal ini juga memungkinkan pengguna untuk melakukan *front-running*. Jika seorang pemain melihat mereka bukan pemenangnya, mereka bisa mencoba memanggil `refund` sebelum transaksi `selectWinner` masuk ke blok.

## Dampak (Impact)
Setiap pengguna atau validator dapat memprediksi siapa yang akan menang. Mereka dapat memanipulasi waktu transaksi atau alamat pengirim untuk memastikan mereka menang, mengambil seluruh *prize pool*, dan mendapatkan NFT paling langka (*rarest puppy*).

## Bukti Konsep (Proof of Concept)
1.  **Validator Manipulation**: Validator dapat mengetahui nilai `block.timestamp` dan `block.difficulty` (sekarang `prevrandao` di PoS) lebih awal.
2.  **Address Mining**: Pengguna dapat mencoba berbagai alamat (`msg.sender`) sampai menemukan satu yang menghasilkan kemenangan jika digunakan pada blok tertentu.
3.  **Transaction Revert**: Pengguna dapat membuat kontrak pintar untuk memanggil `selectWinner`, lalu melakukan `revert` jika hasilnya bukan mereka yang menang.

Penggunaan nilai *on-chain* sebagai *seed* acak adalah vektor serangan yang sudah terdokumentasi dengan baik di ekosistem blockchain.

## Rekomendasi (Recommendation)
Gunakan solusi angka acak yang terbukti secara kriptografis dan tidak dapat dimanipulasi secara *on-chain*, seperti **Chainlink VRF (Verifiable Random Function)**.
