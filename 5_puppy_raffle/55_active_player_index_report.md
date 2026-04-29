# Laporan Temuan Audit: Ketidakjelasan Indeks Pemain [L-1]

Materi ini membahas pelaporan temuan audit kategori **Low Severity** terkait ambiguitas nilai balik pada fungsi `getActivePlayerIndex`.

## Judul Temuan
**[L-1] `PuppyRaffle::getActivePlayerIndex` mengembalikan nilai 0 baik untuk pemain yang tidak ada maupun pemain di indeks 0, menyebabkan kebingungan status pendaftaran.**

## Deskripsi (Description)
Fungsi `getActivePlayerIndex` dirancang untuk mencari indeks seorang pemain dalam array `players`. Namun, terdapat masalah logika pada nilai yang dikembalikan jika pemain tidak ditemukan:

```javascript
function getActivePlayerIndex(address player) external view returns (uint256) {
    for (uint256 i = 0; i < players.length; i++) {
        if (players[i] == player) {
            return i;
        }
    }
    return 0; // @Audit: Nilai 0 juga merupakan indeks valid (pemain pertama)
}
```

Jika seorang pemain berada di indeks 0, fungsi mengembalikan 0. Jika pemain *tidak ada* dalam array, fungsi juga mengembalikan 0. Hal ini bertentangan dengan dokumentasi (natspec) yang menyatakan bahwa 0 dikembalikan jika pemain tidak aktif/tidak ada.

## Dampak (Impact)
Pemain yang berada di indeks 0 mungkin salah mengira bahwa mereka belum terdaftar karena fungsi mengembalikan 0 (yang diinterpretasikan sebagai "tidak ada"). Hal ini dapat menyebabkan pengguna mencoba mendaftar ulang dan membuang-buang gas secara sia-sia.

## Bukti Konsep (Proof of Concept)
1. Seorang pengguna mendaftar ke raffle dan menjadi pendaftar pertama (indeks 0).
2. Pengguna memanggil `getActivePlayerIndex`.
3. Fungsi mengembalikan 0.
4. Berdasarkan dokumentasi, pengguna menganggap pendaftaran mereka gagal atau tidak aktif.

## Rekomendasi (Recommendation)
Ada beberapa cara untuk memperbaiki masalah ini:

1.  **Revert**: Alih-alih mengembalikan 0, fungsi bisa melakukan `revert` dengan pesan error jika pemain tidak ditemukan.
2.  **Return int256**: Mengubah tipe data kembalian menjadi `int256` dan mengembalikan `-1` jika pemain tidak ditemukan.
3.  **Reserve Index 0**: Memastikan indeks 0 tidak digunakan untuk pemain (misalnya dengan mengisi indeks 0 dengan `address(0)` saat inisialisasi), namun ini kurang efisien secara gas.

Solusi terbaik biasanya adalah mengembalikan nilai yang jelas membedakan antara "ditemukan di awal" dan "tidak ditemukan sama sekali".
