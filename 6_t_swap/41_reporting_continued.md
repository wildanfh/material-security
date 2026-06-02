# Ringkasan Materi: Reporting Lanjutan (Low & High Severity Findings)

## Pendahuluan

Setelah membahas temuan informational dan medium (missing deadline), kita melanjutkan ke penulisan laporan untuk temuan **Low** dan **High** severity yang ditemukan di TSwap. Auditor disarankan menulis laporan sendiri terlebih dahulu sebelum membandingkan dengan contoh.

> **Catatan:** Temuan informational tambahan tidak dibahas secara eksplisit karena dianggap mudah setelah pengalaman sebelumnya. Namun, tetaplah menulisnya untuk latihan.

---

## [L-1] Urutan Parameter Event `LiquidityAdded` Terbalik

**Lokasi:** `TSwapPool::_addLiquidityMintAndTransfer`

**Kode bermasalah:**

```solidity
emit LiquidityAdded(msg.sender, poolTokensToDeposit, wethToDeposit);
```

**Masalah:** Event `LiquidityAdded` dideklarasikan dengan urutan `(address liquidityProvider, uint256 wethDeposited, uint256 poolTokensDeposited)`, tetapi dalam pemanggilan, parameter kedua dan ketiga tertukar.

**Dampak:** Sistem off-chain yang mengandalkan event akan mendapatkan data yang salah (weth dan poolToken tertukar), berpotensi menyebabkan malfungsi.

**Rekomendasi Mitigasi:**

```diff
- emit LiquidityAdded(msg.sender, poolTokensToDeposit, wethToDeposit);
+ emit LiquidityAdded(msg.sender, wethToDeposit, poolTokensToDeposit);
```

> **Catatan:** PoC tidak disertakan karena temuan ini cukup jelas, tetapi auditor dapat membuatnya sebagai tantangan tambahan.

---

## [H-1] Perhitungan Fee Salah di `getInputAmountBasedOnOutput`

**Lokasi:** `TSwapPool::getInputAmountBasedOnOutput`

**Kode bermasalah:**

```solidity
return ((inputReserves * outputAmount) * 10000) / ((outputReserves - outputAmount) * 997);
```

**Masalah:** Fungsi ini seharusnya menghitung input yang diperlukan untuk mendapatkan output tertentu dengan fee 0.3% (997/1000). Namun, karena menggunakan `10000` (bukan `1000`) di numerator, fee yang dihitung menjadi **90.03%** – sangat besar dan tidak sesuai standar.

**Dampak:** Protokol mengambil terlalu banyak token dari pengguna (hampir seluruh nilai swap), menyebabkan kerugian besar.

**Rekomendasi Mitigasi:**

```diff
- return ((inputReserves * outputAmount) * 10000) / ((outputReserves - outputAmount) * 997);
+ return ((inputReserves * outputAmount) * 1000) / ((outputReserves - outputAmount) * 997);
```

> **Penting:** Menulis PoC untuk temuan ini sangat dianjurkan. PoC membuktikan bahwa bug benar-benar ada dan membantu auditor menguji apakah perbaikan sudah tepat.

---

## [L-2] Nilai Return `swapExactInput` Tidak Di-assign

**Lokasi:** `TSwapPool::swapExactInput`

**Kode bermasalah:**

```solidity
function swapExactInput(...) public returns (uint256 output) {
    // ... 
    uint256 outputAmount = getOutputAmountBasedOnInput(...);
    // ...
    _swap(..., outputAmount);
    // tidak ada assignment ke `output`
}
```

**Masalah:** Fungsi mendeklarasikan named return value `output`, tetapi tidak pernah mengisinya. Akibatnya, nilai yang dikembalikan selalu `0`.

**Dampak:** Pemanggil fungsi mendapatkan informasi yang salah (selalu 0), yang dapat mengganggu integrasi dengan kontrak lain atau front-end.

**Rekomendasi Mitigasi:**

Gunakan named return value dengan mengassign `output` dari hasil perhitungan:

```diff
- uint256 outputAmount = getOutputAmountBasedOnInput(...);
+ output = getOutputAmountBasedOnInput(...);
- _swap(..., outputAmount);
+ _swap(..., output);
```

Dan sesuaikan pemeriksaan `minOutputAmount` menggunakan `output` (bukan `outputAmount`).

---

## Ringkasan Temuan yang Dibahas

| ID | Judul | Severity |
|----|-------|----------|
| L-1 | Event `LiquidityAdded` parameter out of order | Low |
| H-1 | Incorrect fee calculation in `getInputAmountBasedOnOutput` | High |
| L-2 | Default value returned by `swapExactInput` | Low |

---

## Kesimpulan

- **PoC sangat penting** – terutama untuk temuan High. Menulis PoC melatih keterampilan teknis dan membuktikan validitas temuan.
- **Event yang salah urutan** mungkin terlihat sepele, tetapi dapat merusak sistem off-chain yang bergantung pada event.
- **Kesalahan magic number** (10000 vs 1000) menunjukkan pentingnya menggunakan konstanta daripada literal.
- **Return value yang tidak di-assign** adalah kesalahan umum yang mudah dilewatkan, tetapi dapat membingungkan pengguna protokol.

> *"Writing lots of PoCs is how you'll get better at them and PoCs are how you prove there's an issue."*