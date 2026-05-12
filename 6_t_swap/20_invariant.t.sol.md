# Invariant.t.sol – Setup Stateful Fuzz Testing untuk TSwap

## Pendahuluan

Setelah memahami *constant product formula*, kita akan menulis **stateful fuzz suite** untuk protokol TSwap. Ini adalah bagian paling menantang dalam kursus ini. Jangan berkecil hati jika kesulitan; Anda bisa kembali lagi setelah menyelesaikan bagian lain.

---

## Struktur Folder dan File

Buat file dan folder berikut dalam repositori TSwap:

```

test/invariant/
├── Invariant.t.sol
└── Handler.t.sol

```

---

## Invariant.t.sol – Setup Awal

Mulai dengan kode dasar:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {Test} from "forge-std/Test.sol";
import {StdInvariant} from "forge-std/StdInvariant.sol";

contract Invariant is StdInvariant, Test {

}
```

---

Membuat Mock ERC20

Kita perlu token palsu untuk pengujian. Buat folder test/mocks/ dan file ERC20Mock.sol:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract ERC20Mock is ERC20 {
    constructor() ERC20("Mock", "MOCK") {}

    function mint(address to, uint256 amount) public {
        _mint(to, amount);
    }
}
```

---

Mengimpor Kontrak yang Diperlukan

Tambahkan import untuk PoolFactory, TSwapPool, dan ERC20Mock:

```solidity
import {ERC20Mock} from "test/mocks/ERC20Mock.sol";
import {PoolFactory} from "../../src/PoolFactory.sol";
import {TSwapPool} from "../../src/TSwapPool.sol";
```

---

Mendeklarasikan Variabel dan Fungsi setUp

```solidity
contract Invariant is StdInvariant, Test {
    ERC20Mock poolToken;
    ERC20Mock weth;

    PoolFactory factory;
    TSwapPool pool;

    int256 constant STARTING_X = 100e18; // poolToken (ERC20)
    int256 constant STARTING_Y = 50e18;  // WETH

    function setUp() public {
        // 1. Deploy token mock
        poolToken = new ERC20Mock();
        weth = new ERC20Mock();

        // 2. Deploy factory dengan alamat weth
        factory = new PoolFactory(address(weth));

        // 3. Buat pool untuk pasangan poolToken-WETH
        pool = TSwapPool(factory.createPool(address(poolToken)));

        // 4. Mint token untuk kontrak test (address(this))
        poolToken.mint(address(this), uint256(STARTING_X));
        weth.mint(address(this), uint256(STARTING_Y));

        // 5. Approve pool untuk menghabiskan token
        poolToken.approve(address(pool), type(uint256).max);
        weth.approve(address(pool), type(uint256).max);

        // 6. Deposit awal ke pool (initial liquidity)
        pool.deposit(
            uint256(STARTING_Y),        // wethToDeposit
            uint256(STARTING_Y),        // minimumLiquidityTokensToMint
            uint256(STARTING_X),        // maximumPoolTokensToDeposit
            uint64(block.timestamp)     // deadline
        );
    }
}
```

---

Penjelasan Deposit Awal

Fungsi deposit di TSwapPool.sol memiliki logika khusus untuk deposit pertama (saat totalLiquidityTokenSupply() == 0):

· Parameter 1: jumlah WETH yang akan disetor (STARTING_Y).
· Parameter 2: jumlah minimum liquidity tokens yang akan dicetak – untuk deposit awal, ini menentukan 100% pasokan token likuiditas. Diset sama dengan nilai WETH.
· Parameter 3: jumlah token pool (ERC20) yang akan disetor (STARTING_X).
· Parameter 4: batas waktu (deadline) – tidak terlalu penting untuk pengujian.

Deposit awal ini menentukan rasio awal antara x (poolToken) dan y (WETH) yang mendefinisikan konstanta k dalam constant product formula (x * y = k).

---

Mengapa Perlu Handler?

Setelah setup selesai, kita bisa langsung menulis fungsi invariant test. Namun, memverifikasi rumus ∆x = (β/(1-β)) * x secara langsung dalam fuzz test sulit karena kita harus melacak perubahan saldo di setiap swap.

Dengan Handler, kita dapat:

· Membatasi input fuzzer hanya pada skenario yang masuk akal.
· Menyederhanakan pelacakan ∆x dan ∆y.
· Menghindari path explosion.

Handler akan menjadi subjek pelajaran berikutnya.

---

Kesimpulan

· Setup Invariant.t.sol meliputi pembuatan mock token, deployment factory, pembuatan pool, dan deposit awal likuiditas.
· Deposit awal menentukan rasio awal x dan y yang menjadi dasar invariant x * y = k.
· Verifikasi invariant secara langsung rumit; kita akan menggunakan Handler untuk mempermudah pengujian stateful fuzzing.

"This is the most challenging part of this entire course."