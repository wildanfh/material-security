# Breakdown Kerentanan: Mengapa Variabel On-chain Tidak Aman?

## Pendahuluan

Setelah mengetahui bahwa keacakan di blockchain bersifat deterministik, kita perlu memahami risiko spesifik dari setiap variabel yang sering disalahgunakan untuk menghasilkan angka acak. Berikut adalah rincian bahaya dari masing-masing variabel tersebut.

---

## 1. `block.timestamp` (Waktu Blok)

Bergantung pada `block.timestamp` sangat berisiko karena validator atau penambang memiliki kendali atas waktu publikasi blok.

-   **Manipulasi Validator**: Validator yang terpilih dapat menunda pengiriman transaksi hingga waktu yang dianggap lebih menguntungkan bagi mereka.
-   **Penolakan Transaksi**: Jika hasil acak pada detik tertentu tidak menguntungkan, validator bisa memilih untuk tidak memproses blok tersebut.
-   **Slippage**: Di beberapa jaringan (seperti Arbitrum), terdapat toleransi waktu beberapa detik yang dapat dimanfaatkan oleh penyerang untuk mencari celah keacakan.

---

## 2. `block.prevrandao` (Dulu `block.difficulty`)

`block.prevrandao` diperkenalkan setelah *The Merge* untuk menggantikan `block.difficulty`. Meskipun dirancang untuk membantu pemilihan validator, nilai ini tidak aman untuk digunakan sebagai sumber keacakan Smart Contract.

Berdasarkan [**EIP-4399**](https://eips.ethereum.org/EIPS/eip-4399), terdapat dua masalah utama:
-   **Biasability (Dapat Dimanipulasi)**: Pengusul blok (*proposer*) memiliki kekuatan 1-bit untuk mempengaruhi hasil RANDAO. Mereka bisa sengaja menolak mengusulkan blok (dengan mengorbankan biaya transaksi) hanya untuk mencegah pembaruan nilai acak yang tidak menguntungkan mereka.
-   **Predictability (Dapat Diprediksi)**: Nilai historis dari oracle desentralisasi mana pun 100% dapat diprediksi. Bahkan nilai di masa depan pun dapat diprediksi dalam batas-batas tertentu oleh entitas yang memahami algoritma pemilihan validator.

---

## 3. `msg.sender` (Alamat Pemanggil)

Setiap kolom yang dikendalikan oleh pemanggil dapat dimanipulasi dengan mudah.

-   **Address Mining**: Penyerang dapat menggunakan skrip untuk "menambang" (mencari) ribuan alamat wallet yang berbeda hingga menemukan satu alamat yang jika dimasukkan ke dalam rumus hash, akan menghasilkan angka keberuntungan yang diinginkan.
-   **Kontrol Penuh**: Karena penyerang menentukan siapa pengirimnya, mereka secara otomatis mengontrol salah satu input utama dari fungsi "acak" Anda.

---

## 4. Kesimpulan: Determinisme adalah Musuh

Blockchain adalah sistem yang **deterministik**. Segala sesuatu yang Anda hitung di dalamnya:
1.  Dapat dihitung ulang oleh siapa pun.
2.  Hasilnya akan selalu sama jika inputnya sama.
3.  Inputnya (variabel blok) tersedia secara publik.

Oleh karena itu, setiap angka yang diturunkan dari data on-chain, berdasarkan definisi, adalah angka yang dapat ditentukan sebelumnya (deterministik), bukan angka acak yang sebenarnya.
