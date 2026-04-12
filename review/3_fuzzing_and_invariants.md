## Apa Itu Fuzz Testing?

**Fuzz testing** (atau fuzzing) adalah teknik pengujian dengan menyediakan data acak ke dalam sistem untuk mencoba "merusak" atau menemukan kegagalan yang tidak terduga. Analoginya: kode Anda seperti balon yang tidak bisa pecah, lalu Anda melakukan berbagai hal acak (menusuk, menekan, menendang) dengan tujuan mencari celah.

---

## Prinsip Dasar: Menguji Invariant

**Invariant** adalah properti integral dari suatu sistem (fungsi, kontrak, atau program) yang harus selalu benar dalam kondisi apa pun.

### Contoh Fungsi dengan Invariant

```solidity
function doStuff(uint256 data) public {
    if (data == 2){
        shouldAlwaysBeZero = 1;
    }
    if(hiddenValue == 7) {
        shouldAlwaysBeZero = 1;
    }
    hiddenValue = data;
}
```

Invariant: `shouldAlwaysBeZero` harus selalu bernilai 0.

### Unit Test Biasa (Tidak Menyeluruh)

```solidity
function testIsAlwaysGetZero() public {
    uint256 data = 0;
    exampleContract.doStuff(data);
    assert(exampleContract.shouldAlwaysBeZero() == 0);
}
```

> **Kelemahan**: Test di atas hanya menguji satu skenario (data=0). Jika data=2, invariant akan rusak. Menulis test untuk setiap kemungkinan input tidak praktis.

---

## Solusi: Fuzz Test dan Invariant Test

Dua pendekatan untuk menguji skenario tepi (edge cases) secara massal:
1. **Fuzz testing / Invariant testing** (dibahas di sini)
2. **Symbolic execution** (topik lain)

### Stateless Fuzzing (Contoh di Foundry)

Foundry secara otomatis menyediakan data acak ke parameter test.

```solidity
function testIsAlwaysGetZeroFuzz(uint256 data) public {
    exampleContract.doStuff(data);
    assert(exampleContract.shouldAlwaysBeZero() == 0);
}
```

- Foundry akan mengacak `data` dari `0` hingga `uint256.max()` sebanyak yang dikonfigurasi di `foundry.toml`.
- Mekanisme ini **pseudo-random** dan tidak ekshaustif (tidak semua kemungkinan input diuji).

### Konfigurasi Jumlah Runs

Di file `foundry.toml`:

```toml
[fuzz]
runs = 256   # jumlah iterasi fuzz test
```

---

## Stateless vs Stateful Fuzzing

| Stateless Fuzzing | Stateful Fuzzing |
|-------------------|------------------|
| Setiap run dimulai dari state kontrak yang baru (reset). | State akhir dari run sebelumnya menjadi state awal run berikutnya. |
| Hanya data input yang diacak. | Data input **dan** urutan fungsi yang dipanggil diacak. |
| Cocok untuk fungsi independen. | Cocok untuk skenario interaksi berantai antar fungsi. |

### Contoh Stateful Fuzzing di Foundry (Invariant Test)

Stateful fuzzing disebut juga **invariant test** karena menguji invariant setelah serangkaian aksi acak.

#### Langkah-langkah:

1. **Import dan inherit `StdInvariant`**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {StdInvariant} from "forge-std/StdInvariant.sol";

contract MyContractTest is StdInvariant, Test {
    // ...
}
```

2. **Tentukan target contract di fungsi `setUp`**

```solidity
function setUp() public {
    exampleContract = new MyContract();
    targetContract(address(exampleContract));
}
```

3. **Tulis invariant test dengan keyword `invariant`**

```solidity
function invariant_testAlwaysReturnsZero() public {
    assert(exampleContract.shouldAlwaysBeZero() == 0);
}
```

- Foundry akan memanggil **fungsi acak** pada `targetContract` (dalam contoh ini `doStuff` berulang kali, jika ada fungsi lain akan dipanggil dalam urutan acak).
- Setiap fungsi dipanggil dengan **data acak**.
- Setelah setiap urutan aksi, invariant diperiksa.

---

## Ringkasan dan Manfaat

- **Fuzz testing** membantu menemukan skenario tak terduga yang tidak terpikirkan dalam unit test biasa.
- **Stateless fuzzing** → hanya data acak, state direset setiap run.
- **Stateful fuzzing (invariant test)** → data acak + urutan fungsi acak, state berlanjut antar panggilan.
- Foundry menyediakan alat bawaan untuk kedua jenis fuzzing.

> *"Fuzz testing adalah teknik yang belum diadopsi oleh beberapa protokol top, namun dapat membantu menemukan kerentanan severity tinggi di smart contracts."* — Alex Rohn, Co-founder Cyfrin.
