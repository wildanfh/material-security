# Handler.t.sol – Membangun Handler untuk Stateful Fuzzing TSwap

## Pendahuluan

Handler bertindak sebagai pembungkus (wrapper) yang membatasi dan mengarahkan panggilan fuzzer ke kontrak `TSwapPool`. Dengan handler, kita dapat mengontrol input, melacak perubahan state, dan akhirnya memverifikasi invariant `∆x = (β/(1-β)) * x`.

> **Catatan:** Bagian ini adalah salah satu yang paling kompleks dalam kursus. Jika belum sepenuhnya paham, ikuti saja dulu. *Practice makes perfect.*

---

## Boilerplate Handler

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {Test, console2} from "forge-std/Test.sol";
import {TSwapPool} from "../../src/TSwapPool.sol";

contract Handler is Test {
    TSwapPool pool;

    constructor(TSwapPool _pool) {
        pool = _pool;
    }
}
```

---

Menambah Konteks: Fungsi yang Akan Diuji

Fungsi utama di TSwapPool yang relevan dengan invariant:

· deposit
· withdraw
· swapExactOutput
· swapExactInput (tidak memiliki dokumentasi – ini sendiri sudah menjadi temuan informational)

Dari NatSpec swapExactOutput:

Menghitung berapa banyak input yang diperlukan berdasarkan jumlah output yang diinginkan.

Parameter swapExactOutput:

· inputToken – token yang ingin ditukar pengguna (misal poolToken)
· outputToken – token yang ingin diterima (misal WETH)
· outputAmount – jumlah output yang diinginkan
· deadline – batas waktu (akan diisi block.timestamp)

---

Menginisialisasi Handler dengan Token yang Tepat

Handler perlu mengetahui token mana yang terkait dengan pool. Kita gunakan fungsi getter dari TSwapPool:

```solidity
import {ERC20Mock} from "../mocks/ERC20Mock.sol";

contract Handler is Test {
    TSwapPool pool;
    ERC20Mock weth;
    ERC20Mock poolToken;

    constructor(TSwapPool _pool) {
        pool = _pool;
        weth = ERC20Mock(_pool.getWeth());
        poolToken = ERC20Mock(_pool.getPoolToken());
    }
}
```

---

Ghost Variables (Variabel Hantu)

Kita perlu melacak perubahan nilai token sebelum dan sesudah aksi. Variabel ini hanya ada di handler, tidak di kontrak asli, disebut ghost variables.

```solidity
int256 public expectedDeltaY;
int256 public expectedDeltaX;
int256 startingY;
int256 startingX;
int256 public actualDeltaX;
int256 public actualDeltaY;
```

· expectedDeltaX / expectedDeltaY – perubahan yang diharapkan berdasarkan perhitungan invariant.
· actualDeltaX / actualDeltaY – perubahan nyata setelah eksekusi.

---

Fungsi deposit dalam Handler

Langkah-langkah dalam deposit:

1. Batasi input agar tidak melebihi batas wajar (misal uint64.max ≈ 18.5 ETH).
2. Catat state awal – saldo weth dan poolToken di handler (address ini adalah liquidity provider).
3. Hitung perubahan yang diharapkan – expectedDeltaX = jumlah WETH yang akan disetor; expectedDeltaY dipanggil dari pool.getPoolTokensToDepositBasedOnWeth().
4. Prank sebagai liquidity provider – mint token yang diperlukan, approve ke pool.
5. Panggil deposit pool.
6. Catat state akhir – hitung actualDeltaX dan actualDeltaY.

Kode lengkap:

```solidity
address liquidityProvider = address(this); // atau buat alamat terpisah

function deposit(uint256 wethAmount) public {
    wethAmount = bound(wethAmount, 0, type(uint64).max);

    startingY = int256(weth.balanceOf(address(this)));
    startingX = int256(poolToken.balanceOf(address(this)));
    expectedDeltaX = int256(wethAmount);
    expectedDeltaY = int256(pool.getPoolTokensToDepositBasedOnWeth(wethAmount));

    vm.startPrank(liquidityProvider);
    weth.mint(liquidityProvider, wethAmount);
    poolToken.mint(liquidityProvider, uint256(expectedDeltaY)); // perhatikan: expectedDeltaY adalah jumlah poolToken yang diperlukan
    weth.approve(address(pool), type(uint256).max);
    poolToken.approve(address(pool), type(uint256).max);

    pool.deposit(wethAmount, 0, uint256(expectedDeltaY), uint64(block.timestamp));
    vm.stopPrank();

    uint256 endingX = poolToken.balanceOf(address(this));
    uint256 endingY = weth.balanceOf(address(this));

    actualDeltaX = int256(endingX) - int256(startingX);
    actualDeltaY = int256(endingY) - int256(startingY);
}
```

Catatan: Parameter kedua deposit (minimumLiquidityTokensToMint) kita set 0 untuk menyederhanakan – tidak mempengaruhi invariant inti.

---

Mengapa Handler Diperlukan?

· Membatasi input – fuzzer tidak akan memanggil deposit dengan jumlah melebihi saldo atau token tidak dikenal.
· Melacak delta – handler menyimpan state awal dan akhir, memungkinkan kita memverifikasi rumus invariant.
· Mengelola prasyarat – seperti mint dan approve yang diperlukan sebelum deposit.

---

Selanjutnya

Setelah fungsi deposit siap, kita akan mengimplementasikan fungsi swapExactInput dan swapExactOutput di handler, kemudian menulis invariant test yang membandingkan expectedDeltaX dengan actualDeltaX sesuai rumus:

```
∆x = (β/(1-β)) * x
```

Dimana β = ∆y / y (perubahan Y relatif terhadap saldo Y awal).

"This lesson has been a lot already. Follow along as best you can and I promise it'll make sense by the end."