# Business Logic Edge Case – Fungsi `getActivePlayerIndex`

## Pendahuluan

Setelah mengidentifikasi kerentanan DoS pada `enterRaffle`, kita beralih ke fungsi `refund` yang disebutkan dalam dokumentasi:

> *"Users are allowed to get a refund of their ticket & value if they call the refund function"*

Fungsi `refund` bergantung pada `getActivePlayerIndex` untuk menemukan indeks pemain.

---

## Kode yang Diperiksa

### Fungsi `refund`

```solidity
function refund(uint256 playerIndex) public {
    address playerAddress = players[playerIndex];
    require(playerAddress == msg.sender, "PuppyRaffle: Only the player can refund");
    require(playerAddress != address(0), "PuppyRaffle: Player already refunded, or is not active");

    payable(msg.sender).sendValue(entranceFee);

    players[playerIndex] = address(0);
    emit RaffleRefunded(playerAddress);
}
```

Fungsi getActivePlayerIndex

```solidity
function getActivePlayerIndex(address player) external view returns (uint256) {
    for (uint256 i = 0; i < players.length; i++) {
        if (players[i] == player) {
            return i;
        }
    }
    return 0;
}
```

---

Masalah (Edge Case)

Pertanyaan Kritis

"Why is this returning zero?"

Array di Solidity dimulai dari indeks 0. Jika pemain yang berada di indeks 0 memanggil fungsi getActivePlayerIndex, fungsi akan mengembalikan 0 – yang ambigu:

Skenario Nilai Return Arti
Pemain aktif di indeks 0 0 "Anda aktif di indeks 0"
Pemain tidak aktif (tidak ada dalam array) 0 "Anda tidak terdaftar"

Kesimpulan: Tidak mungkin membedakan antara pemain aktif di indeks 0 dan pemain yang tidak terdaftar sama sekali.

---

Dampak

· Kebingungan pengguna – Pemain di indeks 0 akan menerima nilai balik 0, yang secara keliru dapat diartikan bahwa mereka tidak aktif.
· Potensi kesalahan fungsional – Fungsi refund membutuhkan playerIndex yang akurat. Jika pemain di indeks 0 menggunakan nilai balik 0, mereka mungkin tidak bisa melakukan refund dengan benar (atau mengira mereka tidak bisa refund).

Severity: Informational – tidak menyebabkan kehilangan dana atau kerusakan protokol secara langsung, tetapi mengurangi kualitas pengalaman pengguna dan kejelasan kode.

---

Rekomendasi Mitigasi

Ubah fungsi getActivePlayerIndex sehingga dapat membedakan antara indeks 0 dan status tidak aktif. Beberapa opsi:

1. Kembalikan type(uint256).max atau nilai spesial lainnya untuk menandakan "tidak ditemukan".
2. Ubah fungsi menjadi tryGetActivePlayerIndex yang mengembalikan (bool found, uint256 index).
3. Dokumentasikan bahwa nilai 0 berarti tidak aktif dan pastikan tidak ada pemain yang dapat memiliki indeks 0 dengan cara menambahkan offset (misal: indeks dimulai dari 1). Namun ini mengubah logika internal.

Contoh perbaikan sederhana (tanpa mengubah logika terlalu banyak):

```solidity
function getActivePlayerIndex(address player) external view returns (int256) {
    for (uint256 i = 0; i < players.length; i++) {
        if (players[i] == player) {
            return int256(i);
        }
    }
    return -1; // -1 menandakan tidak ditemukan
}
```

Dengan tipe int256, nilai negatif dapat digunakan sebagai penanda "tidak aktif".

---

Tantangan

Tugas Anda: Sebelum melanjutkan ke video berikutnya, buatlah laporan temuan sendiri untuk kerentanan ini dengan mengikuti template yang sudah dipelajari:

· Title: Root Cause + Impact
· Description: Jelaskan masalah ambigu return value 0
· Impact: Dampak pada pengguna (kebingungan, potensi error)
· Recommended Mitigation: Salah satu opsi di atas