# Ringkasan Materi: Manual Review – IThunderLoan.sol

## Pendahuluan

Setelah menyelesaikan review `IPoolFactory.sol` dan `ITSwapPool.sol`, kita melanjutkan ke interface berikutnya: **`IThunderLoan.sol`**. File ini adalah interface untuk kontrak utama `ThunderLoan.sol`.

---

## File: `IThunderLoan.sol`

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

interface IThunderLoan {
    function repay(address token, uint256 amount) external;
}
```

**Analisis awal:**
- Interface ini sangat sederhana – hanya **satu fungsi**: `repay`.
- Fungsi ini menerima `address token` dan `uint256 amount`.
- Ini adalah interface untuk kontrak `ThunderLoan.sol`.

---

## Temuan #1: Interface Tidak Diimplementasikan

**Masalah:** Kontrak `ThunderLoan.sol` **tidak mengimplementasikan** interface `IThunderLoan`.

```solidity
// @Audit-Informational: The IThunderLoan interface should be implemented by the ThunderLoan contract!
```

**Mengapa ini masalah?**
- Interface berfungsi sebagai **kontrak** (perjanjian) antara pengembang dan pengguna.
- Jika sebuah interface dideklarasikan tetapi tidak diimplementasikan, maka tidak ada jaminan bahwa kontrak memiliki fungsi yang dijanjikan.
- Ini adalah **kesalahan desain** (tidak mempengaruhi keamanan secara langsung, tetapi mengurangi kualitas dan kejelasan kode).

**Severity:** Informational

---

## Temuan #2: Parameter `repay` Tidak Cocok

**Masalah:** Interface `IThunderLoan` mendeklarasikan `repay(address token, uint256 amount)`, tetapi implementasi di `ThunderLoan.sol` adalah:

```solidity
function repay(IERC20 token, uint256 amount) public {...}
```

**Perbedaan:** Parameter pertama adalah `address` di interface, tetapi `IERC20` di implementasi.

**Mengapa ini terjadi?**
- Jika `ThunderLoan.sol` mengimplementasikan `IThunderLoan`, compiler akan **menangkap kesalahan ini** karena tipe parameter tidak cocok.
- Karena tidak diimplementasikan, kesalahan ini lolos dari pemeriksaan compiler.

**Dampak:**
- Secara fungsional, `address` dan `IERC20` dapat digunakan secara interchangeable di Solidity (keduanya adalah `address` di bawah hood).
- Namun, ini adalah **inkonsistensi tipe** yang dapat membingungkan pengembang dan pengguna.
- Jika ada kontrak lain yang menggunakan interface ini, mereka mungkin mengasumsikan parameter bertipe `address`, bukan `IERC20`.

**Severity:** Informational (atau Low, tergantung konteks)

```solidity
// @Audit-Low/Informational: Incorrect parameter type being passed
interface IThunderLoan {
    function repay(address token, uint256 amount) external;
}
```

---

## Ilustrasi Perbedaan

![ithunderloan1](/security-section-6/17-ithunderloan/ithunderloan1.png)

**Interface (dideklarasikan):**
```
repay(address token, uint256 amount)
```

**Implementasi (sebenarnya):**
```
repay(IERC20 token, uint256 amount)
```

Keduanya secara ABI kompatibel karena `IERC20` pada dasarnya adalah `address`. Namun, ini tetap inkonsistensi yang perlu diperbaiki.

---

## Ringkasan Temuan

| ID | Temuan | Lokasi | Severity |
|----|--------|--------|----------|
| I-1 | Interface `IThunderLoan` tidak diimplementasikan oleh `ThunderLoan.sol` | Deklarasi interface | Informational |
| I-2 | Parameter `repay` tidak cocok antara interface dan implementasi | `IThunderLoan::repay` | Informational / Low |

---

## Kesimpulan

- `IThunderLoan.sol` adalah interface yang sangat sederhana, tetapi mengandung dua temuan informasional:
  1. Interface tidak diimplementasikan oleh kontrak utama.
  2. Parameter `repay` tidak konsisten antara interface dan implementasi.
- Jika interface diimplementasikan dengan benar, kesalahan kedua akan terdeteksi oleh compiler.

![ithunderloan2](/security-section-6/17-ithunderloan/ithunderloan2.png)