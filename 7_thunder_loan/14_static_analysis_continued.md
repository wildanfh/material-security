# Ringkasan Materi: Static Analysis Lanjutan – Temuan Aderyn (L-2 hingga L-6)

## Pendahuluan

Setelah membahas temuan pertama Aderyn (**L-1: Centralization Risk**), kita melanjutkan ke sisa temuan lainnya. Sebagian besar temuan ini bersifat **informational atau gas optimization**, sehingga dapat langsung disalin ke dalam laporan audit tanpa banyak perubahan. Ini adalah keuntungan besar dari penggunaan Aderyn – menghemat waktu auditor.

---

## Daftar Temuan Aderyn (L-2 hingga L-6)

| ID | Temuan | Severity | Lokasi |
|----|--------|----------|--------|
| L-2 | Missing `address(0)` checks | Low | `OracleUpgradeable.sol` line 16 |
| L-3 | `public` functions bisa diubah ke `external` | Low | Beberapa fungsi di `ThunderLoan.sol` dan `ThunderLoanUpgraded.sol` |
| L-4 | Event missing `indexed` fields | Low | Beberapa event di `AssetToken.sol`, `ThunderLoan.sol`, `ThunderLoanUpgraded.sol` |
| L-5 | PUSH0 tidak didukung semua rantai | Low | Semua file dengan `pragma solidity 0.8.20` |
| L-6 | Empty block (`_authorizeUpgrade`) | Low | `ThunderLoan.sol` dan `ThunderLoanUpgraded.sol` |

---

## [L-2] Missing Checks for `address(0)`

**Lokasi:** `src/protocol/OracleUpgradeable.sol` line 16

**Kode bermasalah:**

```solidity
function __Oracle_init_unchained(address poolFactoryAddress) internal onlyInitializing {
    // @Audit-Informational: Written in Aderyn
    s_poolFactory = poolFactoryAddress;
}
```

**Masalah:** Tidak ada pemeriksaan apakah `poolFactoryAddress != address(0)`. Jika alamat nol diteruskan, protokol akan memiliki oracle yang tidak valid.

**Tindakan:** Tambahkan komentar audit pada baris tersebut.

---

## [L-3] `public` Functions Could Be `external`

**Lokasi:** Beberapa fungsi di `ThunderLoan.sol` dan `ThunderLoanUpgraded.sol`

**Fungsi yang terkena:**

- `repay(IERC20 token, uint256 amount)`
- `getAssetFromToken(IERC20 token)`
- `isCurrentlyFlashLoaning(IERC20 token)`

**Masalah:** Fungsi `public` yang tidak dipanggil secara internal dapat diubah menjadi `external` untuk menghemat gas.

**Tindakan:** Tambahkan komentar audit `// @Audit-Gas: Could be external` pada setiap fungsi.

---

## [L-4] Event Missing `indexed` Fields

**Lokasi:** Beberapa event di `AssetToken.sol`, `ThunderLoan.sol`, dan `ThunderLoanUpgraded.sol`

**Event yang terkena:**

| File | Event |
|------|-------|
| `AssetToken.sol` | `ExchangeRateUpdated` |
| `ThunderLoan.sol` | `Deposit`, `AllowedTokenSet`, `Redeemed`, `FlashLoan` |
| `ThunderLoanUpgraded.sol` | `Deposit`, `AllowedTokenSet`, `Redeemed`, `FlashLoan` |

**Masalah:** Parameter event tidak semuanya di-`indexed`. `indexed` memudahkan pencarian oleh alat off-chain tetapi memakan gas. Idealnya, setiap event dengan ≤ 3 parameter sebaiknya semua di-`indexed`.

**Tindakan:** Tambahkan komentar audit.

---

## [L-5] PUSH0 Tidak Didukung Semua Rantai

**Lokasi:** Semua file yang menggunakan `pragma solidity 0.8.20`

**Masalah:** Solc 0.8.20 secara default menargetkan EVM Shanghai yang menggunakan opcode `PUSH0`. Beberapa L2 atau rantai non-mainnet mungkin belum mendukung opcode ini, sehingga deployment bisa gagal.

**Rekomendasi:** Gunakan `evmVersion: "london"` di `foundry.toml` atau tentukan target EVM yang sesuai dengan rantai deployment.

---

## [L-6] Empty Block (`_authorizeUpgrade`)

**Lokasi:** `ThunderLoan.sol` line 292 dan `ThunderLoanUpgraded.sol` line 287

**Kode bermasalah:**

```solidity
function _authorizeUpgrade(address newImplementation) internal override onlyOwner { }
```

**Masalah:** Fungsi override tanpa implementasi – hanya berisi `onlyOwner` tanpa logika tambahan. Ini adalah *empty block* yang dapat dihilangkan atau diisi dengan logika yang berguna (misal: event upgrade).

**Tindakan:** Tambahkan komentar audit `// @Audit: Empty block`.

---

## Kesimpulan: Manfaat Aderyn

- Aderyn secara otomatis menemukan temuan-temuan yang umum dan sudah dikenal.
- Sebagian besar temuan bersifat **informational atau gas optimization**, sehingga langsung dapat dimasukkan ke dalam laporan tanpa perlu banyak riset tambahan.
- Ini menghemat waktu auditor dan memungkinkan fokus pada manual review untuk temuan yang lebih kompleks.