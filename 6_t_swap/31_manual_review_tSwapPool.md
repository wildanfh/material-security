# Manual Review TSwap – Events, State Variables, dan Constructor

## Pendahuluan

Setelah menyelesaikan review `PoolFactory.sol`, kita lanjut ke kontrak utama yang lebih kompleks: **`TSwapPool.sol`**. Di sini kita akan memeriksa bagian awal kontrak: custom errors, state variables, modifiers, dan constructor.

---

## Penggunaan SafeERC20

```solidity
using SafeERC20 for IERC20;
```

Ini adalah praktik yang sangat baik. Library `SafeERC20` dari OpenZeppelin menyediakan fungsi `safeTransfer` dan `safeTransferFrom` yang melindungi dari berbagai perilaku aneh ERC20 (misal: tidak mengembalikan boolean, fee-on-transfer, dll).

---

## State Variables

```solidity
IERC20 private immutable i_wethToken;
IERC20 private immutable i_poolToken;
uint256 private constant MINIMUM_WETH_LIQUIDITY = 1_000_000_000;
uint256 private swap_count = 0;
uint256 private constant SWAP_COUNT_MAX = 10;
```

| Variabel | Penjelasan |
|----------|------------|
| `i_wethToken`, `i_poolToken` | Immutable – token WETH dan token ERC20 yang membentuk pool. Tidak bisa diubah setelah deploy. |
| `MINIMUM_WETH_LIQUIDITY` | Batas minimal WETH yang harus disetor saat deposit awal (1e9 wei ≈ 0.000000001 WETH). Digunakan di fungsi `deposit`. |
| `swap_count` | Penghitung jumlah swap yang telah terjadi. State variable ini **dapat berubah** – digunakan untuk memberikan insentif setiap 10 swap. |
| `SWAP_COUNT_MAX` | Konstanta = 10. Bersama `swap_count` menjadi sumber pelanggaran invariant (transfer token ekstra). |

### Kaitan dengan Invariant yang Rusak

Di fungsi `_swap`:

```solidity
swap_count++;
if (swap_count >= SWAP_COUNT_MAX) {
    swap_count = 0;
    outputToken.safeTransfer(msg.sender, 1_000_000_000_000_000_000);
}
```

Setiap 10 swap, protokol mentransfer **1 token ekstra** (1e18 wei) ke pengguna. Ini melanggar invariant `x * y = k` karena saldo pool berubah tanpa melalui mekanisme swap yang seharusnya.

---

## Events (dari Aderyn)

Beberapa event memiliki parameter yang tidak di-`indexed`. Meskipun ini bukan bug keamanan, beberapa auditor mungkin mencatatnya sebagai saran.

Contoh:

```solidity
event LiquidityAdded(address indexed liquidityProvider, uint256 wethDeposited, uint256 poolTokensDeposited);
event LiquidityRemoved(address indexed liquidityProvider, uint256 wethWithdrawn, uint256 poolTokensWithdrawn);
event Swap(address indexed swapper, IERC20 tokenIn, uint256 amountTokenIn, IERC20 tokenOut, uint256 amountTokenOut);
```

Catatan: `indexed` memudahkan pencarian off-chain tetapi memakan gas. Preferensi tergantung auditor.

---

## Modifiers

```solidity
modifier revertIfDeadlinePassed(uint64 deadline) {
    if (deadline < uint64(block.timestamp)) {
        revert TSwapPool__DeadlineHasPassed(deadline);
    }
    _;
}

modifier revertIfZero(uint256 amount) {
    if (amount == 0) {
        revert TSwapPool__MustBeMoreThanZero();
    }
    _;
}
```

Keduanya jelas dan berfungsi dengan baik. Tidak ada masalah.

---

## Constructor

```solidity
constructor(
    address poolToken,
    address wethToken,
    string memory liquidityTokenName,
    string memory liquidityTokenSymbol
) ERC20(liquidityTokenName, liquidityTokenSymbol) {
    i_wethToken = IERC20(wethToken);
    i_poolToken = IERC20(poolToken);
}
```

**Masalah:** Tidak ada pemeriksaan apakah `poolToken` atau `wethToken` adalah `address(0)`. Jika alamat nol diberikan, kontrak akan memiliki token yang tidak valid.

> **Temuan Low/Informational:** Tambahkan zero-address check.

```solidity
// @Audit - missing zero address checks
require(poolToken != address(0), "Invalid pool token");
require(wethToken != address(0), "Invalid weth token");
```

---

## Ringkasan Temuan (sejauh ini)

| Temuan | Lokasi | Severity | Status |
|--------|--------|----------|--------|
| Transfer ekstra setiap 10 swap | `_swap` | High (ditemukan fuzzing) | Melanggar invariant |
| Zero-address check di constructor | `constructor` | Low / Informational | Perlu ditambahkan |
| Event tanpa `indexed` | Beberapa event | Informational | Tergantung preferensi |

---

## Kesimpulan

Bagian awal `TSwapPool.sol` relatif bersih, dengan beberapa catatan informational dan satu kerentanan high yang sudah ditemukan melalui fuzzing (transfer ekstra). Constructor kurang zero-address check – perbaikan mudah. Penggunaan `SafeERC20` adalah nilai plus.

> *"I'm getting hungry for bugs."*