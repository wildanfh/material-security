# Stateful dan Stateless Fuzzing untuk Menguji Invariant

## Pendahuluan

Menulis kontrak dan menjalankan tes saja tidak cukup untuk menjamin keamanan. Sering kali eksploitasi muncul dari skenario yang tidak terduga.

**Fuzz Testing** adalah metode membombardir protokol dengan data acak untuk mencoba "mematahkannya". Ada dua bentuk:

- **Stateless Fuzzing** – Setiap run dimulai dengan instance protokol baru (state direset). Fungsi dipanggil dengan data acak → mulai lagi → fungsi dipanggil dengan data acak → mulai lagi.
- **Stateful Fuzzing** – Setiap run mengingat state dari run sebelumnya. Fungsi dipanggil dengan data acak → fungsi lain dipanggil dengan data acak (state berlanjut).

---

## Contoh Sederhana: Invariant yang Harus Selalu Benar

```solidity
contract MyContract {
    uint256 public shouldAlwaysBeZero = 0;

    function doStuff(uint256 data) public {
        if(data == 2) {
            shouldAlwaysBeZero = 1;  // Invariant rusak jika data == 2
        }
    }
}
```

Invariant: shouldAlwaysBeZero harus selalu 0.

Unit Test Biasa (Tidak Menyeluruh)

```solidity
function testIAlwaysGetZero() public {
    uint256 data = 0;
    exampleContract.doStuff(data);
    assert(exampleContract.shouldAlwaysBeZero() == 0);
}
```

Tes ini hanya menguji satu skenario (data = 0). Tidak akan menangkap kasus data = 2.

---

Stateless Fuzzing dengan Foundry

Cukup dengan menjadikan data sebagai parameter fungsi test, Foundry akan secara otomatis memasukkan data acak.

```solidity
function testIAlwaysGetZero(uint256 data) public {
    exampleContract.doStuff(data);
    assert(exampleContract.shouldAlwaysBeZero() == 0);
}
```

Ketika dijalankan, Foundry akan menemukan bahwa data = 2 menyebabkan invariant rusak.

Catatan: Data yang dimasukkan sebenarnya adalah semi-random, cara fuzzer memilih data itu penting (topik lanjutan).

Mengatur Jumlah Runs

Di foundry.toml:

```toml
[fuzz]
runs = 1000
```

Semakin banyak runs, semakin lama eksekusi, tetapi cakupan pengujian lebih luas.

---

Keterbatasan Stateless Fuzzing

Perhatikan kontrak yang dimodifikasi:

```solidity
contract MyContract {
    uint256 public shouldAlwaysBeZero = 0;
    uint256 private hiddenValue = 0;

    function doStuff(uint256 data) public {
        if (hiddenValue == 7) {
            shouldAlwaysBeZero = 1;   // Invariant rusak jika hiddenValue == 7
        }
        hiddenValue = data;
    }
}
```

· Stateless fuzzing tidak akan menangkap kerentanan ini karena setiap run dimulai dengan hiddenValue = 0.
· Kerentanan hanya muncul jika doStuff dipanggil dua kali berturut-turut: pertama untuk mengatur hiddenValue = 7, kedua untuk memicu if (hiddenValue == 7).

Di sinilah stateful fuzzing diperlukan.

---

Stateful Fuzzing (Invariant Test)

Stateful fuzzing mempertahankan perubahan state antar run. Dalam Foundry, ini disebut invariant test.

Langkah-langkah:

1. Import dan inherit StdInvariant

```solidity
import {StdInvariant} from "forge-std/StdInvariant.sol";

contract MyContractTest is StdInvariant, Test {
    // ...
}
```

1. Tentukan target contract di fungsi setUp

```solidity
function setUp() public {
    exampleContract = new MyContract();
    targetContract(address(exampleContract));
}
```

1. Tulis invariant test (harus diawali keyword invariant)

```solidity
function invariant_testAlwaysReturnsZero() public {
    assert(exampleContract.shouldAlwaysBeZero() == 0);
}
```

Hasil

Foundry akan memanggil fungsi acak pada target contract secara berulang, mempertahankan state antar panggilan. Dalam contoh di atas, fuzzer dapat memanggil doStuff pertama kali dengan data = 7 (mengatur hiddenValue = 7), lalu memanggil lagi dengan data apapun, sehingga invariant rusak.

---

Ringkasan Perbandingan

Aspek Stateless Fuzzing Stateful Fuzzing (Invariant Test)
State awal Reset setiap run Berlanjut dari run sebelumnya
Kompleksitas setup Minimal (cukup parameter fungsi) Perlu StdInvariant dan targetContract
Kemampuan menemukan bug Bug yang bergantung pada satu panggilan Bug yang memerlukan urutan panggilan tertentu
Keyword test Nama fungsi bebas Harus diawali invariant atau fuzz

Kesimpulan

· Stateless fuzzing cocok untuk menguji fungsi individual dengan input acak.
· Stateful fuzzing (invariant test) diperlukan untuk menemukan kerentanan yang muncul akibat interaksi berantai antar fungsi.
· Keduanya adalah alat yang sangat kuat dalam security review, terutama untuk menguji invariant protokol.