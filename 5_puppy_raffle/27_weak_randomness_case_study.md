# Studi Kasus: Eksploitasi Meebits (Insecure Randomness)

## Pendahuluan

Pada Mei 2021, proyek NFT terkenal **Meebits** (besutan Larva Labs, tim di balik CryptoPunks) mengalami eksploitasi akibat keacakan yang tidak aman (*weak randomness*). Penyerang berhasil mendapatkan NFT langka yang kemudian terjual seharga **$700,000 (~Rp 10 Miliar)** hanya dengan memanipulasi proses pencetakan (*minting*).

---

## 1. Latar Belakang Masalah

Sistem Meebits memungkinkan pemilik CryptoPunks untuk melakukan *minting* NFT Meebit secara gratis. Sifat (traits) dari NFT yang dicetak seharusnya acak. Namun, karena keacakan ini dihasilkan secara on-chain dan dapat diprediksi, penyerang bisa melakukan "reroll" (mengulang-ulang) proses pencetakan sampai mereka mendapatkan sifat yang sangat langka.

---

## 2. Anatomi Serangan

Serangan ini terjadi karena kombinasi dari empat faktor utama:

### A. Pengungkapan Metadata (Disclosure)
Kontrak Meebits berisi hash IPFS yang mengarah ke metadata seluruh koleksi. Di dalam metadata tersebut, terdapat teks yang dengan jelas mengungkapkan urutan kelangkaan sifat (misalnya: *dissected* adalah yang paling langka). Selain itu, fungsi `tokenURI` bersifat publik, memungkinkan siapa pun melihat sifat dari `tokenId` tertentu.

### B. Keacakan yang Tidak Aman (Insecure PRNG)
Meebits menggunakan variabel on-chain untuk menghitung indeks acak:
```javascript
uint index = uint(keccak256(abi.encodePacked(nonce, msg.sender, block.difficulty, block.timestamp))) % totalSize;
```
Seperti yang telah kita pelajari, semua variabel ini (`nonce`, `msg.sender`, `block.difficulty`, `block.timestamp`) dapat diketahui atau diprediksi oleh penyerang.

### C. Mekanisme "Reroll" via Kontrak Penyerang
Penyerang membuat kontrak pintar yang melakukan hal berikut:
1.  Memanggil fungsi `mint`.
2.  Memeriksa ID atau sifat dari NFT yang baru saja dihasilkan secara instan di dalam transaksi yang sama.
3.  **Jika hasilnya tidak langka, kontrak akan memicu `revert`.**

Karena transaksi di-`revert`, pencetakan tersebut dianggap tidak pernah terjadi, dan penyerang tidak kehilangan kuota *mint* mereka. Mereka hanya perlu membayar biaya gas dan mencoba lagi di blok berikutnya.

### D. Eksekusi Bruteforce
Penyerang menjalankan kontrak ini terus-menerus selama **6 jam**, menghabiskan sekitar **$20,000 per jam** untuk biaya gas. Setelah berulang kali mencoba, mereka akhirnya berhasil mencetak **Meebit #16647** yang memiliki sifat "Visitor" (sangat langka) dan menjualnya seharga ~350 ETH ($700k).

---

## 3. Analisis Kode Kontrak Penyerang (Decompiled)

Meskipun kontrak penyerang tidak diverifikasi, dekompilasi bytecode-nya menunjukkan logika kritis:
```javascript
// Memanggil fungsi mint
v0, v1 = stor_2_0_19.mintWithPunkOrGlyph(varg0).gas(msg.gas);

// Melakukan pengecekan kelangkaan (Assertion)
// Jika tidak sesuai kriteria langka, transaksi akan REVERT/FAIL
assert(bool(uint8(map_1[v1])) == bool(1)); 
```
Baris `assert` inilah yang memungkinkan penyerang untuk "menolak" hasil acak yang buruk dan hanya menerima hasil yang menguntungkan.

---

## 4. Kesimpulan & Pelajaran

Kasus Meebits adalah contoh nyata bahwa:
-   **Keacakan on-chain sangat berbahaya** untuk aset bernilai tinggi.
-   **Pola Minting Langsung** tanpa mekanisme "Commit-Reveal" atau Oracle memungkinkan penyerang melakukan *reroll* melalui `revert`.
-   **Biaya Gas bukan penghalang** jika potensi keuntungannya jauh lebih besar daripada biaya operasional serangan.
