# Manual Review TSwap – Fungsi `withdraw` dan Bug Fee pada `getInputAmountBasedOnOutput`

## Pendahuluan

Setelah meninjau fungsi `deposit`, kita lanjut ke fungsi `withdraw` (menghapus likuiditas) dan dua fungsi penghitung matematis: `getOutputAmountBasedOnInput` dan `getInputAmountBasedOnOutput`. Di sini ditemukan **bug kritis** pada perhitungan fee yang menyebabkan potensi kerugian >90% bagi pengguna.

---

## Fungsi `withdraw`

Fungsi ini memungkinkan penyedia likuiditas (LP) membakar token LP mereka dan menarik kembali aset (WETH dan poolToken).

### Perhitungan Jumlah Penarikan

```solidity
uint256 wethToWithdraw = (liquidityTokensToBurn * i_wethToken.balanceOf(address(this))) / totalLiquidityTokenSupply();
uint256 poolTokensToWithdraw = (liquidityTokensToBurn * i_poolToken.balanceOf(address(this))) / totalLiquidityTokenSupply();
```

Rumus ini proporsional: semakin besar `liquidityTokensToBurn` (token LP yang dibakar), semakin besar aset yang ditarik.

### Pemeriksaan Slippage (`minWethToWithdraw`, `minPoolTokensToWithdraw`)

```solidity
if (wethToWithdraw < minWethToWithdraw) { revert ... }
if (poolTokensToWithdraw < minPoolTokensToWithdraw) { revert ... }
```

Parameter `min...` melindungi dari perubahan harga yang tidak menguntungkan (MEV/slippage) – praktik yang baik.

### Pembakaran Token LP dan Transfer

```solidity
_burn(msg.sender, liquidityTokensToBurn);
emit LiquidityRemoved(msg.sender, wethToWithdraw, poolTokensToWithdraw);
i_wethToken.safeTransfer(msg.sender, wethToWithdraw);
i_poolToken.safeTransfer(msg.sender, poolTokensToWithdraw);
```

Urutan: effect (`_burn`), event, interaction (`safeTransfer`) – mengikuti CEI. Event `LiquidityRemoved` diperiksa urutan parameternya – sudah benar.

**Kesimpulan:** Fungsi `withdraw` tidak memiliki masalah keamanan.

---

## Fungsi `getOutputAmountBasedOnInput`

Fungsi ini menghitung jumlah output yang akan diterima berdasarkan input yang diberikan (digunakan untuk `swapExactInput`).

```solidity
uint256 inputAmountMinusFee = inputAmount * 997;
uint256 numerator = inputAmountMinusFee * outputReserves;
uint256 denominator = (inputReserves * 1000) + inputAmountMinusFee;
return numerator / denominator;
```

- **Magic numbers:** `997`, `1000` – sebaiknya diganti konstanta.
- **Logika fee:** Mengurangi output sebesar 0.3% (997/1000) – sesuai standar Uniswap.

> **Temuan:** Informational – gunakan konstanta untuk angka literal.

```solidity
// @Audit-Informational - Use constants instead of literals, avoid magic numbers
```

---

## Fungsi `getInputAmountBasedOnOutput` – BUG KRITIS

Fungsi ini menghitung jumlah input yang diperlukan untuk mendapatkan output tertentu (digunakan untuk `swapExactOutput`).

```solidity
return ((inputReserves * outputAmount) * 10000) / ((outputReserves - outputAmount) * 997);
```

### Perbandingan dengan Fungsi Sebelumnya

| Fungsi | Numerator | Denominator | Efek |
|--------|-----------|-------------|------|
| `getOutputAmountBasedOnInput` | `inputAmount * 997` | `(inputReserves * 1000) + ...` | Fee 0.3% |
| `getInputAmountBasedOnOutput` | `(inputReserves * outputAmount) * 10000` | `(outputReserves - outputAmount) * 997` | **Fee ~90%** |

### Perhitungan Kesalahan

Seharusnya menggunakan angka `1000` untuk denominator (seperti fungsi sebelumnya), tetapi di sini menggunakan `10000`. Akibatnya:

- `997 / 10000 = 0.0997` → hanya **9.97%** token yang sampai ke pengguna, sisanya **90.03%** menjadi fee (atau lebih tepatnya, input yang diperlukan menjadi sangat besar).
- Dampak: Setiap kali `swapExactOutput` dipanggil, pengguna akan dikenakan biaya >90% dari nilai transaksi.

### Severity

- **Impact:** High – pengguna kehilangan hampir seluruh nilai swap.
- **Likelihood:** High – fungsi `swapExactOutput` adalah fungsi utama protokol.

**Severity: HIGH**

```solidity
// @Audit-High - Erroneous fee calculation resulting in 90.03% fees
return ((inputReserves * outputAmount) * 10000) / ((outputReserves - outputAmount) * 997);
```

> **Rekomendasi:** Ganti `10000` dengan `1000` untuk mengikuti formula fee standar 0.3%.

---

## Fungsi `swapExactInput` (Sekilas)

Fungsi ini tidak memiliki NatSpec (dokumentasi). Ini adalah temuan informational.

```solidity
// @Audit-Informational - Where's the natspec?
// @Audit-Informational - functions not used internally can be marked external to save gas.
function swapExactInput(...) public returns (uint256 output) {...}
```

- **Temuan 1:** Kurang dokumentasi – mempersulit pemahaman.
- **Temuan 2:** Fungsi `public` yang tidak dipanggil internal dapat diubah menjadi `external` untuk hemat gas (seperti disarankan Aderyn).

---

## Ringkasan Temuan pada `withdraw` dan Fungsi Matematis

| Temuan | Lokasi | Severity |
|--------|--------|----------|
| Magic numbers (`997`, `1000`, `10000`) | `getOutputAmountBasedOnInput`, `getInputAmountBasedOnOutput` | Informational |
| **Kesalahan fee (10000 bukan 1000)** | `getInputAmountBasedOnOutput` | **High** |
| Fungsi `swapExactInput` tanpa NatSpec | `swapExactInput` | Informational |
| Fungsi `public` bisa diubah `external` | `swapExactInput` | Informational / Gas |
| Fungsi `withdraw` sudah baik | `withdraw` | Tidak ada temuan |

---

## Kesimpulan

- Fungsi `withdraw` aman dan mengikuti praktik baik (CEI, slippage protection).
- `getOutputAmountBasedOnInput` memiliki fee yang benar (0.3%) tetapi menggunakan magic numbers.
- **Bug kritis** ditemukan di `getInputAmountBasedOnOutput` – penggunaan `10000` alih‑alih `1000` menyebabkan fee >90%. Ini harus segera diperbaiki.
- `swapExactInput` kurang dokumentasi dan dapat dioptimalkan visibilitasnya.

> *"This bug is causing more than 90% of tokens transferred to be charged as a fee. This is a massive finding."*