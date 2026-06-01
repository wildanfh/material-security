# Ringkasan Materi: Missing Deadline Write-up (Temuan Medium Severity)

## Pendahuluan

Setelah menyelesaikan temuan informational, langkah selanjutnya adalah menulis laporan untuk temuan **medium severity** pertama. Temuan ini berkaitan dengan parameter `deadline` di fungsi `deposit` yang tidak digunakan. Awalnya penulis mengira ini bisa menjadi high, tetapi karena fungsi yang terkena adalah `deposit` (bukan swap atau withdraw), dampaknya tidak separah jika pengguna bisa menarik likuiditas setelah deadline. Oleh karena itu, severity ditetapkan sebagai **Medium**.

---

## Kerentanan: `deadline` Tidak Digunakan di `TSwapPool::deposit`

### Root Cause

Fungsi `deposit` menerima parameter `deadline` yang seharusnya membatasi transaksi agar hanya dapat dieksekusi sebelum waktu tertentu. Namun, parameter ini **tidak pernah digunakan** dalam logika fungsi. Akibatnya, transaksi deposit dapat dieksekusi kapan saja, bahkan ketika kondisi pasar tidak menguntungkan.

### Impact (Dampak)

- Transaksi dapat terjadi pada kondisi pasar yang tidak menguntungkan (misal: harga token berubah drastis setelah deadline).
- Fungsi menjadi rentan terhadap serangan MEV (Maximal Extractable Value) – meskipun topik ini akan dijelaskan lebih lanjut nanti.
- Melanggar ekspektasi pengguna yang menyertakan `deadline` untuk proteksi.

### Severity: Medium

- **Impact:** Sedang – deposit tidak segera menyebabkan kerugian dana, tetapi dapat merugikan pengguna jika harga berubah.
- **Likelihood:** Tinggi – karena deadline tidak pernah diperiksa, semua transaksi akan terus diproses.

---

## Proses Penulisan Laporan

Template yang digunakan:

```markdown
### [S-#] TITLE (Root Cause + Impact)

**Description:**

**Impact:**

**Proof of Concept:**

**Recommended Mitigation:**
```

### 1. Judul (Title)

Mengikuti format `Root Cause + Impact`:

```markdown
### [M-1] `TSwapPool::deposit` is missing deadline check causing transactions to complete even after the deadline
```

- `[M-1]` menandakan temuan Medium pertama.
- Root Cause: missing deadline check.
- Impact: transaksi tetap berjalan meskipun deadline sudah lewat.

### 2. Deskripsi (Description)

Menjelaskan secara rinci masalah dan konsekuensinya:

```markdown
**Description:** The `deposit` function accepts a deadline parameter, which according to the documentation is "The deadline for the transaction to be completed by". However, this parameter is never used. As a consequence, operations that add liquidity to the pool might be executed at unexpected times, in market conditions where the deposit rate is unfavorable.
```

### 3. Dampak (Impact)

Ringkas tetapi jelas:

```markdown
**Impact:** Transactions could be sent when market conditions are unfavorable, even when adding a deadline parameter.
```

### 4. Proof of Concept (PoC)

Cukup menunjukkan bahwa parameter tidak digunakan (misal dengan mengutip peringatan compiler):

```markdown
**Proof of Concept:** The `deadline` parameter is unused.

Warning (5667): Unused function parameter. Remove or comment out the variable name to silence this warning.
  --> src/TSwapPool.sol:96:9:
   |
96 |         uint64 deadline
   |         ^^^^^^^^^^^^^^^
```

> Dalam kasus ini, tidak perlu membuat test terpisah karena compiler sudah menunjukkan warning.

### 5. Rekomendasi Mitigasi (Recommended Mitigation)

TSwap sudah memiliki modifier `revertIfDeadlinePassed` yang siap pakai. Rekomendasinya adalah menambahkan modifier tersebut ke fungsi `deposit`:

```diff
function deposit(
        uint256 wethToDeposit,
        uint256 minimumLiquidityTokensToMint,
        uint256 maximumPoolTokensToDeposit,
        uint64 deadline
    )
        external
+       revertIfDeadlinePassed(deadline)
        revertIfZero(wethToDeposit)
        returns (uint256 liquidityTokensToMint)
    {...}
```

---

## Hasil Laporan Akhir (Ringkasan)

<details>
<summary>Contoh Laporan [M-1] (Klik untuk lihat)</summary>

### [M-1] `TSwapPool::deposit` is missing deadline check causing transactions to complete even after the deadline

**Description:** The `deposit` function accepts a deadline parameter, which according to the documentation is "The deadline for the transaction to be completed by". However, this parameter is never used. As a consequence, operations that add liquidity to the pool might be executed at unexpected times, in market conditions where the deposit rate is unfavorable.

**Impact:** Transactions could be sent when market conditions are unfavorable, even when adding a deadline parameter.

**Proof of Concept:** The `deadline` parameter is unused.

```
Warning (5667): Unused function parameter. Remove or comment out the variable name to silence this warning.
  --> src/TSwapPool.sol:96:9:
   |
96 |         uint64 deadline
   |         ^^^^^^^^^^^^^^^
```

**Recommended Mitigation:** Consider making the following change to the function:

```diff
function deposit(
        uint256 wethToDeposit,
        uint256 minimumLiquidityTokensToMint,
        uint256 maximumPoolTokensToDeposit,
        uint64 deadline
    )
        external
+       revertIfDeadlinePassed(deadline)
        revertIfZero(wethToDeposit)
        returns (uint256 liquidityTokensToMint)
    {...}
```

</details>

---

## Kesimpulan

- Temuan `deadline` tidak digunakan adalah **medium severity** karena dampaknya terhadap deposit (tidak separah penarikan likuiditas).
- Laporan ditulis dengan template standar: judul, deskripsi, impact, PoC, mitigasi.
- PoC cukup mengutip peringatan compiler – tidak selalu perlu kode yang rumit.
- Mitigasi sederhana: tambahkan modifier `revertIfDeadlinePassed` yang sudah tersedia.

> *"Great work as always, let's keep going."*