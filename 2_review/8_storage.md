## 1. Storage: Penyimpanan Persisten

**Storage** adalah area penyimpanan permanen di smart contract. Setiap variabel state (variabel global kontrak) disimpan di storage.

- Storage seperti **array besar** berukuran 32 byte per slot.
- Indeks slot dimulai dari 0, 1, 2, dst.
- Nilai yang disimpan dalam format bytes (contoh: angka 25 disimpan sebagai `0x19`).

### Contoh

```solidity
contract FundamentalStorage {
    uint256 favoriteNumber = 25;  // slot 0
    bool ourBool = true;           // slot 1
}
```

> Semua variabel state secara default disimpan di storage.

---

## 2. Variabel Dinamis (Dynamic Arrays & Mapping)

Variabel dinamis seperti **array dinamis** (`uint[]`) dan **mapping** tidak menyimpan seluruh datanya di satu slot.

### Cara Kerja Array Dinamis

- Slot yang ditempati array menyimpan **panjang (length)** array.
- Elemen-elemen array disimpan di slot yang dihitung dengan fungsi hash:
  - Posisi elemen ke-`i` = `keccak256(slot_array) + i`

### Contoh

```solidity
contract Example {
    uint[] myArray;  // slot 0 menyimpan panjang array

    function addToArray(uint _number) public {
        myArray.push(_number);  // elemen disimpan di slot keccak256(0)
    }
}
```

| Data | Lokasi Storage |
|------|----------------|
| `myArray.length` | Slot `0` |
| `myArray[0]` | Slot `keccak256(0)` |
| `myArray[1]` | Slot `keccak256(0) + 1` |

### Mapping

- Tidak ada panjang yang disimpan.
- Nilai untuk key `k` berada di slot `keccak256(k . slot_mapping)`

---

## 3. Constant dan Immutable (Tidak Pakai Storage)

Variabel `constant` dan `immutable` **tidak menempati slot storage**. Nilainya disimpan langsung dalam **bytecode** kontrak.

| Keyword | Kapan ditentukan | Contoh |
|---------|------------------|--------|
| `constant` | Saat kompilasi | `uint256 constant x = 123;` |
| `immutable` | Di constructor | `uint256 immutable y;` |

### Keuntungan
- Lebih hemat gas (tidak perlu operasi baca/tulis storage).
- Nilai langsung disubstitusi oleh compiler.

---

## 4. Variabel Sementara di Function (Memory)

Variabel yang dideklarasikan di dalam **function** bersifat **sementara** dan tidak disimpan di storage. Mereka disimpan di area **memory** yang akan dihapus setelah function selesai.

### Contoh

```solidity
function myFunction(uint256 val) public pure {
    uint256 newVar = val + 5;   // hanya hidup di dalam function
}
```

- `newVar` tidak persisten.
- Setiap panggilan function akan membuat `newVar` baru.

---

## 5. Keyword `memory` untuk String dan Array Dinamis

Solidity memerlukan keyword `memory` untuk parameter atau variabel lokal yang bertipe **string**, **array dinamis**, atau **struct** karena mereka membutuhkan alokasi ruang yang jelas.

### Contoh dengan string

```solidity
function getString() public pure returns (string memory) {
    return "this is a string!";
}
```

- `string` adalah array dinamis (tidak tahu ukuran tetap).
- `memory` memberi tahu Solidity bahwa data ini bersifat sementara, bukan di storage.

> **Catatan:** Ada juga keyword `calldata` untuk parameter function (read-only, tidak dimodifikasi).

---

## Ringkasan Perbandingan

| Lokasi Data | Karakteristik | Contoh Penggunaan |
|-------------|---------------|-------------------|
| **Storage** | Permanen, disimpan di blockchain, biaya gas tinggi | Variabel state kontrak |
| **Memory** | Sementara, dihapus setelah function selesai, lebih murah | Variabel lokal, parameter string/array |
| **Calldata** | Sementara, read-only, paling murah | Parameter function eksternal |
| **Bytecode** | Tidak terpisah, disimpan dalam kode kontrak | `constant`, `immutable` |

## Sumber Belajar

Dokumentasi resmi Solidity: [docs.soliditylang.org](https://docs.soliditylang.org)

> *"Understanding the nitty-gritty of Solidity variables and storage will significantly amplify your solidity coding skills."*
