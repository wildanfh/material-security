# Ringkasan Materi: `sellPoolTokens` Write-up (High Severity)

## Pendahuluan

Temuan ketiga dengan severity **High** ditemukan pada fungsi `sellPoolTokens`. Fungsi ini seharusnya memudahkan pengguna menjual token pool (i_poolToken) untuk mendapatkan WETH, tetapi terdapat kesalahan logika dalam pemanggilan fungsi swap yang menyebabkan perhitungan jumlah token menjadi salah.

---

## Kode yang Rentan

```solidity
function sellPoolTokens(uint256 poolTokenAmount) external returns (uint256 wethAmount) {
    // @Audit - Incorrect parameter being passed for swapExactOutput function - poolTokenAmount!
    return swapExactOutput(i_poolToken, i_wethToken, poolTokenAmount, uint64(block.timestamp));
}
```

Parameter `swapExactOutput` menurut definisinya:

```solidity
function swapExactOutput(
    IERC20 inputToken,   // token yang akan dibayar pengguna (input)
    IERC20 outputToken,  // token yang ingin diterima (output)
    uint256 outputAmount,// jumlah output yang diinginkan
    uint64 deadline
)
```

---

## Akar Masalah (Root Cause)

- `sellPoolTokens` menerima `poolTokenAmount` sebagai jumlah token yang **ingin dijual** oleh pengguna (input).
- Namun, fungsi tersebut memanggil `swapExactOutput` dengan `poolTokenAmount` sebagai parameter **`outputAmount`** (jumlah output yang diinginkan).
- Akibatnya, protokol mengartikan `poolTokenAmount` sebagai jumlah WETH yang ingin diperoleh, bukan jumlah token pool yang dijual.
- Fungsi yang benar untuk kasus ini seharusnya adalah `swapExactInput`, di mana pengguna menentukan jumlah input (token pool) dan menerima output WETH yang dihitung oleh protokol.

---

## Dampak (Impact)

- Pengguna akan menukar jumlah token yang salah secara fundamental.
- Protokol tidak berfungsi seperti yang diharapkan (pelanggaran terhadap invariant bisnis).
- Potensi kerugian finansial jika pengguna tidak menyadari perilaku fungsi ini.
- **Severity: High** – karena kesalahan logika bisnis yang menyebabkan disrupti parah pada fungsionalitas protokol.

---

## Proof of Concept (PoC)

*Diserahkan kepada pembaca untuk menulis. PoC harus menunjukkan bahwa memanggil `sellPoolTokens(100 ether)` (misal) tidak menghasilkan penjualan 100 token pool, tetapi justru mencoba mendapatkan 100 WETH sebagai output, yang akan menyebabkan perhitungan input menjadi sangat besar (atau revert jika reserve tidak mencukupi).*

---

## Rekomendasi Mitigasi

Ubah implementasi `sellPoolTokens` untuk menggunakan `swapExactInput` (bukan `swapExactOutput`) dan tambahkan parameter `minWethToReceive` sebagai bentuk slippage protection.

```diff
    function sellPoolTokens(
        uint256 poolTokenAmount,
+       uint256 minWethToReceive,
        ) external returns (uint256 wethAmount) {
-        return swapExactOutput(i_poolToken, i_wethToken, poolTokenAmount, uint64(block.timestamp));
+        return swapExactInput(i_poolToken, poolTokenAmount, i_wethToken, minWethToReceive, uint64(block.timestamp));
    }
```

Selain itu, sebaiknya tambahkan parameter `deadline` pada `sellPoolTokens` untuk melindungi dari transaksi yang tertunda (MEV). Namun, karena topik MEV akan dibahas kemudian, rekomendasi ini dapat ditambahkan sebagai catatan tambahan.

---

## Laporan Akhir (Ringkasan)

<details>
<summary>[H-3] `TSwapPool::sellPoolTokens` mismatches input and output tokens (klik untuk lihat)</summary>

### [H-3] `TSwapPool::sellPoolTokens` mismatches input and output tokens causing users to receive the incorrect amount of tokens

**Description:** The `sellPoolTokens` function is intended to allow users to easily sell pool tokens and receive WETH in exchange. Users indicate how many pool tokens they're willing to sell in the `poolTokenAmount` parameter. However, the function currently miscalculates the swapped amount because it calls `swapExactOutput` (which expects an output amount) instead of `swapExactInput`.

**Impact:** Users will swap the wrong amount of tokens, which is a severe disruption of protocol functionality.

**Proof of Concept:** *(diserahkan ke pembaca)*

**Recommended Mitigation:** Replace the call with `swapExactInput` and add a `minWethToReceive` parameter.

</details>

---

## Kesimpulan

- `sellPoolTokens` adalah contoh sempurna dari **business logic error** – kode secara sintaks benar, tetapi logika yang diimplementasikan salah.
- Kesalahan ini tidak akan terdeteksi oleh static analysis atau fuzzing biasa (kecuali jika invariant tentang input/output ditulis dengan sangat spesifik).
- Auditor harus memeriksa setiap pemanggilan fungsi eksternal dan memastikan parameter sesuai dengan maksud desain.
- Perbaikan mengubah fungsi dari "output tetap" menjadi "input tetap", yang sesuai dengan ekspektasi nama fungsi (`sellPoolTokens` = menjual sejumlah token pool).