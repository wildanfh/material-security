# Ringkasan Materi: Reporting – Temuan Informational pada TSwap

## Pendahuluan

Setelah menyelesaikan manual review, langkah selanjutnya adalah menyusun laporan temuan. Materi ini mencakup **lima temuan tingkat informational** yang ditemukan selama audit TSwap. Auditor ditantang untuk menulis laporan sendiri terlebih dahulu sebelum membandingkan dengan contoh yang diberikan.

> Gunakan template `finding_layout.md` dari repositori kursus.

---

## Daftar Temuan Informational

### [I-1] `PoolFactory__PoolDoesNotExist` Tidak Digunakan

**Lokasi:** `PoolFactory.sol`

**Masalah:** Custom error `PoolFactory__PoolDoesNotExist` dideklarasikan tetapi tidak pernah digunakan di mana pun dalam kontrak. Ini adalah dead code.

**Rekomendasi:** Hapus error tersebut.

```diff
- error PoolFactory__PoolDoesNotExist(address tokenAddress);
```

---

### [I-2] `PoolFactory.constructor` Kurang Pemeriksaan Zero Address

**Lokasi:** `PoolFactory.sol` – constructor

**Masalah:** Parameter `wethToken` tidak diperiksa apakah bernilai `address(0)`. Jika alamat nol diteruskan, protokol akan menggunakan token WETH yang tidak valid.

**Rekomendasi:** Tambahkan pemeriksaan zero address.

```diff
constructor(address wethToken) {
+   if (wethToken == address(0)) {
+       revert();
+   }
    i_wethToken = wethToken;
}
```

---

### [I-3] `PoolFactory.createPool` Menggunakan `.name()` untuk Simbol

**Lokasi:** `PoolFactory.sol` – `createPool`

**Masalah:** Pembuatan `liquidityTokenSymbol` menggunakan `IERC20(tokenAddress).name()`, bukan `.symbol()`. Ini dapat menghasilkan simbol yang terlalu panjang atau tidak sesuai.

**Rekomendasi:** Gunakan `.symbol()` untuk simbol token likuiditas.

```diff
- string memory liquidityTokenSymbol = string.concat("ts", IERC20(tokenAddress).name());
+ string memory liquidityTokenSymbol = string.concat("ts", IERC20(tokenAddress).symbol());
```

---

### [I-4] `TSwapPool.constructor` Kurang Pemeriksaan Zero Address

**Lokasi:** `TSwapPool.sol` – constructor

**Masalah:** Parameter `poolToken` dan `wethToken` tidak diperiksa terhadap `address(0)`. Alamat nol dapat menyebabkan kerusakan protokol.

**Rekomendasi:** Tambahkan pemeriksaan zero address.

```diff
constructor(
    address poolToken,
    address wethToken,
    string memory liquidityTokenName,
    string memory liquidityTokenSymbol
) ERC20(liquidityTokenName, liquidityTokenSymbol) {
+   if (poolToken == address(0) || wethToken == address(0)) {
+       revert();
+   }
    i_wethToken = IERC20(wethToken);
    i_poolToken = IERC20(poolToken);
}
```

---

### [I-5] Event di `TSwapPool` Tidak Memiliki `indexed` Fields

**Lokasi:** `TSwapPool.sol` – event `Swap`

**Masalah:** Beberapa event tidak menggunakan `indexed` pada parameter yang penting. Parameter `indexed` memudahkan pencarian oleh alat off-chain (misal subgraph) tetapi memakan gas. Untuk event dengan lebih dari tiga parameter, sebaiknya maksimal tiga parameter di-`indexed`.

**Contoh perbaikan untuk event `Swap`:**

```diff
- event Swap(address indexed swapper, IERC20 tokenIn, uint256 amountTokenIn, IERC20 tokenOut, uint256 amountTokenOut);
+ event Swap(address indexed swapper, IERC20 indexed tokenIn, uint256 amountTokenIn, IERC20 indexed tokenOut, uint256 amountTokenOut);
```

> **Catatan:** Ada perdebatan apakah perlu meng-`indexed` semua field. Auditor dapat menyesuaikan dengan kebijakan masing-masing.

---

## Ringkasan

| ID | Temuan | Lokasi | Perbaikan |
|----|--------|--------|-----------|
| I-1 | Error tidak digunakan | `PoolFactory` | Hapus error |
| I-2 | Zero address check | `PoolFactory.constructor` | Tambahkan require |
| I-3 | Menggunakan `.name()` untuk simbol | `PoolFactory.createPool` | Ganti dengan `.symbol()` |
| I-4 | Zero address check | `TSwapPool.constructor` | Tambahkan require |
| I-5 | Event tidak di-`indexed` | `TSwapPool.events` | Tambahkan indexed pada parameter penting |

---

## Kesimpulan

Temuan informational tidak berdampak pada keamanan secara langsung, tetapi meningkatkan kualitas kode, keterbacaan, dan efisiensi gas. Auditor tetap harus melaporkannya sebagai bagian dari laporan profesional. Pada pelajaran berikutnya, akan dibahas temuan dengan severity lebih tinggi (High/Medium) yang memerlukan write-up lebih kompleks.

> *"How'd you do!? In the next lesson we're going to go through some more severe vulnerabilities with more complex write ups."*