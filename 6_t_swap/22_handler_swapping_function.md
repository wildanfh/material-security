# Handler.t.sol – Menambahkan Fungsi Swap untuk Menguji Invariant TSwap

## Pendahuluan

Setelah berhasil membuat fungsi `deposit` dalam handler, langkah selanjutnya adalah menambahkan fungsi swap. Tujuannya adalah menguji invariant inti TSwap:

```

∆x = (β/(1-β)) * x

```

Di mana `β = ∆y / y` (perubahan Y relatif terhadap saldo Y awal).

---

## Memanfaatkan Fungsi `getInputAmountBasedOnOutput`

Kontrak `TSwapPool` menyediakan fungsi `getInputAmountBasedOnOutput` yang menghitung jumlah input yang diperlukan (dalam poolToken) untuk mendapatkan sejumlah output tertentu (dalam WETH). Ini akan menjadi `∆x` yang diharapkan.

Fungsi tersebut (disederhanakan tanpa fee):

```solidity
function getInputAmountBasedOnOutput(
    uint256 outputAmount,    // ∆y (weth yang diinginkan)
    uint256 inputReserves,   // total poolToken di pool
    uint256 outputReserves   // total weth di pool
) public pure returns (uint256 inputAmount) {
    return (inputReserves * outputAmount) / (outputReserves - outputAmount);
}
```

Dengan fee, rumusnya menjadi:

```solidity
return ((inputReserves * outputAmount) * 10000) / ((outputReserves - outputAmount) * 997);
```

(10000 dan 997 merepresentasikan fee 0.3%).

---

Implementasi Fungsi Swap dalam Handler

Kita akan membuat fungsi swapPoolTokenForWethBasedOnOutputWeth yang menerima outputWeth (jumlah WETH yang ingin diperoleh) sebagai parameter. Langkah-langkahnya:

1. Batasi nilai outputWeth agar tidak melebihi saldo WETH pool dan tidak lebih kecil dari deposit minimum (untuk menghindari error).
2. Hitung poolTokenAmount (jumlah poolToken yang perlu dibayarkan) menggunakan getInputAmountBasedOnOutput.
3. Catat state awal – saldo poolToken dan WETH di pool (startingX, startingY).
4. Tentukan perubahan yang diharapkan – expectedDeltaX = -outputWeth (berkurangnya WETH), expectedDeltaY = poolTokenAmount (bertambahnya poolToken).
5. Siapkan swapper – buat alamat baru (swapper), mint poolToken secukupnya jika saldonya kurang.
6. Lakukan swap dengan swapExactOutput (kita menentukan output yang diinginkan).
7. Catat state akhir – hitung actualDeltaX dan actualDeltaY sebagai selisih saldo pool setelah swap.

Kode Lengkap Fungsi Swap

```solidity
function swapPoolTokenForWethBasedOnOutputWeth(uint256 outputWeth) public {
    // Hindari swap jika saldo WETH pool terlalu kecil
    if (weth.balanceOf(address(pool)) <= pool.getMinimumWethDepositAmount()) {
        return;
    }

    // Batasi outputWeth antara minimum deposit dan uint64.max
    outputWeth = bound(outputWeth, pool.getMinimumWethDepositAmount(), type(uint64).max);
    if (outputWeth >= weth.balanceOf(address(pool))) {
        return;
    }

    // Hitung input poolToken yang diperlukan
    uint256 poolTokenAmount = pool.getInputAmountBasedOnOutput(
        outputWeth,
        poolToken.balanceOf(address(pool)),
        weth.balanceOf(address(pool))
    );

    if (poolTokenAmount >= type(uint64).max) return;

    // Catat state awal
    startingY = int256(poolToken.balanceOf(address(pool)));
    startingX = int256(weth.balanceOf(address(pool)));
    expectedDeltaX = -1 * int256(outputWeth);   // berkurangnya WETH
    expectedDeltaY = int256(poolTokenAmount);   // bertambahnya poolToken

    // Pastikan swapper memiliki cukup poolToken
    if (poolToken.balanceOf(swapper) < poolTokenAmount) {
        poolToken.mint(swapper, poolTokenAmount - poolToken.balanceOf(swapper) + 1);
    }

    // Eksekusi swap
    vm.startPrank(swapper);
    poolToken.approve(address(pool), type(uint256).max);
    pool.swapExactOutput(poolToken, weth, outputWeth, uint64(block.timestamp));
    vm.stopPrank();

    // Catat state akhir dan hitung delta aktual
    uint256 endingY = poolToken.balanceOf(address(pool));
    uint256 endingX = weth.balanceOf(address(pool));
    actualDeltaY = int256(endingY) - int256(startingY);
    actualDeltaX = int256(endingX) - int256(startingX);
}
```

---

Penjelasan Variabel Hantu (Ghost Variables)

Handler menyimpan beberapa variabel yang hanya ada di dalam handler (tidak di kontrak asli) untuk melacak perubahan:

· startingX, startingY – saldo pool sebelum aksi.
· expectedDeltaX, expectedDeltaY – perubahan yang diharapkan berdasarkan invariant.
· actualDeltaX, actualDeltaY – perubahan nyata yang terjadi.

Nantinya, dalam invariant test, kita akan membandingkan expectedDeltaX dengan actualDeltaX dan expectedDeltaY dengan actualDeltaY untuk memverifikasi bahwa invariant ∆x = (β/(1-β)) * x selalu terpenuhi.

---

Ringkasan Handler TSwap

Handler yang telah lengkap mencakup:

· Fungsi deposit untuk menambah likuiditas.
· Fungsi swapPoolTokenForWethBasedOnOutputWeth untuk melakukan swap.
· Variabel ghost untuk melacak delta yang diharapkan dan aktual.
· Alamat terpisah untuk liquidity provider (lp) dan swapper.

Dengan handler ini, fuzzer dapat memanggil fungsi deposit dan swap secara acak, sambil mempertahankan state, sehingga kita dapat menguji invariant TSwap secara otomatis.

"The math and working out these invariants can be quite difficult, but it's important to keep in mind what we're trying to do."