# Open Stateful Fuzzing – Mengatasi Keterbatasan Stateless Fuzzing

## Pengantar

**Stateful fuzzing** (juga dikenal sebagai *invariant testing*) adalah metode pengujian di mana **hasil setiap run menjadi input untuk run berikutnya**. Ini memungkinkan kita menguji *invariant* dalam kondisi *state* kontrak yang terus berubah, seperti halnya protokol yang sudah di‑deploy di dunia nyata.

---

## Contoh Kontrak yang Tidak Bisa Diuji dengan Stateless Fuzzing

Kontrak `StatefulFuzzCatches.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

// INVARIANT: doMoreMathAgain should never return 0
contract StatefulFuzzCatches {
    uint256 public myValue = 1;
    uint256 public storedValue = 100;

    function doMoreMathAgain(uint128 myNumber) public returns (uint256) {
        uint256 response = (uint256(myNumber) / 1) + myValue;
        storedValue = response;
        return response;
    }

    function changeValue(uint256 newValue) public {
        myValue = newValue;
    }
}
```

Invariant: doMoreMathAgain tidak boleh mengembalikan 0.

Masalah: Kerentanan hanya muncul jika changeValue dipanggil terlebih dahulu untuk mengubah myValue menjadi 0, lalu doMoreMathAgain dipanggil dengan input 0, menyebabkan fungsi mengembalikan 0. Stateless fuzzing tidak bisa menangkap karena setiap run dimulai dengan state awal myValue = 1.

---

Stateless Fuzz Test (Gagal)

```solidity
import {StatefulFuzzCatches} from "src/invariant-break/StatefulFuzzCatches.sol";
import {Test} from "forge-std/Test.sol";

contract StatefulFuzzCatchesTest is Test {
    StatefulFuzzCatches statefulFuzzCatches;

    function setUp() public {
        statefulFuzzCatches = new StatefulFuzzCatches();
    }

    function testDoMoreMathAgain(uint128 data) public view {
        assert(statefulFuzzCatches.doMoreMathAgain(data) != 0);
    }
}
```

Hasil: Test selalu lulus karena setiap run state direset. Kerentanan tidak ditemukan meskipun dijalankan 100.000 kali.

---

Stateful Fuzz Test (Berhasil)

Kita perlu mengimpor StdInvariant, mewarisinya, dan menentukan target contract.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

import {StatefulFuzzCatches} from "src/invariant-break/StatefulFuzzCatches.sol";
import {Test} from "forge-std/Test.sol";
import {StdInvariant} from "forge-std/StdInvariant.sol";

contract StatefulFuzzCatchesTest is StdInvariant, Test {
    StatefulFuzzCatches statefulFuzzCatches;

    function setUp() public {
        statefulFuzzCatches = new StatefulFuzzCatches();
        targetContract(address(statefulFuzzCatches)); // Tentukan kontrak target
    }

    function statefulFuzz_catchesInvariant() public view {
        assert(statefulFuzzCatches.storedValue() != 0);
    }
}
```

Catatan penting: Fungsi stateful fuzz test harus diawali dengan kata kunci statefulFuzz_ atau invariant_. Ini memberi tahu Foundry cara menjalankan tes.

---

Konfigurasi Invariant di foundry.toml

```toml
[invariant]
runs = 64          # Jumlah fuzz test yang akan dijalankan
depth = 32          # Jumlah fungsi acak yang dipanggil per run
fail_on_revert = true   # Apakah test gagal jika terjadi revert?
```

· runs : Berapa kali proses fuzzing diulang.
· depth : Dalam satu run, berapa banyak fungsi yang akan dipanggil secara acak (berurutan).
· fail_on_revert : Jika true, tes akan gagal begitu ada transaksi yang revert. Jika false, tes mengabaikan revert dan tetap melanjutkan untuk memeriksa invariant.

Saat pertama kali dijalankan dengan fail_on_revert = true, Foundry melaporkan revert karena arithmetic underflow/overflow, bukan karena invariant rusak. Ini mengganggu fokus pada invariant yang sebenarnya.

Menyetel fail_on_revert = false

```toml
[invariant]
runs = 64
depth = 32
fail_on_revert = false
```

Dengan pengaturan ini, Foundry mengabaikan revert dan tetap melanjutkan fuzzing. Sekarang ia akan menemukan bahwa:

1. Fuzzer memanggil changeValue(0) → myValue = 0.
2. Kemudian memanggil doMoreMathAgain(0) → perhitungan 0 + 0 = 0 → fungsi mengembalikan 0.
3. storedValue menjadi 0 → invariant storedValue != 0 rusak.

where-stateless-fuzzing-fails3.png

---

Kesimpulan

· Stateful fuzzing diperlukan untuk menemukan kerentanan yang muncul dari interaksi berurutan antar fungsi (perubahan state kontrak).
· Implementasi di Foundry melibatkan StdInvariant, targetContract(), dan fungsi tes berawalan statefulFuzz_ atau invariant_.
· Konfigurasi [invariant] seperti runs, depth, dan fail_on_revert sangat mempengaruhi efektivitas pengujian.
· Dengan fail_on_revert = false, kita dapat fokus memeriksa invariant meskipun ada transaksi yang revert di tengah jalan.

Stateful fuzzing adalah alat yang sangat kuat untuk menguji invariant dalam protokol DeFi yang kompleks.