# Ringkasan Materi: Lack of Slippage Protection – Write-up (High Severity)

## Pendahuluan

Temuan selanjutnya adalah **kurangnya perlindungan slippage** pada fungsi `swapExactOutput`. Kerentanan ini diklasifikasikan sebagai **High Severity** karena dapat menyebabkan pengguna menerima hasil swap yang jauh lebih buruk dari yang diharapkan akibat perubahan kondisi pasar sebelum transaksi dikonfirmasi.

---

## Kode yang Rentan

```solidity
function swapExactOutput(
    IERC20 inputToken,
    IERC20 outputToken,
    uint256 outputAmount,
    uint64 deadline
) public returns (uint256 inputAmount) {
    // ... hitung inputAmount berdasarkan outputAmount
    inputAmount = getInputAmountBasedOnOutput(outputAmount, inputReserves, outputReserves);
    _swap(inputToken, inputAmount, outputToken, outputAmount);
}
```

**Catatan audit:** Tidak ada parameter `maxInputAmount` untuk membatasi jumlah input maksimum yang bersedia dibayar pengguna.

---

## Severity

| Faktor | Penilaian | Alasan |
|--------|-----------|--------|
| **Impact** | High | Swap dapat menjadi sangat tidak menguntungkan bagi pengguna (harga bisa berubah drastis). |
| **Likelihood** | Medium/High | Kondisi pasar selalu berubah, dan MEV (Maximal Extractable Value) dapat memperparah. |

**Severity: HIGH**

---

## Deskripsi Masalah

Fungsi `swapExactOutput` tidak memiliki perlindungan slippage. Berbeda dengan `swapExactInput` yang memiliki parameter `minOutputAmount` (jumlah output minimum yang diharapkan), fungsi ini seharusnya memiliki parameter `maxInputAmount` (jumlah input maksimum yang bersedia dibayar).

Tanpa batasan ini, jika harga berubah setelah transaksi dikirim (misal karena front-running atau pergerakan pasar), pengguna dapat membayar input yang jauh lebih besar dari yang diperkirakan.

---

## Dampak (Impact)

- Pengguna dapat kehilangan dana dalam jumlah besar karena membayar input yang tidak wajar.
- Tidak ada batas atas perlindungan, sehingga transaksi dapat dieksekusi dalam kondisi pasar yang sangat tidak menguntungkan.

---

## Proof of Concept (PoC)

1. Harga 1 WETH saat ini = 1.000 USDC.
2. Pengguna mengirim transaksi `swapExactOutput`:
   - `inputToken` = USDC
   - `outputToken` = WETH
   - `outputAmount` = 1 WETH
3. Transaksi tertunda di mempool.
4. Harga pasar berubah drastis: 1 WETH menjadi **10.000 USDC** (10x lipat).
5. Transaksi dieksekusi, dan pengguna **membayar 10.000 USDC** bukannya 1.000 USDC seperti yang diharapkan.

---

## Rekomendasi Mitigasi

Tambahkan parameter `maxInputAmount` dan periksa setelah menghitung `inputAmount`:

```diff
    function swapExactOutput(
        IERC20 inputToken,
+       uint256 maxInputAmount,
        IERC20 outputToken,
        uint256 outputAmount,
        uint64 deadline
    ) public returns (uint256 inputAmount) {
        // ... hitung inputAmount
+       if (inputAmount > maxInputAmount) {
+           revert TSwapPool__InputTooHigh(inputAmount, maxInputAmount);
+       }
        _swap(inputToken, inputAmount, outputToken, outputAmount);
    }
```

Dengan ini, pengguna dapat menentukan batas maksimum input yang bersedia mereka bayar, melindungi dari perubahan harga yang tidak menguntungkan.

---

## Laporan Akhir (Ringkasan)

<details>
<summary>[H-2] Lack of slippage protection in `TSwapPool::swapExactOutput` (klik untuk lihat)</summary>

### [H-2] Lack of slippage protection in TSwapPool::swapExactOutput causes users to potentially receive way fewer tokens

**Description:** The `swapExactOutput` function does not include any sort of slippage protection. This function is similar to what is done in `TSwapPool::swapExactInput`, where the function specifies a `minOutputAmount`, the `swapExactOutput` function should specify a `maxInputAmount`.

**Impact:** If market conditions change before the transaction processes, the user could get a much worse swap.

**Proof of Concept:** [skenario seperti di atas]

**Recommended Mitigation:** Include a `maxInputAmount` parameter and check after calculating `inputAmount`.

</details>

---

## Catatan Tambahan

Ada sedikit perdebatan apakah temuan ini **High** atau **Medium**. Karena terkait dengan persetujuan pengguna dan sifat akuntabilitas, beberapa auditor mungkin menganggapnya Medium. Namun, dalam materi ini ditetapkan sebagai **High** karena potensi kerugian yang besar dan tingginya kemungkinan terjadi (terutama di lingkungan MEV).

> *"Slippage protection is a necessary consideration in nearly all token swaps."*