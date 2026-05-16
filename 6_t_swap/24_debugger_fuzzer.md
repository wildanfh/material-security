# Debugging the Fuzzer – Mengatasi Error dalam Stateful Fuzzing TSwap

## Pendahuluan

Setelah menulis `Invariant.t.sol` dan `Handler.t.sol`, saat pertama kali menjalankan fuzz test, hampir pasti akan gagal. Ini adalah hal yang normal dalam pengembangan fuzzing suite. Materi ini memandu proses debugging error yang sengaja ditanamkan dalam kode.

> **Setup:** Tambahkan `seed = "0x1"` di bagian `[fuzz]` pada `foundry.toml` agar hasil run konsisten saat debugging.

---

## Menjalankan Test Pertama

```bash
forge test --mt statefulFuzz_constantProductFormulaStaysTheSameY --vvvv
```

Hasil: **Fail** – terjadi exception saat memanggil fungsi `deposit`.

### Melacak Error

Dengan flag `--vvvv`, kita dapat melihat trace lengkap. Scroll ke atas menemukan pesan error:

```
TSwapPool___MustBeMoreThanZero()
```

Ini adalah custom error dari TSwapPool yang terjadi ketika fungsi deposit dipanggil dengan argumen `0`. Penyebabnya ada di `Handler.t.sol`:

```solidity
function deposit(uint256 wethAmount) public {
    wethAmount = bound(wethAmount, 0, type(uint64).max);
    // ...
}
```

Fungsi `bound` mengizinkan nilai 0, padahal TSwapPool melarang deposit dengan jumlah 0.

---

## Perbaikan Pertama – Menggunakan Minimum Deposit

TSwapPool menyediakan fungsi `getMinimumWethDepositAmount()` yang mengembalikan `MINIMUM_WETH_LIQUIDITY`. Kita gunakan nilai ini sebagai batas bawah.

Perbaikan:

```solidity
function deposit(uint256 wethAmount) public {
    wethAmount = bound(wethAmount, pool.getMinimumWethDepositAmount(), type(uint64).max);
    // ...
}

function swapPoolTokenForWethBasedOnOutputWeth(uint256 outputWeth) public {
    outputWeth = bound(outputWeth, pool.getMinimumWethDepositAmount(), type(uint64).max);
    // ...
}
```

---

## Menjalankan Kembali – Error Baru

Setelah perbaikan, test masih gagal, tetapi dengan error berbeda. Kali ini assertion melibatkan nilai `actualDeltaY` dan `expectedDeltaY` yang aneh – keduanya bernilai 0.

### Melacak Error

Periksa kode `swapPoolTokenForWethBasedOnOutputWeth` di handler. Di bagian akhir fungsi:

```solidity
uint256 endingY = poolToken.balanceOf(address(this));
uint256 endingX = weth.balanceOf(address(this));

actualDeltaY = int256(endingY) - int256(startingY);
actualDeltaX = int256(endingX) - int256(startingX);
```

**Masalah:** `endingY` dan `endingX` mengambil saldo dari `address(this)` (kontrak test), bukan saldo pool. Yang harus dilacak adalah perubahan saldo dalam **pool**, bukan di handler.

---

## Perbaikan Kedua – Melacak Saldo Pool

Ubah assignment `startingY`, `startingX`, `endingY`, `endingX` untuk menggunakan `balanceOf(address(pool))`:

```solidity
startingY = int256(poolToken.balanceOf(address(pool)));
startingX = int256(weth.balanceOf(address(pool)));

// ... setelah swap

uint256 endingY = poolToken.balanceOf(address(pool));
uint256 endingX = weth.balanceOf(address(pool));
```

Lakukan hal yang sama di fungsi `deposit` jika belum.

---

## Hasil Akhir – Test Berhasil

Setelah semua perbaikan, test berjalan tanpa error:

```bash
forge test --mt statefulFuzz_constantProductFormulaStaysTheSameY
```

Output menunjukkan bahwa semua invariant test lulus. **Tidak ditemukan bug** pada TSwap untuk invariant ini. Ini adalah hasil yang baik – berarti protokol mempertahankan invariant dengan benar.

---

## Proses Debugging yang Iteratif

Debugging fuzzer adalah proses berulang:

1. Jalankan test dengan verbosity tinggi (`-vvvv`).
2. Identifikasi error pertama dalam trace.
3. Perbaiki penyebabnya (batasan nilai, variabel yang salah target, dll).
4. Ulangi sampai semua error teratasi.

> *"Don't be discouraged if you run into more errors, or different errors. This can be one of the hardest parts of this process."*

---

## Kode Handler Akhir (Ringkasan Perubahan Penting)

- **Batasan deposit dan swap** menggunakan `pool.getMinimumWethDepositAmount()` sebagai lower bound.
- **Tracking saldo** menggunakan `balanceOf(address(pool))` untuk `startingX`, `startingY`, `endingX`, `endingY`.
- **Ghost variables** `actualDeltaX`, `actualDeltaY`, `expectedDeltaX`, `expectedDeltaY` dihitung berdasarkan perubahan saldo pool.

Referensi kode lengkap ada di materi (Handler.t.sol final).

---

## Kesimpulan

- Fuzz test hampir selalu gagal pada percobaan pertama. Debugging adalah bagian penting.
- Gunakan fungsi bantuan dari protokol (seperti `getMinimumWethDepositAmount()`) untuk menentukan batasan nilai yang valid.
- Pastikan variabel ghost melacak perubahan pada **pool** (kontrak target), bukan pada handler atau kontrak test.
- Setelah semua error diperbaiki, test dapat lulus, yang mengindikasikan bahwa invariant terpenuhi (belum tentu berarti tidak ada bug lain, hanya invariant ini yang aman).

> *"We didn't find any bugs with this test ... let's keep looking."*