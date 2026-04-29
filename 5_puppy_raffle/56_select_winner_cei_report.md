# Laporan Temuan Audit: Pelanggaran Best Practice CEI [I-4]

Materi ini membahas temuan kategori **Informational** pada fungsi `selectWinner` yang tidak mengikuti pola Checks-Effects-Interactions (CEI).

## Judul Temuan
**[I-4] `PuppyRaffle::selectWinner` tidak mengikuti pola CEI (Checks, Effects, Interactions).**

## Deskripsi (Description)
Meskipun tidak ditemukan celah keamanan yang dapat dieksploitasi (seperti reentrancy) dalam fungsi ini, struktur kodenya melanggar prinsip *best practice* CEI. Fungsi ini melakukan panggilan eksternal (mengirim Ether ke pemenang) sebelum menyelesaikan semua perubahan status internal (pencetakan NFT).

```javascript
// Potongan kode yang bermasalah:
(bool success,) = winner.call{value: prizePool}(""); // Interaksi eksternal
require(success, "PuppyRaffle: Failed to send prize pool to winner");

_safeMint(winner, tokenId); // Efek internal (setelah interaksi)
```

## Mengapa Informational?
Berbeda dengan fungsi `refund` di mana pelanggaran CEI menyebabkan kerentanan kritis, pada `selectWinner`, urutan ini tidak memberikan peluang bagi penyerang untuk menguras dana atau merusak status kontrak secara signifikan. Namun, tetap disarankan untuk mengikuti CEI demi konsistensi dan keamanan jangka panjang.

## Rekomendasi (Recommendation)
Selesaikan semua perubahan status internal (seperti `_safeMint`) sebelum melakukan panggilan eksternal ke alamat yang bisa berupa kontrak.

### Perbandingan Kode (Diff)
```diff
-   (bool success,) = winner.call{value: prizePool}("");
-   require(success, "PuppyRaffle: Failed to send prize pool to winner");
    _safeMint(winner, tokenId);
+   (bool success,) = winner.call{value: prizePool}("");
+   require(success, "PuppyRaffle: Failed to send prize pool to winner");
```

## Catatan Audit
Laporan temuan *informational* seringkali bersifat subjektif dan berfokus pada kualitas kode (*clean code*). Meskipun tidak memiliki dampak keamanan langsung, mendokumentasikan hal ini membantu pengembang membangun sistem yang lebih tangguh dan mudah dipelihara.
