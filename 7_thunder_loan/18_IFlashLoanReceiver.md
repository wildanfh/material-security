# Ringkasan Materi: Manual Review – IFlashLoanReceiver.sol

## Pendahuluan

Setelah menyelesaikan review ketiga interface (`IPoolFactory.sol`, `ITSwapPool.sol`, dan `IThunderLoan.sol`), kita melanjutkan ke **`IFlashLoanReceiver.sol`** – interface untuk kontrak yang menerima flash loan.

---

## File: `IFlashLoanReceiver.sol`

```solidity
// SPDX-License-Identifier: AGPL-3.0
pragma solidity 0.8.20;

import { IThunderLoan } from "./IThunderLoan.sol";

/**
 * @dev Inspired by Aave:
 * https://github.com/aave/aave-v3-core/blob/master/contracts/flashloan/interfaces/IFlashLoanReceiver.sol
 */
interface IFlashLoanReceiver {
    function executeOperation(
        address token,
        uint256 amount,
        uint256 fee,
        address initiator,
        bytes calldata params
    )
        external
        returns (bool);
}
```

---

## Temuan #1: Unused Import (Impor Tidak Digunakan)

**Masalah:** Interface `IFlashLoanReceiver.sol` mengimpor `IThunderLoan`, tetapi **tidak pernah menggunakannya** di dalam kontrak.

```solidity
import { IThunderLoan } from "./IThunderLoan.sol"; // ❌ Tidak digunakan
```

### Investigasi

- Cari di seluruh workspace (shortcut `Ctrl+Shift+F` / `Cmd+Shift+F`) untuk mengetahui di mana `IFlashLoanReceiver` digunakan.
- Ternyata impor ini **hanya digunakan di file mock testing** (`MockFlashLoanReceiver.sol`), bukan di kontrak utama.

### Mengapa Ini Praktik Buruk?

- Mengubah file kontrak live (produksi) hanya untuk kebutuhan testing adalah **praktik yang buruk**.
- Test suite seharusnya mengimpor interface sendiri jika diperlukan, bukan memodifikasi file utama.

### Perbaikan yang Disarankan

```solidity
// ❌ Buruk:
import { IFlashLoanReceiver, IThunderLoan } from "../../src/interfaces/IFlashLoanReceiver.sol";

// ✅ Baik: Impor dari lokasi yang tepat
import { IThunderLoan } from "../../src/interfaces/IThunderLoan.sol";
```

**Severity:** Informational

**Catatan audit:**

```solidity
// @Audit-Informational: Unused Import. Bad practice to edit live code for tests/mocks.
// Remove the import from `MockFlashLoanReceiver.sol` and import directly from IThunderLoan.sol
```

---

## Temuan #2: Kurangnya NatSpec

**Masalah:** Fungsi `executeOperation` tidak memiliki dokumentasi NatSpec yang menjelaskan parameter dan tujuannya.

```solidity
// @Audit-Informational: Where's the NATSPEC?
function executeOperation(
    address token,
    uint256 amount,
    uint256 fee,
    address initiator,
    bytes calldata params
) external returns (bool);
```

### Pertanyaan yang Timbul

| Parameter | Pertanyaan |
|-----------|------------|
| `token` | Token apa yang sedang dipinjam? |
| `amount` | Jumlah token yang dipinjam? |
| `fee` | Berapa biaya yang harus dibayar? |
| `initiator` | Siapa yang memulai flash loan? |
| `params` | Data tambahan apa yang dikirimkan? |

Tanpa dokumentasi, pengembang dan pengguna harus menebak atau membaca implementasi untuk memahami fungsi ini.

**Severity:** Informational

---

## Referensi: Aave Implementation

File ini terinspirasi dari implementasi Aave. Sangat disarankan untuk membaca [kode Aave](https://github.com/aave/aave-v3-core/blob/master/contracts/flashloan/interfaces/IFlashLoanReceiver.sol) untuk memahami desain dan praktik terbaik.

---

## Ringkasan Temuan

| ID | Temuan | Lokasi | Severity |
|----|--------|--------|----------|
| I-1 | Impor `IThunderLoan` tidak digunakan | `IFlashLoanReceiver.sol` | Informational |
| I-2 | Fungsi `executeOperation` tanpa NatSpec | `IFlashLoanReceiver.sol` | Informational |
| I-3 | Beberapa pertanyaan parameter tidak terjawab | `IFlashLoanReceiver.sol` | Informational |

---

## Kesimpulan

- `IFlashLoanReceiver.sol` secara struktural baik dan tidak memiliki bug kritis.
- Temuan bersifat **informational**: unused import dan kurangnya dokumentasi.
- Sebagai auditor, kita harus mencatat praktik buruk (seperti mengubah file produksi untuk testing) dan menyarankan perbaikan.
- Ini adalah contoh bagaimana **engineering best practices** juga menjadi bagian dari security review – tidak hanya mencari bug, tetapi juga meningkatkan kualitas kode.

![flashloan-receiver2](/security-section-6/18-flashloan-receiver/flashloan-receiver2.png)