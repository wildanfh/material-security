# Manual Review TSwap – Fungsi Final di TSwapPool

## Pendahuluan

Setelah memeriksa fungsi utama seperti `deposit`, `withdraw`, `swapExactInput`, `swapExactOutput`, dan `sellPoolTokens`, kita menyelesaikan manual review dengan meninjau fungsi-fungsi getter dan utilitas lainnya di `TSwapPool.sol`.

---

## Fungsi `totalLiquidityTokenSupply`

```solidity
/// @notice a more verbose way of getting the total supply of liquidity tokens
function totalLiquidityTokenSupply() public view returns (uint256) {
    return totalSupply();
}
```

**Temuan:** Fungsi ini hanya membungkus `totalSupply()` dari kontrak ERC20. Karena tidak dipanggil secara internal, visibilitas dapat diubah menjadi `external` untuk menghemat gas.

```solidity
// @Audit-Informational - should be marked external
```

---

## Fungsi Getter Sederhana

```solidity
function getPoolToken() external view returns (address) {
    return address(i_poolToken);
}

function getWeth() external view returns (address) {
    return address(i_wethToken);
}

function getMinimumWethDepositAmount() external pure returns (uint256) {
    return MINIMUM_WETH_LIQUIDITY;
}
```

**Penilaian:** Ketiganya sudah menggunakan visibilitas `external` yang tepat, tidak ada masalah.

---

## Fungsi Harga (Price Feeds)

```solidity
function getPriceOfOneWethInPoolTokens() external view returns (uint256) {
    return getOutputAmountBasedOnInput(
        1e18, 
        i_wethToken.balanceOf(address(this)), 
        i_poolToken.balanceOf(address(this))
    );
}

function getPriceOfOnePoolTokenInWeth() external view returns (uint256) {
    return getOutputAmountBasedOnInput(
        1e18, 
        i_poolToken.balanceOf(address(this)), 
        i_wethToken.balanceOf(address(this))
    );
}
```

**Penjelasan:**
- `getPriceOfOneWethInPoolTokens` mengembalikan berapa banyak poolToken yang diperlukan untuk mendapatkan 1 WETH (dengan asumsi input 1e18 WETH).
- `getPriceOfOnePoolTokenInWeth` mengembalikan berapa banyak WETH yang diperoleh dari 1 poolToken.

**Penilaian:** Secara matematis fungsi ini benar. Namun, perlu diingat bahwa fungsi yang mengembalikan harga berdasarkan reserve saat ini dapat menjadi sumber kerentanan jika digunakan sebagai oracle harga (misal untuk protokol lain). Topik ini akan dibahas lebih lanjut di bagian **Thunder Loan**.

> Catatan untuk auditor: Fungsi seperti ini bisa "menipu" jika tidak dipahami konteks penggunaannya.

---

## Ringkasan Temuan Final

| Temuan | Lokasi | Severity |
|--------|--------|----------|
| `totalLiquidityTokenSupply` dapat diubah ke `external` | `totalLiquidityTokenSupply` | Informational / Gas |
| Fungsi harga (price feeds) – potensi risiko jika digunakan sebagai oracle | `getPriceOfOne...` | Informational (bergantung konteks) |

---

## Kesimpulan

Semua fungsi getter dan utilitas lainnya di `TSwapPool.sol` sudah ditulis dengan baik, dengan beberapa catatan kecil untuk optimasi gas dan potensi risiko penggunaan harga sebagai oracle. Dengan ini, manual review TSwap secara keseluruhan telah selesai.

> *"Though in a later section - Thunder Loan - you'll learn that functions like these can be deceiving 😉"*
