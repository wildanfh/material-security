# Menjawab Pertanyaan Audit (Critical Questions & Answers)

Setelah melakukan *Reconnaissance* (penelitian awal), auditor harus menjawab pertanyaan-pertanyaan teknis untuk menentukan apakah sebuah temuan valid atau tidak. Berikut adalah ringkasan jawaban atas pertanyaan krusial pada protokol Puppy Raffle:

### 1. Logika Pembersihan Data (State Reset)
*   **Pertanyaan:** Kapan array `players` direset?
*   **Jawaban:** Di dalam fungsi `selectWinner`. Array dihapus (`delete players`), `raffleStartTime` diperbarui ke `block.timestamp`, dan pemenang sebelumnya dicatat sebelum dana dikirim.

### 2. Edge Case: Input Kosong
*   **Pertanyaan:** Apa yang terjadi jika `enterRaffle` dipanggil dengan array kosong?
*   **Jawaban:** Fungsi tetap berjalan dan tetap memancarkan event `RaffleEnter(newPlayers)`. Meskipun tidak merusak kontrak, ini adalah perilaku yang tidak efisien dan akan dimasukkan ke dalam laporan (Informational).

### 3. Masalah Indeks Nol (The Zero Index Problem)
*   **Pertanyaan:** Bagaimana jika pemain berada di Indeks 0 pada fungsi `getActivePlayerIndex`?
*   **Jawaban:** Fungsi mengembalikan `0` baik jika pemain berada di indeks pertama **maupun** jika pemain tidak ditemukan. Ini sangat menyesatkan karena pemain aktif di indeks 0 mungkin mengira mereka tidak terdaftar. **Temuan Valid (Informational/Low).**

### 4. Checks-Effects-Interactions (CEI)
*   **Pertanyaan:** Apakah fungsi `selectWinner` mengikuti pola CEI?
*   **Jawaban:** **Tidak.** Fungsi ini melakukan interaksi eksternal (mengirim ETH) setelah beberapa perubahan state, namun urutannya tidak ideal. Meskipun dalam kasus ini tidak langsung menyebabkan reentrancy yang fatal, auditor tetap akan menyarankan perbaikan (Informational).

### 5. Manipulasi dan Eksploitasi
*   **Pertanyaan:** Bisakah pengguna memaksa fungsi `selectWinner` untuk revert jika mereka tidak suka hasilnya?
*   **Jawaban:** **Ya.** Jika pemenang adalah kontrak yang sengaja menolak ETH (melalui fungsi `receive`/`fallback` yang sengaja dibuat revert), maka seluruh proses pemilihan pemenang akan gagal. Ini adalah vektor serangan **Denial of Service (DoS)**.

### 6. Kegagalan Pengiriman Dana
*   **Pertanyaan:** Apa yang terjadi jika pemenang adalah kontrak yang tidak bisa menerima ETH?
*   **Jawaban:** Pemenang tersebut tidak akan bisa menerima hadiahnya, dan karena `selectWinner` menggunakan `.call` yang di-require sukses, seluruh fungsi akan revert. Ini adalah kerentanan serius.
*   **Pertanyaan:** Bagaimana jika `feeAddress` yang tidak bisa menerima ETH?
*   **Jawaban:** Fungsi `withdrawFees` akan gagal, namun dampaknya rendah karena `owner` bisa dengan mudah mengganti `feeAddress` ke alamat lain melalui `changeFeeAddress`.

### 7. Detail Teknis NFT & Akuntansi
*   **Token ID:** Peningkatan `totalSupply` dan `tokenId` ditangani secara internal oleh library OpenZeppelin `ERC721.sol`.
*   **Kalkulasi Hadiah:** Pembagian 80% untuk pemenang dan 20% untuk biaya (fees) sudah sesuai dengan dokumentasi protokol.
*   **Accounting:** Protokol memilih menggunakan variabel internal untuk melacak biaya daripada `address(this).balance`. Ini bisa jadi pilihan desain, namun tetap perlu diawasi (seperti materi Mishandling ETH sebelumnya).

### Kesimpulan Auditor
Banyak dari jawaban di atas yang langsung menjadi poin-poin dalam laporan audit akhir. Seorang auditor yang baik tidak hanya menemukan bug, tetapi juga memahami alasan di balik setiap baris kode.
