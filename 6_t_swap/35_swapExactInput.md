# Manual Review TSwap – Fungsi `swapExactInput` dan `_swap`

## Pendahuluan

Setelah menemukan bug kritis pada `getInputAmountBasedOnOutput`, kita melanjutkan ke fungsi `swapExactInput` – fungsi utama untuk swap dengan input tetap. Sayangnya fungsi ini **tidak memiliki NatSpec**, sehingga pemahaman awal mengandalkan `swapExactOutput` (kebalikannya). Dalam audit nyata, kita akan bertanya langsung ke tim protokol.

---

## Fungsi `swapExactInput`

### Parameter dan Modifier

```solidity
function swapExactInput(
    IERC20 inputToken,
    uint256 inputAmount,
    IERC20 outputToken,
    uint256 minOutputAmount,
    uint64 deadline
) public returns (uint256 output)
```

· inputToken – token yang akan dijual
· inputAmount – jumlah token yang akan dijual
· outputToken – token yang akan dibeli
· minOutputAmount – jumlah output minimum yang diharapkan (slippage protection)
· deadline – waktu kadaluwarsa transaksi

Modifier:

· revertIfZero(inputAmount) – memastikan inputAmount > 0
· revertIfDeadlinePassed(deadline) – memastikan transaksi belum kadaluwarsa

Logika Utama

1. Ambil cadangan pool:
   ```solidity
   uint256 inputReserves = inputToken.balanceOf(address(this));
   uint256 outputReserves = outputToken.balanceOf(address(this));
   ```
2. Hitung outputAmount berdasarkan rumus fee 0.3%:
   ```solidity
   uint256 outputAmount = getOutputAmountBasedOnInput(inputAmount, inputReserves, outputReserves);
   ```
3. Periksa slippage:
   ```solidity
   if (outputAmount < minOutputAmount) {
       revert TSwapPool__OutputTooLow(outputAmount, minOutputAmount);
   }
   ```
4. Lakukan swap dengan memanggil _swap:
   ```solidity
   _swap(inputToken, inputAmount, outputToken, outputAmount);
   ```

Kesimpulan: Fungsi ini secara struktural benar, tidak ada bug baru selain yang sudah ditemukan di getOutputAmountBasedOnInput (magic numbers).

---

Fungsi _swap (Internal)

Fungsi ini adalah jantung eksekusi swap. Di sinilah letak pelanggaran invariant yang sudah ditemukan oleh fuzzing.

Pemeriksaan Token

```solidity
if (_isUnknown(inputToken) || _isUnknown(outputToken) || inputToken == outputToken) {
    revert TSwapPool__InvalidToken();
}
```

· _isUnknown mengembalikan true jika token bukan weth atau poolToken → hanya dua token yang valid dalam satu pool.
· Menolak swap token yang sama (buang-buang gas).

Pelanggaran Invariant (High Severity)

```solidity
// @Audit-High - Breaks protocol invariant
swap_count++;
if (swap_count >= SWAP_COUNT_MAX) {
    swap_count = 0;
    outputToken.safeTransfer(msg.sender, 1_000_000_000_000_000_000);
}
```

Setiap 10 swap, protokol mengirim 1 token ekstra (1e18 wei) ke msg.sender. Ini melanggar invariant x * y = k karena saldo pool berubah tanpa melalui mekanisme swap yang seharusnya.

Dampak: Harga dalam pool menjadi tidak akurat, memungkinkan arbitrase dan potensi kerugian bagi LP.

Event dan Transfer

```solidity
emit Swap(msg.sender, inputToken, inputAmount, outputToken, outputAmount);
inputToken.safeTransferFrom(msg.sender, address(this), inputAmount);
outputToken.safeTransfer(msg.sender, outputAmount);
```

· Event Swap sudah baik (parameter terurut).
· Transfer menggunakan safeTransfer – aman terhadap token aneh.

---

Catatan Tambahan

· Beberapa tim mungkin menanamkan invariant langsung dalam kode (misal require(x * y >= k) setelah swap) sebagai lapisan perlindungan ekstra. TSwap tidak melakukannya.
· Fungsi swapExactOutput akan diperiksa selanjutnya.

---

Ringkasan Temuan

Temuan Lokasi Severity
Fungsi swapExactInput tanpa NatSpec swapExactInput Informational
Magic numbers (997, 1000) getOutputAmountBasedOnInput Informational
Transfer token ekstra setiap 10 swap _swap High (melanggar invariant)

---

Kesimpulan

swapExactInput secara struktural baik, namun _swap mengandung bug serius yang merusak invariant protokol. Bug ini sudah terdeteksi oleh fuzzing dan harus diperbaiki dengan menghapus mekanisme insentif yang mengubah saldo pool tanpa mengikuti constant product formula.

"Some teams may even hard code their invariant into their code base, something akin to require(x * y = k) as an extra layer of assurance that invariants won't break."