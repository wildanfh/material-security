# Manual Review TSwap – Fungsi `deposit` dan `_addLiquidityMintAndTransfer`

## Pendahuluan

Setelah memeriksa bagian awal `TSwapPool.sol`, kita melanjutkan ke fungsi `deposit`. Di sini kita menemukan beberapa temuan, mulai dari *informational* hingga *low*, serta satu potensi *medium* tergantung konteks.

---

## Fungsi `deposit`

### 1. Pemeriksaan `MINIMUM_WETH_LIQUIDITY`

```solidity
if (wethToDeposit < MINIMUM_WETH_LIQUIDITY) {
    revert TSwapPool__WethDepositAmountTooLow(MINIMUM_WETH_LIQUIDITY, wethToDeposit);
}
```

**Temuan:** Custom error mencetak nilai `MINIMUM_WETH_LIQUIDITY` yang merupakan konstanta. Informasi ini tidak berguna karena siapa pun bisa membaca konstanta tersebut.

> **Severity:** Informational – tidak mempengaruhi keamanan, hanya membuang gas dan membuat error message kurang informatif.

```solidity
// @Audit-Informational: MINIMUM_WETH_LIQUIDITY is a constant, emitting unnecessary
revert TSwapPool__WethDepositAmountTooLow(MINIMUM_WETH_LIQUIDITY, wethToDeposit);
```

---

## Fungsi `_addLiquidityMintAndTransfer`

Fungsi ini dipanggil oleh `deposit` baik untuk deposit awal maupun deposit setelah pool memiliki likuiditas.

### 2. Event Emission dengan Urutan Parameter Terbalik

```solidity
emit LiquidityAdded(msg.sender, poolTokensToDeposit, wethToDeposit);
```

Event declaration:

```solidity
event LiquidityAdded(address indexed liquidityProvider, uint256 wethDeposited, uint256 poolTokensDeposited);
```

**Masalah:** Parameter kedua dan ketiga tertukar. Seharusnya `(wethToDeposit, poolTokensToDeposit)` tetapi yang dikirim `(poolTokensToDeposit, wethToDeposit)`.

**Dampak:**
- Sistem off-chain yang mendengarkan event akan mendapatkan data yang salah (jumlah WETH dan poolToken tertukar).
- Jika ada protokol lain yang mengandalkan event ini sebagai oracle atau trigger, bisa terjadi kesalahan logika.

**Severity:** 
- **Low** – karena tidak langsung menyebabkan kehilangan dana, hanya data tidak akurat.
- Namun, tergantung konteks (misal event digunakan untuk oracle), bisa meningkat ke Medium.

```solidity
// @Audit-Low - Ordering of event emissions incorrect, should be `emit LiquidityAdded(msg.sender, wethToDeposit, poolTokensToDeposit)`
```

### 3. Fungsi `_addLiquidityMintAndTransfer` Mengikuti CEI

Fungsi ini melakukan:
1. `_mint` (effect)
2. `emit` (event)
3. `safeTransferFrom` (interaction)

Urutan **Checks‑Effects‑Interactions (CEI)** diikuti dengan benar – bagus.

```solidity
// Follows CEI
```

---

## Kembali ke `deposit` – Bagian Awal (Initial Deposit)

Pada deposit pertama (ketika `totalLiquidityTokenSupply() == 0`):

```solidity
_addLiquidityMintAndTransfer(wethToDeposit, maximumPoolTokensToDeposit, wethToDeposit);
liquidityTokensToMint = wethToDeposit;
```

### 4. Update Variabel Setelah Eksternal Call

```solidity
_addLiquidityMintAndTransfer(...);
// @Audit-Informational - Not a state variable but would be better to follow CEI
liquidityTokensToMint = wethToDeposit;
```

`liquidityTokensToMint` adalah variabel lokal (bukan state), sehingga tidak berbahaya. Namun, pola ini jika terjadi pada state variable bisa menjadi reentrancy. Catat sebagai *informational*.

---

## Bagian Deposit Selanjutnya (Saat Pool Sudah Memiliki Likuiditas)

### 5. Variabel `poolTokenReserves` Tidak Digunakan

```solidity
uint256 poolTokenReserves = i_poolToken.balanceOf(address(this));
```

Variabel ini dideklarasikan tetapi tidak pernah dipakai (sudah terdeteksi compiler). Ini adalah *dead code* yang membuang gas.

> **Severity:** Informational / Gas

```solidity
// @Audit-Informational - Line unused, can be removed to save gas
```

### 6. Pemeriksaan `maximumPoolTokensToDeposit` dan `minimumLiquidityTokensToMint`

```solidity
if (maximumPoolTokensToDeposit < poolTokensToDeposit) { revert ... }
if (liquidityTokensToMint < minimumLiquidityTokensToMint) { revert ... }
```

Sudah baik – memberikan batasan bagi pengguna (slippage protection).

---

## Ringkasan Temuan pada `deposit` dan `_addLiquidityMintAndTransfer`

| Temuan | Lokasi | Severity |
|--------|--------|----------|
| Custom error mencetak konstanta | `deposit` | Informational |
| Event `LiquidityAdded` parameter tertukar | `_addLiquidityMintAndTransfer` | Low (bisa Medium tergantung konteks) |
| Variabel `poolTokenReserves` tidak digunakan | `deposit` | Informational / Gas |
| Update variabel lokal setelah eksternal call | `deposit` (initial) | Informational |

---

## Kesimpulan

Fungsi `deposit` dan helper `_addLiquidityMintAndTransfer` secara umum mengikuti logika yang benar, dengan beberapa catatan kecil. Temuan terpenting adalah **event yang salah urutan** – meskipun low, perlu diperbaiki agar data off-chain akurat. Penggunaan CEI pada `_addLiquidityMintAndTransfer` adalah nilai plus.

> *"Leaving regular notes in the code base as you go will make report writing much easier in the future, so get into this habit!"*