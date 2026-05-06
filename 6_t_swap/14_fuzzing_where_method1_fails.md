# Closed Stateful Fuzzing dengan Handler – Mengatasi Path Explosion

## Latar Belakang

Setelah mempelajari *stateless fuzzing* dan *open stateful fuzzing*, kita sekarang menghadapi kontrak yang lebih kompleks: `HandlerStatefulFuzzCatches.sol`. Kontrak ini mirip dengan *vault* ERC20 yang akan kita temui di TSwap.

## Kontrak Target: HandlerStatefulFuzzCatches

Kontrak ini memiliki invariant:

> **INVARIANT:** Pengguna harus selalu bisa menarik (withdraw) jumlah persis dari saldo yang mereka setorkan.

Fungsi utama:

```solidity
function depositToken(IERC20 token, uint256 amount) external requireSupportedToken(token) {
    tokenBalances[msg.sender][token] += amount;
    token.safeTransferFrom(msg.sender, address(this), amount);
}

function withdrawToken(IERC20 token) external requireSupportedToken(token) {
    uint256 currentBalance = tokenBalances[msg.sender][token];
    tokenBalances[msg.sender][token] = 0;
    token.safeTransfer(msg.sender, currentBalance);
}
```

Hanya token yang didukung (didaftarkan di `tokenIsSupported`) yang dapat digunakan.

## Upaya Open Stateful Fuzzing (Gagal)

Kita mencoba pendekatan *open stateful fuzzing* seperti sebelumnya:

1. Buat kontrak test dengan `StdInvariant`, `targetContract()`.
2. Siapkan *mock token* (`MockUSDC`, `YieldERC20`).
3. Beri `user` sejumlah token.
4. Tulis *invariant test* yang melakukan `withdrawToken` untuk kedua token dan membandingkan saldo akhir dengan `startingAmount`.

```solidity
function statefulFuzz_TestInvariantBreaks() public {
    vm.startPrank(user);
    handlerstatefulFuzzCatches.withdrawToken(IERC20(address(mockUSDC)));
    handlerstatefulFuzzCatches.withdrawToken(IERC20(address(yieldERC20)));
    vm.stopPrank();

    assert(handlerstatefulFuzzCatches.tokenBalances(user, IERC20(address(mockUSDC))) == 0);
    assert(handlerstatefulFuzzCatches.tokenBalances(user, IERC20(address(yieldERC20))) == 0);
    assert(mockUSDC.balanceOf(user) == startingAmount);
    assert(yieldERC20.balanceOf(user) == startingAmount);
}
```

Saat dijalankan dengan `fail_on_revert = false`, test **lulus** (padahal seharusnya tidak). Dengan `fail_on_revert = true`, kita melihat banyak *revert* karena `HandlerStatefulFuzzCatches__UnsupportedToken()`.

## Masalah: Path Explosion

- Fuzzer memanggil fungsi secara acak dengan **alamat token acak**.
- Hanya **2 token** yang didukung, tetapi fuzzer bisa memilih alamat token apa pun (termasuk acak tidak valid).
- Akibatnya, sebagian besar panggilan *revert* karena token tidak didukung.
- Fuzzer membuang banyak waktu pada jalur yang tidak relevan → tidak pernah menemukan *invariant* yang sebenarnya rusak.

Ini disebut **path explosion** – terlalu banyak kemungkinan jalur sehingga fuzzer kesulitan menemukan celah.

## Solusi: Closed Stateful Fuzzing dengan Handler

Kita perlu **membatasi** (*confine*) ruang lingkup fuzzing agar hanya menggunakan token yang didukung dan urutan operasi yang masuk akal. Pendekatan ini disebut **closed stateful fuzzing** dengan **handler**.

- **Handler** adalah kontrak terpisah yang bertindak sebagai "pintu gerbang" antara fuzzer dan kontrak target.
- Handler akan:
  - Menerima input dari fuzzer (misal: `uint256 tokenIndex` bukan `address token`).
  - Memetakan indeks tersebut ke token yang benar-benar didukung.
  - Melakukan *approve* token jika diperlukan.
  - Memanggil fungsi kontrak target dengan parameter yang valid.

Dengan handler, kita mengurangi *path explosion* secara drastis dan membuat fuzzing lebih efisien.

## Kekurangan Open Stateful Fuzzing (Rangkuman)

| Kelebihan | Kekurangan |
|-----------|------------|
| Mudah diimplementasikan | **Path explosion** – terlalu banyak jalur tidak valid |
| Tidak perlu kode tambahan | Bisa melewatkan *invariant* karena fuzzer sibuk dengan *revert* |
| Cocok untuk protokol sederhana | Sulit menangani prasyarat seperti *approve* token |

## Langkah Selanjutnya

Kita akan:
1. Membuat **handler contract** yang mengelola token yang didukung dan *approve*.
2. Mengarahkan fuzzer ke handler, bukan langsung ke kontrak target.
3. Menulis *invariant test* yang lebih fokus dan andal.