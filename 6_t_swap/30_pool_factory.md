# Manual Review TSwap – PoolFactory.sol

## Pendahuluan

Dengan metode `Tincho`, kita melakukan manual review baris per baris pada `PoolFactory.sol` – kontrak yang menjadi titik awal protokol TSwap. `PoolFactory` digunakan untuk membuat instance `TSwapPool` bagi setiap pasangan token ERC20 dengan WETH.

---

## Imports dan Custom Errors

### Imports

```solidity
import { TSwapPool } from "./TSwapPool.sol";
import { IERC20 } from "forge-std/interfaces/IERC20.sol";
```

Import terlihat baik. Versi IERC20 dari Forge mungkin perlu dicek, tetapi tidak ada masalah mendasar.

### Custom Errors

```solidity
error PoolFactory__PoolAlreadyExists(address tokenAddress);
error PoolFactory__PoolDoesNotExist(address tokenAddress);
```

- `PoolFactory__PoolAlreadyExists` digunakan dengan benar di `createPool` (memeriksa apakah pool sudah ada).
- **`PoolFactory__PoolDoesNotExist` TIDAK pernah digunakan** di mana pun dalam kontrak.

> **Temuan Informational:** Error yang tidak digunakan sebaiknya dihapus untuk menjaga kebersihan kode.

```solidity
// @Audit-Informational: Error is not used, can be removed.
error PoolFactory__PoolDoesNotExist(address tokenAddress);
```

---

## State Variables dan Events

### Mappings

```solidity
mapping(address token => address pool) private s_pools;
mapping(address pool => address token) private s_tokens;
```

- `s_pools`: memetakan alamat token ke alamat pool.
- `s_tokens`: memetakan alamat pool ke alamat token.
- Keduanya diperbarui dengan benar di `createPool`.

### Immutable Variable

```solidity
address private immutable i_wethToken;
```

- Diset di constructor, tidak bisa diubah – sesuai untuk token WETH yang menjadi pasangan tetap.

### Event

```solidity
event PoolCreated(address tokenAddress, address poolAddress);
```

- Hanya satu event. Tidak ada parameter `indexed`. Ini adalah pilihan desain; beberapa auditor mungkin menganggapnya kurang ideal, tetapi tidak salah.

---

## Fungsi `createPool`

Fungsi utama yang membuat pool baru.

### Pemeriksaan Duplikat

```solidity
if (s_pools[tokenAddress] != address(0)) {
    revert PoolFactory__PoolAlreadyExists(tokenAddress);
}
```

Praktik yang baik – mencegah pembuatan pool duplikat untuk token yang sama.

### Penamaan Liquidity Token

```solidity
string memory liquidityTokenName = string.concat("T-Swap ", IERC20(tokenAddress).name());
string memory liquidityTokenSymbol = string.concat("ts", IERC20(tokenAddress).name());
```

**Masalah 1:** Fungsi `.name()` pada token ERC20 bisa **revert** jika token tidak mengimplementasikannya (tidak semua token memiliki `name`). Ini dapat menghalangi pembuatan pool.

**Masalah 2:** Untuk `liquidityTokenSymbol`, seharusnya menggunakan `.symbol()` dari token, bukan `.name()`. Menggunakan `.name()` bisa menghasilkan simbol yang terlalu panjang atau tidak sesuai.

> **Temuan Informational:** Gunakan `.symbol()` untuk simbol token likuiditas, dan pertimbangkan risiko token yang tidak memiliki `name()`.

```solidity
// @Audit-Informational: should use .symbol() not .name()
string memory liquidityTokenSymbol = string.concat("ts", IERC20(tokenAddress).name());
```

### Membuat Pool Baru

```solidity
TSwapPool tPool = new TSwapPool(tokenAddress, i_wethToken, liquidityTokenName, liquidityTokenSymbol);
```

Parameter sesuai dengan konstruktor `TSwapPool` (token, weth, nama, simbol). Tidak ada masalah.

### Memperbarui Mappings dan Event

```solidity
s_pools[tokenAddress] = address(tPool);
s_tokens[address(tPool)] = tokenAddress;
emit PoolCreated(tokenAddress, address(tPool));
return address(tPool);
```

Semua baik – mapping diperbarui, event dipancarkan, alamat pool dikembalikan.

---

## Fungsi Getter (View)

```solidity
function getPool(address tokenAddress) external view returns (address) {
    return s_pools[tokenAddress];
}

function getToken(address pool) external view returns (address) {
    return s_tokens[pool];
}

function getWethToken() external view returns (address) {
    return i_wethToken;
}
```

Sederhana, aman, dan tidak ada masalah.

---

## Ringkasan Temuan pada PoolFactory.sol

| Temuan | Lokasi | Severity | Status |
|--------|--------|----------|--------|
| Error `PoolFactory__PoolDoesNotExist` tidak digunakan | Deklarasi error | Informational | Hapus untuk membersihkan kode |
| Penggunaan `.name()` untuk simbol token likuiditas | `createPool` | Informational | Sebaiknya gunakan `.symbol()` |
| Risiko token tanpa fungsi `name()` | `createPool` | Informational | Dokumentasikan batasan |

---

## Kesimpulan

`PoolFactory.sol` secara keseluruhan solid dan ditulis dengan baik. Beberapa temuan informational tidak mempengaruhi keamanan secara langsung, tetapi meningkatkan kualitas dan kejelasan kode. Tidak ada kerentanan kritis atau high severity.