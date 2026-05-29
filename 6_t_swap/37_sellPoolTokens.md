# Manual Review TSwap – Bug pada Fungsi `sellPoolTokens`

## Pendahuluan

Fungsi `sellPoolTokens` adalah *wrapper* sederhana yang memudahkan pengguna menjual token pool (i_poolToken) untuk mendapatkan WETH dengan memanggil `swapExactOutput`. Namun, ditemukan **kesalahan parameter kritis** yang merupakan *business logic error* – bukan kerentanan teknis, tetapi kesalahan penggunaan variabel.

---

## Kode Fungsi

```solidity
/**
 * @notice wrapper function to facilitate users selling pool tokens in exchange of WETH
 * @param poolTokenAmount amount of pool tokens to sell
 * @return wethAmount amount of WETH received by caller
 */
function sellPoolTokens(uint256 poolTokenAmount) external returns (uint256 wethAmount) {
    return swapExactOutput(i_poolToken, i_wethToken, poolTokenAmount, uint64(block.timestamp));
}
```

Parameter `swapExactOutput` berdasarkan definisinya:

```solidity
function swapExactOutput(
    IERC20 inputToken,   // token yang akan dibayar pengguna (input)
    IERC20 outputToken,  // token yang ingin diterima (output)
    uint256 outputAmount,// jumlah output yang diinginkan
    uint64 deadline
)
```

---

## Bug: Parameter `outputAmount` Salah

Dalam pemanggilan `swapExactOutput` di `sellPoolTokens`:

- `inputToken` = `i_poolToken` (token pool yang dijual) ✅
- `outputToken` = `i_wethToken` (WETH yang ingin diterima) ✅
- `outputAmount` = `poolTokenAmount` ❌ **SALAH!**

**Penjelasan:**  
`outputAmount` seharusnya adalah **jumlah WETH yang diinginkan** (output), bukan jumlah token pool yang dijual. Jika pengguna ingin menjual `poolTokenAmount` token pool, maka `outputAmount` harus dihitung terlebih dahulu (misalnya dengan `getOutputAmountBasedOnInput`) atau setidaknya menggunakan nilai yang sesuai – tetapi di sini `poolTokenAmount` langsung dimasukkan sebagai `outputAmount`, yang berarti pengguna menyatakan ingin menerima WETH sebanyak `poolTokenAmount` (bukan menjual token pool sebanyak itu).

Akibatnya:
- Fungsi akan menghitung `inputAmount` berdasarkan `outputAmount = poolTokenAmount` menggunakan `getInputAmountBasedOnOutput`.
- Perhitungan ini akan menghasilkan jumlah token pool yang harus dibayar pengguna sebagai **input**.
- Karena `poolTokenAmount` digunakan sebagai output, pengguna justru akan *menerima* WETH sebesar `poolTokenAmount` dan *membayar* dengan token pool dalam jumlah yang dihitung ulang – ini **terbalik** dari yang diharapkan. Fungsi ini tidak berfungsi seperti yang dimaksudkan.

> Intinya: `sellPoolTokens` seharusnya menjual sejumlah token pool (`poolTokenAmount`) untuk mendapatkan WETH, tetapi implementasi saat ini memperlakukan `poolTokenAmount` sebagai **target output WETH**, yang merupakan kesalahan logika bisnis.

---

## Dampak (Impact)

- **High** – Protokol tidak berfungsi sesuai spesifikasi. Pengguna yang memanggil `sellPoolTokens` dengan niat menjual token pool akan mendapatkan hasil yang tidak terduga (bisa jauh lebih sedikit atau lebih banyak, tergantung reserve pool).
- Potensi kerugian finansial jika pengguna tidak menyadari perilaku fungsi ini.
- Tidak ada mekanisme proteksi karena ini murni kesalahan penggunaan parameter.

---

## Rekomendasi Mitigasi

Perbaiki `sellPoolTokens` dengan menghitung `outputAmount` yang benar menggunakan `getOutputAmountBasedOnInput` atau mengubah parameter menjadi `wethAmount` yang diinginkan. Opsi perbaikan:

**Opsi 1 (sesuai nama fungsi – menjual token pool):**
```solidity
function sellPoolTokens(uint256 poolTokenAmount) external returns (uint256 wethAmount) {
    uint256 wethToReceive = getOutputAmountBasedOnInput(poolTokenAmount, i_poolToken.balanceOf(address(this)), i_wethToken.balanceOf(address(this)));
    return swapExactOutput(i_poolToken, i_wethToken, wethToReceive, uint64(block.timestamp));
}
```

**Opsi 2 (ubah parameter menjadi `wethAmount` yang diinginkan):**
```solidity
function sellPoolTokensForExactWeth(uint256 wethAmount) external returns (uint256 poolTokenAmount) {
    // hitung poolToken yang diperlukan
    poolTokenAmount = getInputAmountBasedOnOutput(wethAmount, i_poolToken.balanceOf(address(this)), i_wethToken.balanceOf(address(this)));
    swapExactOutput(i_poolToken, i_wethToken, wethAmount, uint64(block.timestamp));
}
```

Tim protokol harus mengklarifikasi maksud sebenarnya dari fungsi ini dan memperbaikinya.

---

## Ringkasan Temuan

| Temuan | Lokasi | Severity |
|--------|--------|----------|
| Parameter `outputAmount` salah – menggunakan `poolTokenAmount` seharusnya menggunakan nilai WETH yang sesuai | `sellPoolTokens` | **High** (business logic error) |

---

## Kesimpulan

Bug ini murni karena kesalahan logika bisnis – parameter yang salah diteruskan ke `swapExactOutput`. Alat static analysis sulit mendeteksi ini karena secara sintaks kode valid; hanya pemahaman konteks yang dapat menemukannya. Auditor harus memeriksa setiap pemanggilan fungsi dan memastikan parameter sesuai dengan niat desain.

> *"This is a perfect example of a bug that's a product of business logic. It's not a problem with the Solidity, simply the wrong variable was used. This is the type of thing that can be difficult for tooling to catch."*