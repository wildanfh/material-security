# Ringkasan Materi: Manual Review – IPoolFactory.sol (Metode Tincho)

## Pendahuluan

Setelah menjalankan static analysis (Slither & Aderyn) dan memperoleh konteks awal, kita memulai **manual review** ThunderLoan dengan menerapkan **Metode Tincho** secara lebih serius. Pendekatan ini dimulai dari kontrak yang paling kecil dan sederhana, kemudian meningkat ke kontrak yang lebih kompleks.

---

## Menjalankan Solidity Metrics

Sebelum memulai review, jalankan **Solidity Metrics** pada folder `src`:

> Klik kanan folder `src` → pilih `Solidity: Metrics` untuk menghasilkan laporan.

![recon-ipoolfactory1](/security-section-6/15-recon-ipoolfactory/recon-ipoolfactory1.png)

### Menyusun Tabel Prioritas

Hasil metrics (file dan complexity) disalin ke spreadsheet (misal Google Sheets) untuk memudahkan pelacakan dan pengurutan.

![recon-ipoolfactory2](/security-section-6/15-recon-ipoolfactory/recon-ipoolfactory2.png)

| File | Complexity | Status |
|------|------------|--------|
| IFlashLoanReceiver.sol | 1 | ✅ |
| IPoolFactory.sol | 0 | ✅ |
| ITSwapPool.sol | 1 | ⬜ |
| IThunderLoan.sol | 3 | ⬜ |
| AssetToken.sol | 31 | ⬜ |
| OracleUpgradeable.sol | 6 | ⬜ |
| ThunderLoan.sol | 137 | ⬜ |
| ThunderLoanUpgraded.sol | 128 | ⬜ |

**Tujuan:** Mulai dari file dengan kompleksitas terendah (`IPoolFactory.sol`) untuk meraih "kemenangan kecil" sebelum menghadapi kontrak besar.

---

## The Tincho Method: Mulai dari yang Kecil

### File: `IPoolFactory.sol`

```solidity
// SPDX-License-Identifier: AGPL-3.0-only
pragma solidity 0.8.20;

interface IPoolFactory {
    function getPool(address tokenAddress) external view returns (address);
}
```

**Analisis:**
- Interface ini sangat sederhana – hanya satu fungsi.
- Dari konteks sebelumnya, kita tahu bahwa `PoolFactory` merujuk pada **TSwap's PoolFactory**.
- Fungsi `getPool` digunakan untuk mendapatkan alamat pool dari alamat token.

### Catatan Audit (Notes.md)

Buat file `notes.md` untuk mencatat konteks, pertanyaan, dan ide:

```solidity
// @Audit-Explainer: This is the interface for the TSwap PoolFactory.sol contract
// @Audit-Question: Why are we using TSwap?
interface IPoolFactory {
    function getPool(address tokenAddress) external view returns (address);
}
```

- **@Audit-Explainer:** Menjelaskan bahwa ini adalah interface untuk TSwap PoolFactory – memberikan konteks bagi auditor lain atau untuk referensi di masa depan.
- **@Audit-Question:** Mengapa ThunderLoan menggunakan TSwap? Ini adalah pertanyaan yang mungkin perlu dijawab saat memahami arsitektur protokol.

### Kesimpulan untuk IPoolFactory.sol

- Tidak ada masalah keamanan yang terlihat.
- Interface sederhana dan jelas.
- File ini dapat "dicoret" dari daftar prioritas.

![recon-ipoolfactory4](/security-section-6/15-recon-ipoolfactory/recon-ipoolfactory4.png)

---

## Hal yang Perlu Diperhatikan

1. **Test coverage** ThunderLoan masih rendah (ditunjukkan di awal materi) – ini bisa menjadi temuan informational tersendiri.
2. **Metode Tincho** sangat berguna untuk mengorganisasi review: mulai dari kontrak kecil, pahami, dan tandai selesai sebelum melanjutkan.
3. **Mencatat** dengan format `@Audit-...` membantu pelacakan dan memudahkan penulisan laporan di akhir.

---

## Kesimpulan

- **IPoolFactory.sol** adalah interface sederhana tanpa celah keamanan.
- Metode Tincho membantu auditor memulai dengan cepat dan membangun momentum.
- Review selanjutnya akan melanjutkan ke kontrak dengan kompleksitas sedikit lebih tinggi.