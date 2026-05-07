# Handler-based Stateful Fuzzing – Mengarahkan Fuzzer dengan Handler

## Mengapa Perlu Handler?

*Open stateful fuzzing* sering mengalami *path explosion* karena fuzzer dapat memanggil fungsi dengan parameter acak yang tidak valid (misal alamat token tidak didukung). **Handler** adalah kontrak pembungkus (*wrapper*) yang bertindak sebagai perantara antara fuzzer dan kontrak target, untuk:

- **Membatasi input** hanya pada skenario yang realistis.
- **Mengurangi revert** karena parameter tidak valid.
- **Meningkatkan efisiensi** fuzzing sehingga lebih mungkin menemukan *invariant* yang rusak.

---

## Struktur Dasar Handler

Handler mewarisi `Test` dari Foundry, mengimpor kontrak target dan *mock token*, serta menyimpan referensi ke kontrak target, token yang didukung, dan pengguna (user) yang akan bertindak.

Contoh: `Handler.t.sol`

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

import {HandlerStatefulFuzzCatches} from "src/invariant-break/HandlerStatefulFuzzCatches.sol";
import {Test} from "forge-std/Test.sol";
import {MockUSDC} from "test/mocks/MockUSDC.sol";
import {YieldERC20} from "test/mocks/YieldERC20.sol";

contract Handler is Test {
    HandlerStatefulFuzzCatches handlerStatefulFuzzCatches;
    MockUSDC mockUSDC;
    YieldERC20 yieldERC20;
    address user;

    constructor(HandlerStatefulFuzzCatches _handlerStatefulFuzzCatches, MockUSDC _mockUSDC, YieldERC20 _yieldERC20, address _user) {
        handlerStatefulFuzzCatches = _handlerStatefulFuzzCatches;
        mockUSDC = _mockUSDC;
        yieldERC20 = _yieldERC20;
        user = _user;
    }
}
```

---

## Membatasi Fungsi Fuzzing

Alih‑alih fuzzer memanggil `depositToken(address,uint256)` secara langsung, handler menyediakan fungsi‑fungsi spesifik untuk setiap token yang didukung. Di dalam fungsi tersebut, kita juga melakukan:

- **Batasan jumlah** deposit (`bound`).
- **Approve token** sebelum transfer.
- **Prank** sebagai `user`.

Contoh fungsi deposit:

```solidity
function depositYieldERC20(uint256 _amount) public {
    uint256 amount = bound(_amount, 0, yieldERC20.balanceOf(user));
    vm.startPrank(user);
    yieldERC20.approve(address(handlerStatefulFuzzCatches), amount);
    handlerStatefulFuzzCatches.depositToken(yieldERC20, amount);
    vm.stopPrank();
}

function depositMockUSDC(uint256 _amount) public {
    uint256 amount = bound(_amount, 0, mockUSDC.balanceOf(user));
    vm.startPrank(user);
    mockUSDC.approve(address(handlerStatefulFuzzCatches), amount);
    handlerStatefulFuzzCatches.depositToken(mockUSDC, amount);
    vm.stopPrank();
}
```

Fungsi *withdraw* juga disediakan tanpa parameter (karena penarikan seluruh saldo):

```solidity
function withdrawYieldERC20() public {
    vm.startPrank(user);
    handlerStatefulFuzzCatches.withdrawToken(yieldERC20);
    vm.stopPrank();
}

function withdrawMockUSDC() public {
    vm.startPrank(user);
    handlerStatefulFuzzCatches.withdrawToken(mockUSDC);
    vm.stopPrank();
}
```

Dengan ini, fuzzer hanya akan memanggil fungsi‑fungsi yang sah dan tidak pernah mencoba token tidak didukung.

---

## Menghubungkan Handler ke Invariant Test

Di file `Invariant.t.sol`, kita melakukan:

1. **Import** `Handler` dan komponen lain.
2. **Deploy** kontrak target (`HandlerStatefulFuzzCatches`) beserta token‑tokennya.
3. **Deploy** `Handler` dengan meneruskan alamat target, token, dan user.
4. **Tentukan *selector* fungsi** dari handler yang boleh dipanggil fuzzer.

```solidity
import {Handler} from "test/invariant-break/handler/Handler.t.sol";

