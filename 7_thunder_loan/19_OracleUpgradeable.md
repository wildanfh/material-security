# Ringkasan Materi: Manual Review – OracleUpgradeable.sol

## Pendahuluan

Setelah menyelesaikan review semua interface ThunderLoan, kita mulai memasuki kontrak yang sebenarnya (real contracts). Pertama adalah **`OracleUpgradeable.sol`** – kontrak yang menangani oracle harga dengan menggunakan TSwap sebagai sumber data.

---

## File: `OracleUpgradeable.sol`

```solidity
// SPDX-License-Identifier: AGPL-3.0-only
pragma solidity 0.8.20;

import { ITSwapPool } from "../interfaces/ITSwapPool.sol";
import { IPoolFactory } from "../interfaces/IPoolFactory.sol";
import { Initializable } from "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

contract OracleUpgradeable is Initializable {
    address private s_poolFactory;

    function __Oracle_init(address poolFactoryAddress) internal onlyInitializing {
        __Oracle_init_unchained(poolFactoryAddress);
    }

    function __Oracle_init_unchained(address poolFactoryAddress) internal onlyInitializing {
        s_poolFactory = poolFactoryAddress;
    }

    function getPriceInWeth(address token) public view returns (uint256) {
        address swapPoolOfToken = IPoolFactory(s_poolFactory).getPool(token);
        return ITSwapPool(swapPoolOfToken).getPriceOfOnePoolTokenInWeth();
    }

    function getPrice(address token) external view returns (uint256) {
        return getPriceInWeth(token);
    }

    function getPoolFactoryAddress() external view returns (address) {
        return s_poolFactory;
    }
}
```

---

## Komponen Utama

### 1. Impor yang Digunakan

| Impor | Fungsi |
|-------|--------|
| `ITSwapPool` | Interface untuk TSwapPool – digunakan untuk mendapatkan harga token. |
| `IPoolFactory` | Interface untuk PoolFactory – digunakan untuk mendapatkan alamat pool dari token. |
| `Initializable` | Library OpenZeppelin untuk kontrak upgradeable – menggantikan constructor. |

---

## 2. Initializable: Mengapa Upgradeable Contracts Tidak Bisa Punya Constructor?

**Masalah:** Dalam pola proxy, **storage disimpan di proxy contract**, sedangkan **logic disimpan di implementation contract**.

- Jika constructor di implementation contract menginisialisasi storage, perubahan tersebut tidak akan berpengaruh karena storage sebenarnya ada di proxy.
- Akibatnya, **upgradeable contracts tidak bisa menggunakan constructor** untuk inisialisasi state.

**Solusi:** `Initializable.sol` menyediakan fungsi `initializer` yang memungkinkan inisialisasi storage setelah proxy dideploy.

> Referensi: [Initializable.sol – OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/proxy/utils/Initializable.sol)

### Fungsi `__init` dan `__init_unchained`

Dalam `OracleUpgradeable.sol`, pola ini diterapkan:

```solidity
function __Oracle_init(address poolFactoryAddress) internal onlyInitializing {
    __Oracle_init_unchained(poolFactoryAddress);
}

function __Oracle_init_unchained(address poolFactoryAddress) internal onlyInitializing {
    s_poolFactory = poolFactoryAddress;
}
```

| Fungsi | Peran |
|--------|-------|
| `__Oracle_init` | Fungsi inisialisasi utama – memanggil `__Oracle_init_unchained`. |
| `__Oracle_init_unchained` | Fungsi "unchained" – melakukan inisialisasi state tanpa memicu event atau logika tambahan. |

> **Protip:** Perbedaan antara `__init` dan `__init_unchained` dijelaskan di [forum OpenZeppelin](https://forum.openzeppelin.com/t/difference-between-init-and-init-unchained/25255).

### Modifier `onlyInitializing`

Kedua fungsi di atas menggunakan modifier `onlyInitializing` yang memastikan bahwa inisialisasi **hanya terjadi sekali** (tidak bisa diulang).

---

## 3. Fungsi Oracle

### `getPriceInWeth`

```solidity
function getPriceInWeth(address token) public view returns (uint256) {
    address swapPoolOfToken = IPoolFactory(s_poolFactory).getPool(token);
    return ITSwapPool(swapPoolOfToken).getPriceOfOnePoolTokenInWeth();
}
```

**Alur:**
1. Dapatkan alamat pool dari `PoolFactory` menggunakan `getPool(token)`.
2. Panggil `getPriceOfOnePoolTokenInWeth()` di pool tersebut untuk mendapatkan harga 1 token dalam WETH.

### `getPrice`

```solidity
function getPrice(address token) external view returns (uint256) {
    return getPriceInWeth(token);
}
```

Fungsi wrapper sederhana – hanya memanggil `getPriceInWeth`.

### `getPoolFactoryAddress`

```solidity
function getPoolFactoryAddress() external view returns (address) {
    return s_poolFactory;
}
```

Getter untuk alamat PoolFactory.

---

## 4. Temuan: Parameter `tswapAddress` Kurang Konsisten

Di dalam fungsi `initialize` di `ThunderLoan.sol`, ditemukan pemanggilan:

```solidity
function initialize(address tswapAddress) external initializer {
    __Ownable_init(msg.sender);
    __UUPSUpgradeable_init();
    __Oracle_init(tswapAddress);  // ← parameter bernama `tswapAddress`
    s_feePrecision = 1e18;
    s_flashLoanFee = 3e15;
}
```

**Masalah:** Parameter `tswapAddress` tidak konsisten dengan nama parameter di `OracleUpgradeable.__Oracle_init` yang menggunakan `poolFactoryAddress`.

**Rekomendasi:** Ubah nama parameter menjadi `poolFactoryAddress` untuk konsistensi dan keterbacaan.

```solidity
// @Audit-Informational: Change parameter name to `poolFactoryAddress` for consistency with OracleUpgradeable.sol::__Oracle_init
function initialize(address poolFactoryAddress) external initializer {...}
```

**Severity:** Informational

---

## 5. Catatan Penting tentang Upgradeable Contracts

- **Storage vs Logic:** Proxy menyimpan state, implementation menyimpan logic.
- **Constructor tidak berguna:** Karena storage tidak ada di implementation, constructor tidak akan menginisialisasi state.
- **Initializer functions:** Menggantikan constructor, dipanggil setelah proxy dideploy.
- **`onlyInitializing`:** Mencegah re-initialization (hanya dipanggil sekali).

---

## Ringkasan Temuan

| ID | Temuan | Lokasi | Severity |
|----|--------|--------|----------|
| I-1 | Parameter `tswapAddress` sebaiknya `poolFactoryAddress` | `ThunderLoan.initialize` | Informational |

---

## Kesimpulan

- `OracleUpgradeable.sol` adalah kontrak sederhana yang menyediakan oracle harga menggunakan TSwap.
- Kontrak ini menggunakan pola upgradeable dengan `Initializable`.
- Temuan yang ditemukan bersifat **informational** (konsistensi penamaan parameter).
- Pemahaman tentang `Initializable` sangat penting untuk mengaudit kontrak upgradeable.