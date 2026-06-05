# Ringkasan Materi: Invariant Break Write-up (High Severity)

## Pendahuluan

Temuan **High Severity** terakhir dalam audit TSwap adalah pelanggaran invariant protokol yang ditemukan melalui **stateful fuzzing**. Kerentanan ini terletak pada fungsi `_swap` yang memberikan token ekstra setiap 10 kali swap, melanggar rumus invariant `x * y = k` yang menjadi fondasi protokol.

---

## Kerentanan: Invariant `x * y = k` Dilanggar

### Lokasi
`TSwapPool.sol` – fungsi internal `_swap`

### Kode Bermasalah

```solidity
swap_count++;
if (swap_count >= SWAP_COUNT_MAX) {
    swap_count = 0;
    outputToken.safeTransfer(msg.sender, 1_000_000_000_000_000_000);
}
```

Setiap 10 swap, protokol mentransfer **1 token ekstra** (1e18 wei) ke pengguna sebagai insentif. Transfer ini tidak diperhitungkan dalam invariant `x * y = k`, sehingga mengubah saldo pool tanpa melalui mekanisme swap yang sah.

---

## Dampak (Impact)

- **Pelanggaran invariant inti** – rasio antara `poolToken` dan `weth` tidak lagi konstan.
- **Potensi pengurasan dana** – penyerang dapat melakukan banyak swap untuk mengumpulkan token ekstra, secara perlahan menguras saldo pool.
- **Kerusakan protokol** – harga dalam pool menjadi tidak akurat, merusak fungsi DEX.

### Severity: **High**

- Impact: High – protokol kehilangan invariant fundamental.
- Likelihood: High – setiap 10 swap, pelanggaran terjadi.

---

## Proof of Concept (PoC)

PoC ditulis sebagai unit test terpisah (bukan hanya mengandalkan output fuzzing) untuk memudahkan reproduksi.

### Langkah-langkah PoC

1. Siapkan liquidity provider yang menyetor 100e18 WETH dan 100e18 poolToken.
2. User (swapper) mendapat 100e18 poolToken dan melakukan **9 kali swap** (belum memicu insentif).
3. Catat saldo WETH pool sebelum swap ke-10 (`startingX`).
4. Lakukan swap ke-10 – sekarang `swap_count` mencapai 10, sehingga transfer ekstra aktif.
5. Hitung perubahan aktual (`actualDeltaX`) dan bandingkan dengan perubahan yang diharapkan (`expectedDeltaX = -outputWeth`).
6. Assertion gagal karena `actualDeltaX` tidak sama dengan `expectedDeltaX` (ada token ekstra yang keluar dari pool).

### Kode PoC (ditempatkan di `TSwapPool.t.sol`)

```solidity
function testInvariantBroken() public {
    // Deposit likuiditas awal
    vm.startPrank(liquidityProvider);
    weth.approve(address(pool), 100e18);
    poolToken.approve(address(pool), 100e18);
    pool.deposit(100e18, 100e18, 100e18, uint64(block.timestamp));
    vm.stopPrank();

    uint256 outputWeth = 1e17;

    vm.startPrank(user);
    poolToken.approve(address(pool), type(uint256).max);
    poolToken.mint(user, 100e18);
    // 9 swap pertama
    for (uint256 i = 0; i < 9; i++) {
        pool.swapExactOutput(poolToken, weth, outputWeth, uint64(block.timestamp));
    }

    // Catat state sebelum swap ke-10
    int256 startingX = int256(weth.balanceOf(address(pool)));
    int256 expectedDeltaX = -1 * int256(outputWeth);

    // Swap ke-10 (memicu transfer ekstra)
    pool.swapExactOutput(poolToken, weth, outputWeth, uint64(block.timestamp));
    vm.stopPrank();

    int256 endingX = int256(weth.balanceOf(address(pool)));
    int256 actualDeltaX = endingX - startingX;

    assertEq(actualDeltaX, expectedDeltaX); // Akan gagal
}
```

> **Catatan:** PoC yang baik tidak hanya menyalin urutan fuzzing, tetapi menyederhanakan menjadi unit test yang jelas.

---

## Rekomendasi Mitigasi

### Opsi 1: Hapus Mekanisme Insentif (Direkomendasikan)

```diff
-        swap_count++;
-        if (swap_count >= SWAP_COUNT_MAX) {
-            swap_count = 0;
-            outputToken.safeTransfer(msg.sender, 1_000_000_000_000_000_000);
-        }
```

### Opsi 2: Sesuaikan Invariant dengan Transfer Ekstra

Jika insentif tetap diinginkan, protokol harus memperhitungkan perubahan saldo ini dalam invariant, misalnya dengan menambahkan logika seperti pada fee (menambah `k`). Namun, ini kompleks dan rawan kesalahan. Opsi 1 lebih aman.

---

## Laporan Akhir (Ringkasan)

<details>
<summary>[H-4] In `TSwapPool::_swap` extra tokens break invariant (klik untuk lihat)</summary>

### [H-4] In `TSwapPool::_swap` the extra tokens given to users after every `swapCount` breaks the protocol invariant of `x * y = k`

**Description:** The protocol follows `x * y = k`. The `_swap` function gives an extra token every 10 swaps, changing pool balances without following the invariant.

**Impact:** A user could maliciously drain protocol funds by repeatedly swapping and collecting extra tokens. The core invariant is broken.

**Proof of Concept:** [Unit test seperti di atas]

**Recommended Mitigation:** Remove the extra incentive mechanism.

</details>

---

## Kesimpulan

- **Stateful fuzzing** berhasil menemukan pelanggaran invariant yang tidak terdeteksi oleh stateless fuzzing atau unit test biasa.
- **PoC dari hasil fuzzing** harus disederhanakan menjadi unit test yang jelas dan mudah dijalankan.
- Pelanggaran invariant adalah temuan **High** karena merusak fondasi matematis protokol.
- Mitigasi terbaik adalah menghapus mekanisme yang menyebabkan pelanggaran, bukan menambal.