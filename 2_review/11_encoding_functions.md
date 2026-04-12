# Encoding Function Call dan Low-Level Call di Solidity

## Pendahuluan

Setelah memahami dasar `abi.encode` dan `abi.encodePacked`, sekarang kita lihat bagaimana encoding digunakan dalam **transaksi blockchain** dan bagaimana kita bisa memanggil fungsi secara langsung menggunakan **low-level call** (`call` dan `staticcall`).

---

## Transaksi dan Data Field

Setiap transaksi Ethereum (baik deployment maupun function call) memiliki field `data` yang berisi **binary/hex** instruksi untuk EVM.

| Jenis Transaksi | Isi Field `data` |
|----------------|------------------|
| Deploy kontrak | Binary code kontrak (bytecode) |
| Memanggil fungsi | Encoded function signature + parameter |

### Contoh Input Data di Etherscan

Ketika melihat transaksi di Etherscan, field `Input Data` berisi hexadecimal seperti:

```
Function: enterRaffle(...)
Method ID: 0x2cfcc539
```

**Method ID** (atau function signature) adalah 4 byte pertama dari hash keccak256 nama fungsi dan tipe parameternya.  
Contoh: `keccak256("enterRaffle()")` → 4 byte pertama `0x2cfcc539`.

Ini adalah cara EVM mengetahui fungsi mana yang harus dipanggil.

---

## Membuat Data Field Sendiri

Dengan pemahaman encoding, kita bisa **secara manual** mengisi field `data` transaksi dengan hex yang sesuai, tanpa harus menggunakan ABI atau interface.

### Mengapa perlu?
- Hanya tahu nama fungsi atau parameter, tanpa ABI lengkap.
- Ingin membuat **arbitrary call** (panggilan dinamis).
- Ingin mengirim transaksi dengan data yang tidak standar.

---

## Low-Level Call di Solidity

Solidity menyediakan dua keyword untuk memanggil fungsi lain secara langsung (tanpa ABI):

| Keyword | Deskripsi |
|---------|-----------|
| `call` | Untuk fungsi yang **mengubah state** (non-view/pure) dan bisa mengirim ETH. |
| `staticcall` | Untuk fungsi **view/pure** (tidak mengubah state), tidak bisa mengirim ETH. |

### Sintaks Dasar

```solidity
(bool success, bytes memory returnData) = address(target).call{value: amount}(data);
```

- `target`: alamat kontrak yang akan dipanggil.
- `value`: jumlah ETH yang dikirim (opsional).
- `data`: encoded function signature + parameter (hex/binary).

### Contoh Tanpa Memanggil Fungsi (Hanya Kirim ETH)

```solidity
function withdraw(address recentWinner) public {
    (bool success, ) = recentWinner.call{value: address(this).balance}("");
    require(success, "Transfer Failed");
}
```
- `data` kosong (`""`) → tidak ada fungsi yang dipanggil, hanya transfer ETH.

### Contoh Memanggil Fungsi dengan Data Manual

Misal kita punya fungsi `enterRaffle()` di kontrak `PuppyRaffle` dengan method ID `0x2cfcc539`.

```solidity
function enterRaffle(uint256 entryFee) public payable {
    PuppyRaffle puppyRaffle = new PuppyRaffle();
    (bool success, ) = puppyRaffle.call{value: entryFee}("0x2cfcc539");
    require(success, "Call failed");
}
```

- `entryFee` masuk ke `value`.
- Data field diisi dengan hex `0x2cfcc539` (method ID).
- EVM akan mengeksekusi `enterRaffle()` di kontrak `puppyRaffle`.

### Contoh dengan Parameter

Jika fungsi memiliki parameter, misal `enterRaffle(uint256)`, maka kita perlu encode parameter juga:

```solidity
bytes memory data = abi.encodeWithSignature("enterRaffle(uint256)", 123);
(bool success, ) = puppyRaffle.call(data);
```

Atau dengan method ID manual:

```solidity
bytes memory data = abi.encodePacked(bytes4(keccak256("enterRaffle(uint256)")), abi.encode(123));
```

---

## Keuntungan dan Risiko

| Keuntungan | Risiko |
|------------|--------|
| Fleksibilitas tinggi, bisa panggil fungsi tanpa ABI. | Rentan kesalahan jika salah encoding. |
| Memungkinkan arbitrary call (misal untuk proxy pattern). | `call` tidak memeriksa apakah fungsi benar-benar ada (bisa gagal diam-diam). |
| Bisa mengirim ETH sekaligus memanggil fungsi. | Harus selalu cek `success` dan handle return data. |

> **Peringatan:** `call` mengesampingkan pemeriksaan keamanan Solidity. Gunakan dengan hati-hati dan selalu validasi return value.

---

## Kesimpulan

- **ABI encoding** adalah jantung dari komunikasi EVM.
- Field `data` dalam transaksi berisi encoded function signature + parameter.
- Kita bisa memanggil fungsi secara **low-level** menggunakan `call` dan `staticcall` dengan memasukkan data manual.
- Ini membuka pintu untuk pola desain lanjutan seperti proxy, multicall, dan hook dinamis.

> *"The function of good programming is to do the thinking for you, to the extent possible, so that when you're using it, your mind is free to think."* — Joshua Bloch
