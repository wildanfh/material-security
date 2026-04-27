# Lanjutan Recon: selectWinner & withdrawFees

## Pendahuluan

Kita telah menemukan beberapa bug besar di fungsi `selectWinner`. Sekarang, kita akan menuntaskan analisis pada fungsi tersebut dan mulai membedah fungsi `withdrawFees` untuk mencari kerentanan lainnya.

---

## 1. Analisis Akhir `selectWinner`

### Minting & Total Supply
```javascript
uint256 tokenId = totalSupply();
```
- `totalSupply()` berasal dari inheritance **ERC721**. Ia mengembalikan jumlah total token yang ada.
- Auditor perlu memastikan kapan `tokenId` ini di-increment (biasanya saat `_safeMint`).

### Penentuan Rarity (Weak Randomness Lagi)
Protokol menggunakan rumus yang sama dengan pemilihan pemenang untuk menentukan kelangkaan NFT:
```javascript
uint256 rarity = uint256(keccak256(abi.encodePacked(msg.sender, block.difficulty))) % 100;
```
- **@Audit**: Ini kembali menggunakan **Weak Randomness**. Penyerang bisa memprediksi rarity NFT yang akan mereka dapatkan.

### State Changes (Reset State)
```javascript
delete players; // Meriset array pemain
raffleStartTime = block.timestamp; // Meriset waktu mulai
previousWinner = winner; // Mencatat pemenang sebelumnya
```
- Variabel-variabel ini diupdate **sebelum** melakukan pengiriman dana. Ini adalah praktik yang baik untuk mencegah Re-entrancy pada fungsi ini sendiri.

### Interaksi Eksternal & Risiko DoS
```javascript
(bool success,) = winner.call{value: prizePool}("");
require(success, "PuppyRaffle: Failed to send prize pool to winner");
_safeMint(winner, tokenId);
```
- **Masalah Potensial**: Jika pemenang adalah Smart Contract yang fungsi `receive` atau `fallback`-nya sengaja di-revert, maka fungsi `selectWinner` tidak akan pernah bisa selesai. Ini bisa menyebabkan protokol macet (**DoS**).
- `_safeMint` dipanggil setelah pengiriman dana. Meskipun tampaknya aman, auditor harus selalu waspada terhadap efek samping dari fungsi eksternal.

---

## 2. Analisis Fungsi `withdrawFees`

Fungsi ini digunakan oleh pemilik protokol untuk menarik biaya yang terkumpul.

```javascript
function withdrawFees() external {
    require(address(this).balance == uint256(totalFees), "PuppyRaffle: There are currently players active!");
    uint256 feesToWithdraw = totalFees;
    totalFees = 0;
    (bool success,) = feeAddress.call{value: feesToWithdraw}("");
    require(success, "PuppyRaffle: Failed to withdraw fees");
}
```

### Temuan @Audit:
1.  **Logika `require` yang Bermasalah**:
    ```javascript
    require(address(this).balance == uint256(totalFees), ...);
    ```
    Pengecekan ini sangat ketat. Jika ada dana tambahan yang masuk ke kontrak (misalnya lewat `selfdestruct` atau pengiriman ETH langsung), maka `address(this).balance` akan selalu lebih besar dari `totalFees`, sehingga biaya tidak akan pernah bisa ditarik selamanya!
2.  **Ketergantungan pada Aktifitas Pemain**: Pesan error menunjukkan bahwa biaya hanya bisa ditarik jika tidak ada pemain aktif. Namun, pengecekan balance ini tidak secara akurat mencerminkan status "pemain aktif".
3.  **Risiko `feeAddress`**: Jika `feeAddress` adalah kontrak yang menolak ETH, penarikan biaya akan selalu gagal.

---

## 3. Ringkasan Poin Audit (Recon)

| Komponen | Masalah yang Teridentifikasi |
| :--- | :--- |
| **Rarity Logic** | Menggunakan Pseudo-Randomness yang bisa dimanipulasi. |
| **Recipient Contract** | Jika winner kontrak menolak ETH, fungsi `selectWinner` akan stuck. |
| **Balance Check** | Pengecekan `address(this).balance == totalFees` sangat rapuh terhadap manipulasi saldo kontrak. |
| **Fee Withdrawal** | Penarikan biaya bisa terkunci secara permanen jika saldo kontrak tidak sinkron dengan `totalFees`. |

---

## Kesimpulan

Penelusuran kita mengungkap bahwa protokol ini memiliki banyak ketergantungan yang rapuh antara saldo kontrak, variabel internal, dan perilaku alamat eksternal. Di materi berikutnya, kita akan membahas solusi terbaik untuk menangani penarikan dana dan pengelolaan biaya.
