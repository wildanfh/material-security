# Gas Optimization: Caching Storage Variables in Loops [G-2]

Dalam audit smart contract, salah satu temuan *gas optimization* yang paling umum adalah pembacaan variabel storage secara berulang di dalam loop.

## Deskripsi Masalah

Setiap kali Anda mengakses variabel state (storage) dalam Solidity, EVM melakukan operasi `SLOAD`. Operasi ini jauh lebih mahal (minimal 100 gas) dibandingkan dengan membaca dari memory (operasi `MLOAD`, hanya 3 gas) atau stack.

Pada fungsi `enterRaffle` di PuppyRaffle, terdapat nested loop yang memeriksa duplikasi pemain:

```javascript
// @Audit-Gas: players.length dibaca berulang kali dari storage
for (uint256 i = 0; i < players.length - 1; i++) {
    for (uint256 j = i + 1; j < players.length; j++) {
        require(players[i] != players[j], "PuppyRaffle: Duplicate player");
    }
}
```

Dalam kode di atas, `players.length` dipanggil di setiap iterasi baik di loop luar maupun loop dalam. Jika jumlah pemain besar, biaya gas akan membengkak secara eksponensial.

## Rekomendasi Mitigasi (Caching)

Untuk mengoptimalkan penggunaan gas, kita harus menyalin nilai dari storage ke variabel lokal di memory (cache) sebelum loop dimulai. Dengan cara ini, EVM hanya melakukan satu kali `SLOAD` yang mahal, dan iterasi selanjutnya akan menggunakan pembacaan memory yang murah.

### Perbandingan Kode (Diff)

```diff
+ uint256 playersLength = players.length;
- for (uint256 i = 0; i < players.length - 1; i++) {
+ for (uint256 i = 0; i < playersLength - 1; i++) {
-    for (uint256 j = i + 1; j < players.length; j++) {
+    for (uint256 j = i + 1; j < playersLength; j++) {
      require(players[i] != players[j], "PuppyRaffle: Duplicate player");
    }
}
```

## Analisis Dampak

1.  **Penghematan Gas**: Mengurangi jumlah operasi `SLOAD` secara signifikan, terutama pada array yang panjang.
2.  **Efisiensi Loop**: Membuat eksekusi loop menjadi lebih ringan dan cepat.
3.  **Best Practice**: Selalu cache variabel storage (seperti `.length`) ke variabel lokal jika akan digunakan lebih dari satu kali dalam suatu fungsi, terutama di dalam loop.

> **Catatan Tambahan**: Walaupun optimasi ini penting, dalam kasus `PuppyRaffle`, pemeriksaan duplikasi dengan nested loop sendiri sudah merupakan masalah efisiensi (O(n²)). Solusi yang lebih baik biasanya melibatkan penggunaan `mapping` untuk pengecekan duplikasi yang lebih cepat, namun caching tetap merupakan teknik dasar yang harus dikuasai auditor.
