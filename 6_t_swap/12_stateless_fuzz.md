# Stateless Fuzzing – Praktik dan Penjelasan

## Apa Itu Stateless Fuzzing?

**Stateless fuzzing** (sering disebut hanya "fuzzing") adalah metode pengujian di mana kita memberikan **data acak** ke suatu fungsi untuk mencoba mematahkan *invariant* atau properti yang harus selalu benar.

Istilah "stateless" berarti **setelah setiap kali fuzzing berjalan (*run*), *state* kontrak direset** atau dimulai dari awal lagi. Setiap *run* independen, tidak ada hubungan dengan *run* sebelumnya.

## Contoh Kontrak yang Rentan

Kontrak `StatelessFuzzCatches.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

// INVARIANT: doMath should never return 0
contract StatelessFuzzCatches {
    /*
     * @dev Should never return 0
     */
    function doMath(uint128 myNumber) public pure returns (uint256) {
        if (myNumber == 2) {
            return 0;
        }
        return 1;
    }
}
```

**Invariant:** Fungsi `doMath` tidak boleh mengembalikan nilai 0.

**Masalah:** Jika input `myNumber == 2`, fungsi mengembalikan 0 → *invariant* rusak.

> Dalam basis kode yang cukup kompleks, celahnya tidak akan sejelas ini, tetapi konsep yang dipelajari tetap berlaku.

## Menulis Tes Stateless Fuzzing dengan Foundry

Buat folder `test/invariant-break/` dan file `StatelessFuzzCatchesTest.t.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

import {StatelessFuzzCatches} from "src/invariant-break/StatelessFuzzCatches.sol";
import {Test} from "forge-std/Test.sol";

contract StateLessFuzzCatchesTest is Test {
    StatelessFuzzCatches sfc;

    function setUp() public {
        sfc = new StatelessFuzzCatches();
    }

    function testFuzzCatchesStateless(uint128 randomNumber) public view {
        assert(sfc.doMath(randomNumber) != 0);
    }
}
```

**Penjelasan kode:**
- `import` kontrak yang akan diuji dan `Test.sol` dari Forge.
- Di `setUp`, buat instance baru kontrak yang akan diuji.
- Fungsi tes menerima parameter `uint128 randomNumber`. Foundry akan secara otomatis memasukkan nilai acak ke parameter ini (ini yang disebut *stateless fuzzing*).
- `assert` memastikan bahwa `doMath(randomNumber)` tidak pernah mengembalikan 0, sesuai dengan *invariant* yang diberikan.

## Konfigurasi Foundry (`foundry.toml`)

*Fuzz test* dapat dikonfigurasi di `foundry.toml`. Contoh:

```toml
[fuzz]
runs = 256
seed = '0x2'
```

- **`runs`** : Jumlah kali fungsi tes akan dipanggil dengan data acak (semakin banyak, semakin teliti tetapi semakin lambat).
- **`seed`** : Nilai awal untuk generator angka acak *pseudorandom*; mempengaruhi urutan "acak" yang dihasilkan.

## Menjalankan Tes

Perintah:

```bash
forge test --mt testFuzzCatchesStateless -vvvv
```

- `--mt` menjalankan fungsi tes tertentu (*match test*).
- `-vvvv` memberikan *log* detail.

**Hasil:** Foundry akan menemukan bahwa ketika `randomNumber == 2`, *invariant* rusak (karena fungsi mengembalikan 0) dan tes akan gagal. Ini menunjukkan betapa kuatnya *fuzzer* dalam menangkap kasus tepi (*edge case*).

## Kelebihan dan Kekurangan Stateless Fuzzing

### Kelebihan
- **Sangat kuat** – Dapat menangkap pelanggaran *invariant* yang tidak mungkin ditemukan secara manual dalam kode yang kompleks.
- **Otomatis** – Menghilangkan kebutuhan menulis banyak kasus uji spesifik.

### Kekurangan
- **Tidak dapat menangkap *invariant* yang rusak akibat urutan beberapa panggilan fungsi** (karena setiap *run* dimulai dari *state* baru).
- **Tidak ada jaminan 100%** – Input bersifat acak. Meskipun sudah diuji dengan 1 juta nilai acak, nilai ke-1.000.001 bisa saja menyebabkan kegagalan. *Fuzzing* memberikan kepercayaan tinggi, tetapi bukan bukti formal.

## Kesimpulan

*Stateless fuzzing* adalah teknik dasar yang sangat berguna untuk menguji fungsi individual dengan rentang masukan acak. Namun, ia memiliki batasan: tidak bisa mendeteksi kerentanan yang membutuhkan interaksi berantai antar fungsi. Untuk itu, diperlukan **stateful fuzzing** (*invariant testing*) yang akan dipelajari di tingkat berikutnya.