contract AttemptedBreakTest is StdInvariant, Test {
    Handler handler;
    // ... variabel lain

    function setUp() public {
        // ... deploy target, mint token, dll
        handler = new Handler(handlerstatefulFuzzCatches, mockUSDC, yieldERC20, user);

        bytes4[] memory selectors = new bytes4[](4);
        selectors[0] = handler.depositYieldERC20.selector;
        selectors[1] = handler.depositMockUSDC.selector;
        selectors[2] = handler.withdrawYieldERC20.selector;
        selectors[3] = handler.withdrawMockUSDC.selector;

        targetSelector(FuzzSelector({addr: address(handler), selectors: selectors}));
    }
}
```

Fungsi `targetSelector` membatasi fuzzer hanya pada keempat fungsi tersebut, dengan parameter yang sudah dibatasi handler.

---

## Menulis Invariant Test

Invariant test kita tetap sama secara logika: setelah menyetor dan menarik seluruh token, saldo pengguna harus kembali ke jumlah awal. Namun sekarang kita memanggil fungsi `withdrawToken` langsung pada kontrak target (bukan melalui handler) untuk memverifikasi *invariant*.

```solidity
function statefulFuzz_testInvariantBreaksHandler() public {
    vm.startPrank(user);
    handlerStatefulFuzzCatches.withdrawToken(mockUSDC);
    handlerStatefulFuzzCatches.withdrawToken(yieldERC20);
    vm.stopPrank();

    assert(mockUSDC.balanceOf(address(handlerStatefulFuzzCatches)) == 0);
    assert(yieldERC20.balanceOf(address(handlerStatefulFuzzCatches)) == 0);
    assert(mockUSDC.balanceOf(user) == startingAmount);
    assert(yieldERC20.balanceOf(user) == startingAmount);
}
```

Perhatikan bahwa kita masih menggunakan `handlerStatefulFuzzCatches` (kontrak target) untuk *withdraw* di dalam test, bukan handler. Namun seluruh proses *deposit* dan *withdraw* yang dilakukan fuzzer selama *fuzzing* berlangsung dipandu oleh handler, sehingga jalur yang diuji relevan.

---

## Keuntungan Pendekatan Handler

| Sebelum (Open Stateful Fuzzing) | Sesudah (Handler‑based) |
|--------------------------------|--------------------------|
| Fuzzer memanggil fungsi dengan alamat token acak → banyak revert. | Fungsi deposit/withdraw hanya untuk token yang didukung. |
| Tidak ada *approve* token → revert. | Handler melakukan *approve* sebelum deposit. |
| Path explosion → fuzzer tidak menemukan bug. | Jalur fuzzing terbatas dan realistis → efisien. |
| Sulit menguji skenario multi‑token. | Mudah menambah token baru dengan membuat fungsi handler terkait. |

## Kesimpulan

- **Handler** adalah pola desain untuk *closed stateful fuzzing* yang membatasi masukan fuzzer agar sesuai dengan logika protokol.
- Dengan handler, kita dapat menghilangkan *revert* yang tidak relevan dan mengarahkan fuzzer ke skenario yang benar‑benar mungkin terjadi.
- Foundry menyediakan `targetSelector` untuk menentukan fungsi mana pada handler yang boleh dipanggil fuzzer.
- Hasilnya: pengujian *invariant* menjadi lebih andal, lebih cepat, dan lebih mungkin menemukan kerentanan tersembunyi yang bergantung pada urutan panggilan fungsi.

> *Handler-based fuzzing adalah kunci untuk menguji invariant pada protokol DeFi yang kompleks.*
