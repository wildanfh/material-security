# One Last Huzzah – Menemukan Bug Tersembunyi dengan Fuzzing TSwap

## Pendahuluan

Setelah berhasil menjalankan `statefulFuzz_constantProductFormulaStaysTheSameY` (yang hanya menguji perubahan `poolToken` / `expectedDeltaY`), kita perlu menguji sisi lain dari invariant: perubahan `weth` / `expectedDeltaX`. Ternyata test ini gagal dan mengungkap bug tersembunyi dalam protokol TSwap.

---

## Menambahkan Invariant Test untuk Sisi X

Kita tulis fungsi test kedua yang membandingkan `actualDeltaX` dengan `expectedDeltaX`:

```solidity
function statefulFuzz_constantProductFormulaStaysTheSameX() public {
    assertEq(handler.actualDeltaX(), handler.expectedDeltaX());
}
```

Menjalankan test:

```bash
forge test --mt statefulFuzz_constantProductFormulaStaysTheSameX -vvvv
```

**Hasil:** Error – `expectedDeltaX` dan `actualDeltaX` sangat berbeda.

---

## Debugging – Melacak Panggilan Fungsi

Dengan flag `-vvvv`, trace menunjukkan bahwa fungsi `swapPoolTokenForWethBasedOnOutputWeth` dipanggil. Di dalamnya, `swapExactOutput` seharusnya melakukan **dua transfer**:
1. `swapper` → `TSwapPool` (token input)
2. `TSwapPool` → `swapper` (token output)

Namun trace menunjukkan **tiga transfer** terjadi. Ini mencurigakan.

---

## Menyelidiki `swapExactOutput` dan `_swap`

Kode `swapExactOutput` di `TSwapPool.sol`:

```solidity
function swapExactOutput(...) public returns (uint256 inputAmount) {
    uint256 inputReserves = inputToken.balanceOf(address(this));
    uint256 outputReserves = outputToken.balanceOf(address(this));
    inputAmount = getInputAmountBasedOnOutput(outputAmount, inputReserves, outputReserves);
    _swap(inputToken, inputAmount, outputToken, outputAmount);
}
```

Selanjutnya, fungsi internal `_swap`:

```solidity
function _swap(...) private {
    // ... validasi token
    swap_count++;
    if (swap_count >= SWAP_COUNT_MAX) {
        swap_count = 0;
        outputToken.safeTransfer(msg.sender, 1_000_000_000_000_000_000);
    }
    emit Swap(...);
    inputToken.safeTransferFrom(msg.sender, address(this), inputAmount);
    outputToken.safeTransfer(msg.sender, outputAmount);
}
```

**Penemuan:** Setiap 10 swap (`swap_count >= SWAP_COUNT_MAX`), protokol mentransfer **1 token ekstra** (1e18 wei) ke `msg.sender` sebagai insentif.

> Komentar dalam kode: *"Every 10 swaps, we give the caller an extra token as an extra incentive to keep trading on T-Swap."*

---

## Mengapa Ini Melanggar Invariant?

- **Constant product formula** (`x * y = k`) mengasumsikan tidak ada transfer token di luar swap.
- Transfer ekstra ini **mengubah saldo pool tanpa melalui mekanisme swap** yang seharusnya.
- Akibatnya, `actualDeltaX` (perubahan nyata saldo pool) tidak sesuai dengan `expectedDeltaX` (perubahan yang dihitung dari invariant).
- **Bug ini mirip dengan token fee‑on‑transfer** – token yang memotong biaya setiap transfer. TSwap sendiri justru melakukan transfer ekstra, sehingga rentan terhadap token semacam itu (karena asumsi invariant sudah tidak terpenuhi).

---

## Kesimpulan dan Pembelajaran

- **Fuzzing stateful dengan handler** sangat ampuh untuk menemukan kerentanan yang tidak terduga, bahkan yang tersembunyi dalam logika internal protokol.
- **Jangan hanya menguji satu sisi invariant** – uji semua variabel yang mungkin terpengaruh.
- **Bug ini tidak akan terdeteksi** oleh unit test biasa atau stateless fuzzing, karena memerlukan urutan panggilan (stateful) dan invariant test yang mencakup kedua token.
- **Transfer ekstra ini juga membuat TSwap tidak kompatibel dengan token fee‑on‑transfer** – karena invariant sudah dilanggar oleh protokol itu sendiri.

> *"This was a lot of work to find this bug, there's no denying that, but fuzz test suites are so incredibly powerful."*

---

## Bacaan Lanjutan

- [**Nascent.xyz – "You're Writing Require Statements Wrong"**](https://www.nascent.xyz/idea/youre-writing-require-statements-wrong) – konsep invariant yang langsung diintegrasikan dalam desain protokol (FREI‑PI), yang diterapkan di versi Uniswap selanjutnya.
- **Latihan lanjutan:** Coba modifikasi `_swap` agar tidak merusak invariant, misal dengan menyesuaikan `k` setiap kali transfer ekstra terjadi.

> *"Mastering them is worth putting the time in for given the advantages they bring to this process when a code base is very complex, such as automation."*