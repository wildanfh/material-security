# Manual Review TSwap – Fungsi `swapExactOutput` dan Kurangnya Slippage Protection

## Pendahuluan

Setelah meninjau `swapExactInput`, kita sekarang memeriksa `swapExactOutput` – fungsi untuk swap dengan output tetap (pengguna menentukan jumlah output yang diinginkan, dan protokol menghitung input yang diperlukan). Di sini ditemukan **kekurangan slippage protection** yang dapat menyebabkan kerugian besar bagi pengguna.

---

## Fungsi `swapExactOutput`

```solidity
function swapExactOutput(
    IERC20 inputToken,
    IERC20 outputToken,
    uint256 outputAmount,
    uint64 deadline
) public returns (uint256 inputAmount)
```

### Perbedaan dengan `swapExactInput`

| Aspek | `swapExactInput` | `swapExactOutput` |
|-------|------------------|-------------------|
| Parameter | `minOutputAmount` (slippage protection) | **Tidak ada `maxInputAmount`** |
| Jumlah parameter | 5 | 4 |
| Fungsi | Menentukan output dari input | Menentukan input dari output |

`swapExactOutput` memiliki **satu parameter lebih sedikit** – tidak ada batasan maksimum input yang bersedia dibayar pengguna.

---

## Kerentanan: Kurangnya Slippage Protection

### Skenario Serangan

1. Pengguna ingin menukar DAI dengan **10 WETH** (output tetap).
2. Pengguna mengirim transaksi `swapExactOutput` dengan `outputAmount = 10 WETH`.
3. Sebelum transaksi pengguna dikonfirmasi, **penyerang (front‑runner)** melihat transaksi di mempool dan melakukan swap besar yang mengubah harga WETH secara drastis (menaikkan input yang diperlukan).
4. Transaksi pengguna kemudian dieksekusi dengan harga yang sudah berubah.
5. Pengguna **membayar input DAI yang jauh lebih besar** dari yang diharapkan – bahkan bisa berkali‑lipat.

### Mengapa Ini Terjadi?

- `swapExactOutput` menghitung `inputAmount` berdasarkan **reserve saat itu** menggunakan `getInputAmountBasedOnOutput`.
- Jika reserve berubah (karena transaksi lain) sebelum transaksi pengguna dieksekusi, `inputAmount` akan berbeda.
- Tanpa parameter `maxInputAmount`, tidak ada batasan atas yang dilindungi.

### Dampak (Impact)

- **High** – Pengguna dapat kehilangan dana dalam jumlah besar karena membayar input yang tidak wajar.
- Contoh ekstrim: jika pool hampir kosong, input yang diperlukan untuk mendapatkan 10 WETH bisa sangat besar hingga menguras seluruh saldo pengguna.

### Likelihood

- **High** – Mekanisme front‑running dan MEV (Maximal Extractable Value) sangat umum di blockchain seperti Ethereum. Penyerang dapat dengan mudah memanfaatkan kurangnya perlindungan ini.

**Severity: HIGH**

---

## Catatan Audit

```solidity
// @Audit-High - Needs slippage protection - maxInputAmount
function swapExactOutput(...) public returns (uint256 inputAmount) {
    // ... tanpa batasan input maksimum
}
```

### Rekomendasi

Tambahkan parameter `maxInputAmount` dan periksa setelah menghitung `inputAmount`:

```solidity
function swapExactOutput(
    IERC20 inputToken,
    IERC20 outputToken,
    uint256 outputAmount,
    uint256 maxInputAmount,    // ← parameter baru
    uint64 deadline
) public returns (uint256 inputAmount) {
    // ... hitung inputAmount
    if (inputAmount > maxInputAmount) {
        revert TSwapPool__InputTooHigh(inputAmount, maxInputAmount);
    }
    // ... lanjutkan swap
}
```

Dengan ini, pengguna dapat menentukan batas maksimum input yang bersedia mereka bayar, melindungi dari perubahan harga yang tidak menguntungkan.

---

## Kaitan dengan MEV

Kerentanan ini juga merupakan vektor serangan **MEV (Maximal Extractable Value)**. Front‑runner dapat melihat transaksi `swapExactOutput` di mempool, lalu melakukan transaksi sendiri terlebih dahulu untuk mengubah harga pool, sehingga transaksi korban menjadi tidak menguntungkan. Topik MEV akan dibahas lebih lanjut.

---

## Ringkasan Temuan Baru

| Temuan | Lokasi | Severity |
|--------|--------|----------|
| Kurangnya `maxInputAmount` (slippage protection) | `swapExactOutput` | **High** |

---

## Kesimpulan

`swapExactOutput` tidak memiliki perlindungan slippage. Pengguna dapat dieksploitasi oleh front‑runner yang mengubah harga pool sebelum transaksi mereka dikonfirmasi, mengakibatkan kerugian besar. Ini adalah temuan **High** yang harus segera diperbaiki dengan menambahkan parameter `maxInputAmount` dan pemeriksaan yang sesuai.

> *"This is also a vector for an MEV attack, but we'll come back to those later."*