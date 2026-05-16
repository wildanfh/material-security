# Final Invariant & Tweaks – Menyatukan Semua untuk Fuzzing TSwap

## Pendahuluan

Setelah handler selesai, langkah terakhir adalah menyambungkannya ke `Invariant.t.sol` dan menulis fungsi invariant test yang sederhana namun kuat. Berkat semua persiapan (setup pool, deposit awal, handler, ghost variables), fungsi test final menjadi sangat ringkas.

---

## Invariant Test – Satu Baris Assertion

Fungsi invariant test hanya membandingkan `actualDeltaY` (perubahan nyata poolToken di pool) dengan `expectedDeltaY` (perubahan yang dihitung berdasarkan rumus invariant). Kita akan menguji satu token terlebih dahulu, misalnya poolToken.

```solidity
function statefulFuzz_constantProductFormulaStaysTheSameY() public {
    assertEq(handler.actualDeltaY(), handler.expectedDeltaY());
}
```

- `handler.actualDeltaY()` – diambil dari variabel ghost dalam handler yang menyimpan perubahan nyata poolToken setelah deposit atau swap.
- `handler.expectedDeltaY()` – dihitung menggunakan rumus invariant (baik lewat `getPoolTokensToDepositBasedOnWeth` untuk deposit atau `getInputAmountBasedOnOutput` untuk swap).

Jika kedua nilai selalu sama (dalam toleransi rounding), maka invariant `∆x = (β/(1-β)) * x` terpenuhi.

---

## Menghubungkan Handler sebagai Target Fuzzing

Di `setUp()`, kita perlu:

1. Membuat instance `Handler` dengan mengirimkan alamat pool.
2. Menentukan fungsi mana saja dari handler yang boleh dipanggil fuzzer (`deposit` dan `swapPoolTokenForWethBasedOnOutputWeth`).
3. Mengarahkan fuzzer ke handler tersebut sebagai `targetContract` dan `targetSelector`.

Kode tambahan dalam `setUp()`:

```solidity
import {Handler} from "./Handler.t.sol";

contract Invariant is StdInvariant, Test {
    Handler handler;

    function setUp() public {
        // ... semua setup pool dan deposit awal seperti sebelumnya ...

        handler = new Handler(pool);
        bytes4[] memory selectors = new bytes4[](2);
        selectors[0] = handler.deposit.selector;
        selectors[1] = handler.swapPoolTokenForWethBasedOnOutputWeth.selector;
        targetSelector(FuzzSelector({ addr: address(handler), selectors: selectors }));
        targetContract(address(handler));
    }
}
```

- `targetSelector` membatasi fuzzer hanya pada dua fungsi yang kita tentukan. Ini mencegah pemanggilan fungsi lain yang tidak relevan atau merusak state.
- `targetContract` memberi tahu Foundry bahwa kontrak target untuk stateful fuzzing adalah handler, bukan pool langsung. Handler akan meneruskan panggilan ke pool setelah melakukan pencatatan delta dan pembatasan input.

---

## Kode Lengkap Invariant.t.sol

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import { Test } from "forge-std/Test.sol";
import { StdInvariant } from "forge-std/StdInvariant.sol";
import { ERC20Mock } from "test/mocks/ERC20Mock.sol";
import { PoolFactory } from "../../src/PoolFactory.sol";
import { TSwapPool } from "../../src/TSwapPool.sol";
import { Handler } from "./Handler.t.sol";

contract Invariant is StdInvariant, Test {
    ERC20Mock poolToken;
    ERC20Mock weth;

    PoolFactory factory;
    TSwapPool pool;
    int256 public constant STARTING_X = 100e18;
    int256 public constant STARTING_Y = 50e18;

    Handler handler;

    function setUp() public {
        poolToken = new ERC20Mock();
        weth = new ERC20Mock();
        factory = new PoolFactory(address(weth));
        pool = TSwapPool(factory.createPool(address(poolToken)));

        poolToken.mint(address(this), uint256(STARTING_X));
        weth.mint(address(this), uint256(STARTING_Y));

        poolToken.approve(address(pool), type(uint256).max);
        weth.approve(address(pool), type(uint256).max);

        pool.deposit(uint256(STARTING_Y), uint256(STARTING_Y), uint256(STARTING_X), uint64(block.timestamp));

        handler = new Handler(pool);
        bytes4[] memory selectors = new bytes4[](2);
        selectors[0] = handler.deposit.selector;
        selectors[1] = handler.swapPoolTokenForWethBasedOnOutputWeth.selector;
        targetSelector(FuzzSelector({ addr: address(handler), selectors: selectors }));
        targetContract(address(handler));
    }

    function statefulFuzz_constantProductFormulaStaysTheSameY() public view {
        assertEq(handler.actualDeltaY(), handler.expectedDeltaY());
    }
}
```

> **Catatan:** Fungsi test dapat ditambahkan juga untuk `actualDeltaX` jika diperlukan, namun satu invariant test sudah cukup untuk memverifikasi rumus dasar karena `actualDeltaY` dan `expectedDeltaY` sudah mencakup perubahan poolToken yang mencerminkan invariant.

---

## Kesimpulan

- **Invariant test menjadi sangat sederhana** karena semua kompleksitas (batasan input, perhitungan delta, ghost variables) sudah dienkapsulasi di dalam `Handler`.
- **`targetSelector` dan `targetContract`** mengarahkan fuzzer hanya ke fungsi deposit dan swap melalui handler.
- **Assertion membandingkan delta aktual dengan delta yang diharapkan** berdasarkan rumus invariant. Jika tidak pernah fail, maka invariant terpenuhi.
- Pendekatan ini menunjukkan bagaimana **handler‑based stateful fuzzing** dapat diterapkan pada protokol DeFi nyata seperti TSwap.

> *"Our actual test is going to be surprisingly basic because of all the set up we've done!"*