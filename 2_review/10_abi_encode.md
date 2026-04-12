## Pendahuluan

Encoding di Solidity memungkinkan kita mengubah data (string, angka, array, dll) menjadi format **binary** yang dapat diproses oleh EVM. Dua fungsi utama: `abi.encode` dan `abi.encodePacked`. Juga disinggung `string.concat()` mulai Solidity 0.8.12.

> Semua kode contoh tersedia di [repo ini](https://github.com/PatrickAlphaC/hardhat-nft-fcc/tree/main/contracts/sublesson).

---

## Contoh Dasar: Menggabungkan String

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.7;

contract Encoding {
    function combineStrings() public pure returns (string memory) {
        return string(abi.encodePacked("Hi Mom! ", "Miss you."));
    }
}
```

- `abi.encodePacked("Hi Mom! ", "Miss you.")` mengubah kedua string ke binary, menggabungkannya, lalu di-typecast ke string.
- Hasil: `"Hi Mom! Miss you."`

> Alternatif modern: `string.concat("Hi Mom! ", "Miss you.")` (Solidity 0.8.12+)

---

## Kompilasi Smart Contract

Ketika kita kompilasi, compiler menghasilkan dua hal penting:

| Output | Deskripsi |
|--------|-----------|
| `contract.abi` | Antarmuka (interface) untuk berinteraksi dengan kontrak. |
| `contract.bin` | **Binary representation** dari kode kontrak (yang benar-benar disimpan di blockchain). |

### Struktur Transaksi

Transaksi yang dikirim ke blockchain memiliki field `data` yang berisi binary:

```js
tx = {
  nonce: nonce,
  gasPrice: 10000000000,
  gasLimit: 1000000,
  to: null,           // null jika deployment
  value: 0,
  data: "BINARYGOESHERE",  // <-- di sinilah binary contract disimpan
  chainId: 1337,
};
```

- Saat **deploy** kontrak baru: `to` kosong, `data` berisi init code + bytecode kontrak.
- Saat **memanggil fungsi** kontrak yang sudah ada: `to` berisi alamat kontrak, `data` berisi encoded function signature + parameter.

Contoh transaksi dengan binary data: [Etherscan tx](https://etherscan.io/tx/0x112133a0a74af775234c077c397c8b75850ceb61840b33b23ae06b753da40490)

---

## EVM Opcodes

Binary data yang terlihat acak sebenarnya adalah rangkaian **opcode**. Setiap 2 karakter (1 byte) mewakili satu instruksi untuk EVM.

- Opcode adalah "alfabet" bahasa mesin Ethereum.
- Contoh: `0x60` (PUSH1), `0x01` (ADD), `0x00` (STOP).
- Daftar lengkap: [evm.codes](https://www.evm.codes/?fork=shanghai)

Ketika kita mengirim transaksi, `data` field adalah kumpulan opcode yang akan dieksekusi EVM.

---

## `abi.encode` vs `abi.encodePacked`

| Fungsi | Karakteristik |
|--------|----------------|
| `abi.encode` | Hasil binary **dipadding** menjadi kelipatan 32 byte (standar ABI). Setiap argumen dipisahkan dengan jelas. |
| `abi.encodePacked` | Mode **non-standard packed**: tipe <32 byte digabung langsung tanpa padding, dynamic type dikodekan tanpa length, array elements dipadding tetapi tetap in-place. |

### Kapan menggunakan?

- `abi.encode` → untuk interoperabilitas standar (misal: memanggil fungsi dari kontrak lain, sesuai ABI spec).
- `abi.encodePacked` → lebih hemat gas, tetapi **tidak aman untuk hash signature** karena rentan collision (jangan gunakan untuk `keccak256(abi.encodePacked(...))` jika ada input variabel panjang).

> Baca lebih lanjut: [Non-standard Packed Mode](https://docs.soliditylang.org/en/latest/abi-spec.html#abi-packed-mode)

---

## Decoding: `abi.decode`

Selain encode, Solidity juga menyediakan `abi.decode` untuk mengembalikan binary ke tipe aslinya.

Contoh:

```solidity
bytes memory data = abi.encode(123, "hello");
(uint256 num, string memory text) = abi.decode(data, (uint256, string));
```

Juga bisa melakukan **multiEncode** dan **multiDecode** (encode beberapa nilai sekaligus, lalu decode sesuai urutan).

---

## Kesimpulan

- `abi.encode` dan `abi.encodePacked` adalah alat penting untuk manipulasi data binary di Solidity.
- Encoding digunakan dalam transaksi blockchain untuk mewakili fungsi dan parameter.
- EVM mengeksekusi opcode yang merupakan hasil decoding dari binary `data`.
- Pahami perbedaan kedua fungsi untuk menghindari bug (terutama pada signature hash).
