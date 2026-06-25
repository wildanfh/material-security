# Ringkasan Materi: Manual Review – ITSwapPool.sol

## Pendahuluan

Setelah menyelesaikan review `IPoolFactory.sol` (interface sederhana), kita melanjutkan ke file berikutnya dalam urutan kompleksitas: **`ITSwapPool.sol`**. File ini juga merupakan interface yang sederhana, terhubung dengan kontrak TSwap.

---

## File: `ITSwapPool.sol`

```solidity
// SPDX-License-Identifier: AGPL-3.0-only
pragma solidity 0.8.20;

interface ITSwapPool {
    function getPriceOfOnePoolTokenInWeth() external view returns (uint256);
}
```

**Analisis:**
- Interface ini hanya memiliki **satu fungsi**.
- Fungsi `getPriceOfOnePoolTokenInWeth` mengembalikan harga 1 pool token dalam WETH.
- Dari konteks, ini adalah interface untuk kontrak TSwapPool yang sudah kita audit sebelumnya.

---

## Verifikasi dengan Implementasi TSwapPool

Untuk memastikan interface sesuai dengan implementasi, kita periksa fungsi yang sama di `TSwapPool.sol`:

```solidity
function getPriceOfOnePoolTokenInWeth() external view returns (uint256) {
    return getOutputAmountBasedOnInput(
        1e18, 
        i_poolToken.balanceOf(address(this)), 
        i_wethToken.balanceOf(address(this))
    );
}
```

**Hasil verifikasi:**
- ✅ Tidak ada parameter – sesuai dengan interface.
- ✅ Return type `uint256` – sesuai.
- ✅ Fungsi menggunakan `getOutputAmountBasedOnInput` untuk menghitung harga berdasarkan reserve pool.

---

## Pertanyaan untuk Tim ThunderLoan

Meskipun interface terlihat benar, muncul pertanyaan kontekstual:

```solidity
// @Audit-Question: Why are we only using the price of a pool token in weth?
```

**Mengapa pertanyaan ini penting?**
- ThunderLoan hanya menggunakan harga `poolToken → WETH` dari TSwap.
- Apakah ini cukup untuk semua token yang didukung? Bagaimana dengan token yang tidak memiliki pool langsung dengan WETH?
- Keterbatasan ini mungkin disengaja, tetapi auditor harus memahami alasan desainnya.

---

## Kesimpulan

- **ITSwapPool.sol** adalah interface yang valid dan sesuai dengan implementasi TSwapPool.
- Tidak ada masalah teknis yang ditemukan.
- Pertanyaan kontekstual tentang **mengapa hanya harga poolToken/WETH yang digunakan** perlu diajukan ke tim protokol untuk memahami batasan desain.

![itswappool1](/security-section-6/16-itswappool/itswappool1.png)