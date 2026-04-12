# Ringkasan Materi Prasyarat dan Pengujian dengan Foundry

## Sumber Belajar Lengkap
Materi prasyarat dan lanjutan dapat diakses di:
- [Foundry Full Course (Fundamentals)](https://updraft.cyfrin.io/courses/foundry)
- [Foundry Full Course (Advanced)](https://updraft.cyfrin.io/courses/advanced-foundry)

---

## 1. Prasyarat Solidity (Solidity Basics)

Agar dapat mengikuti kursus ini, Anda harus sudah familiar dengan:
- Fungsi dasar [Remix](https://remix.ethereum.org):
  - **Compiling** (mengkompilasi kode)
  - **Deploying** ke **local blockchain** (misal Ganache) maupun **testnet** (seperti Sepolia, Goerli)
- Semua konsep dasar Solidity:
  - Tipe data (uint, int, bool, address, string, dll)
  - Struktur kontrak (contract, function, modifier, event)
  - Variabel (state, local, global)
  - Visibility (public, private, internal, external)

> **Catatan**: Pengetahuan ini harus sudah "second nature" (sangat dikuasai).

---

## 2. Keakraban dengan Foundry (Foundry Familiarity)

Anda juga harus familiar dengan lingkungan kerja Foundry (atau framework pilihan Anda). Pahami cara:
- **Inisialisasi proyek** baru
- **Menavigasi struktur direktori** proyek (working tree)

### Perintah Dasar Foundry

```bash
forge init    # membuat proyek Foundry baru
forge build   # mengkompilasi kontrak
forge test    # menjalankan pengujian
```

### Contoh Kontrak Sederhana (Counter.sol)

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

contract Counter {
    uint256 public number;

    function setNumber(uint256 newNumber) public {
        number = newNumber;
    }

    function increment() public {
        number++;
    }
}
```

Kode seperti di atas harus sudah dikenali dengan baik.

---

## 3. Pengujian (Testing) di Foundry

Contoh pengaturan pengujian Foundry biasanya berisi **dua jenis tes**:
1. **Regular test** (tes biasa)
2. **Fuzz test** (tes dengan input acak)

### a. Regular Test
- Membuat instance kontrak `Counter`
- Memanggil fungsi `increment()`
- Memastikan `number == 1` (assertion)

### b. Fuzz Test
- Menerima parameter acak, misal `uint256 x`
- Menguji bahwa properti tertentu selalu benar **untuk berapa pun nilai x**
- Dijalankan dengan sejumlah **runs** (iterasi) menggunakan nilai acak berbeda

### Mengubah Jumlah Fuzz Runs

Buka file `foundry.toml` (atau `forge.toml`) dan atur bagian `[fuzz]`:

```toml
[fuzz]
runs = 256                    # jumlah iterasi (default 256)
max_test_rejects = 65536
seed = "0x3e8"
dictionary_weight = 40
include_storage = true
include_push_bytes = true
```

Contoh: ubah `runs` dari `256` menjadi `600`.

Kemudian jalankan:

```bash
forge test
```

Output akan menunjukkan bahwa fuzz test berjalan 600 kali:

```bash
Running 1 test for test/Counter.t.sol:CounterTest
[PASS] testFuzz_SetNumber(uint256) (runs: 600, μ: 27398, ~: 28409)
Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 14.63ms

Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)
```

> **Penjelasan**: `runs: 600` berarti tes dijalankan dengan 600 angka acak berbeda, dan semuanya lolos.

---

## 4. Fuzzing Lanjutan: Stateful Fuzzing dan Invariant Tests

- **Stateful fuzzing** (dikenal juga sebagai **invariant test** di ekosistem Foundry)
- Menguji bahwa **kondisi tertentu (invariant)** selalu terpenuhi **setelah serangkaian aksi acak** terhadap kontrak.
- Contoh invariant: "total supply token tidak boleh melebihi batas maksimum" atau "saldo pengguna tidak boleh negatif".
- Foundry akan secara acak memanggil fungsi-fungsi kontrak dan memeriksa apakah invariant masih benar.

> **Catatan**: Topik ini mungkin belum Anda kuasai, tetapi akan dijelaskan lebih mendalam di pelajaran berikutnya.

---

## Kesimpulan

- Pastikan Anda menguasai **Solidity dasar** dan **operasi Remix**.
- Kuasai perintah dasar Foundry: `forge init`, `forge build`, `forge test`.
- Pahami perbedaan **regular test** dan **fuzz test**.
- Pelajari cara mengatur **jumlah fuzz runs** di `foundry.toml`.
- Selanjutnya akan dipelajari **stateful fuzzing / invariant tests**.
