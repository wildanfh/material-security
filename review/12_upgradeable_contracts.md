# Upgradeable Smart Contracts (Proxy Pattern & `delegatecall`)

## Mengapa Upgradeable?

Smart contract secara default **immutable** (tidak bisa diubah setelah deploy). Ini masalah untuk:
- Memperbaiki bug
- Menambahkan fitur baru
- Update logic tanpa mengubah state

Solusi: **Upgradeable smart contracts** dengan pola **proxy**.

---

## Proxy Pattern: Memisahkan State dari Logic

Dua komponen utama:

| Komponen | Peran | Karakteristik |
|----------|------|---------------|
| **Proxy Contract** | Tempat penyimpanan **state** (data, saldo). Pengguna selalu berinteraksi dengan alamat ini. | Alamat permanen, hanya berisi logika forwarding. |
| **Implementation (Logic) Contract** | Berisi **business logic** (fungsi-fungsi). | Stateless, bisa diganti dengan versi baru. |

### Proses Upgrade
1. Deploy implementation contract baru.
2. Update proxy contract untuk menunjuk ke alamat implementation baru.
3. Pengguna tetap pakai alamat proxy yang sama → state tetap utuh, logic berubah.

---

## Teknik Inti: `delegatecall`

`delegatecall` adalah opcode EVM low-level yang memungkinkan kontrak A menjalankan kode dari kontrak B **dalam konteks storage A**.

### Cara Kerja
- Kode dari B dijalankan.
- Storage yang dibaca/ditulis adalah **milik A**.
- `msg.sender` dan `msg.value` tetap berasal dari panggilan asli ke A.

### Contoh Minimal

```solidity
contract B {
    uint256 public num;
    address public sender;
    uint256 public value;

    function setVars(uint256 _num) public payable {
        num = _num;
        sender = msg.sender;
        value = msg.value;
    }
}

contract A {
    uint256 public num;
    address public sender;
    uint256 public value;

    function setVars(address _contractB, uint256 _num) public payable {
        (bool success, ) = _contractB.delegatecall(
            abi.encodeWithSignature("setVars(uint256)", _num)
        );
        require(success, "delegatecall failed");
    }
}
```

- `A` memanggil `delegatecall` ke `B.setVars`.
- Yang berubah adalah storage milik `A` (`num`, `sender`, `value` di `A`).
- Storage `B` tidak tersentuh.

### Persyaratan Kritis

> **Storage layout harus sama persis** antara proxy dan implementation contract.

Jika tidak:
- Implementasi bisa menimpa slot storage proxy yang salah.
- Akibat: **state corruption**, bug parah.

---

## Membangun Proxy Minimal (EIP-1967)

Proxy modern menggunakan **slot storage khusus** untuk menyimpan alamat implementation, guna menghindari **storage collision** (bentrok dengan variabel state implementasi).

### EIP-1967 Implementation Slot

```
bytes32 private constant _IMPLEMENTATION_SLOT = 
    0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc;
```

- Nilai ini adalah `keccak256("eip1967.proxy.implementation") - 1`.
- Dijamin tidak bertabrakan dengan slot yang digunakan implementasi (yang biasanya mulai dari slot 0).

### Contoh Kode Proxy Minimal

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/proxy/Proxy.sol";

contract SmallProxy is Proxy {
    bytes32 private constant _IMPLEMENTATION_SLOT = 
        0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc;

    function setImplementation(address newImplementation) public {
        assembly {
            sstore(_IMPLEMENTATION_SLOT, newImplementation)
        }
    }

    function _implementation() internal view override returns (address implementationAddress) {
        assembly {
            implementationAddress := sload(_IMPLEMENTATION_SLOT)
        }
    }

    // Fungsi fallback dan _delegate diwarisi dari OpenZeppelin Proxy.sol
    // Semua panggilan ke proxy otomatis didelegasikan ke implementation
}
```

### Penjelasan Kode
- **Inline assembly** (`sstore`/`sload`) digunakan untuk membaca/menulis slot storage tertentu secara efisien.
- **Warisan `Proxy.sol`** dari OpenZeppelin menyediakan `fallback` yang otomatis memanggil `_delegate(_implementation())`.
- Setiap fungsi yang tidak ada di proxy akan di-forward ke implementation.

---

## Implikasi untuk Developer dan Auditor

| Aspek | Perhatian |
|-------|-----------|
| **Storage layout** | Harus identik antara proxy dan semua versi implementation. Jangan mengubah urutan atau tipe variabel state. |
| **Storage collision** | Gunakan EIP-1967 untuk slot milik proxy. Jangan simpan variabel state di proxy selain di slot khusus. |
| **Access control** | Fungsi `setImplementation` harus dilindungi (hanya owner/DAO). Jika tidak, siapa pun bisa mengganti logic. |
| **Initialization** | Gunakan `initializer` modifier (bukan constructor) karena implementation dipanggil via delegatecall. |
| **Selfdestruct** | Hindari `selfdestruct` di implementation karena bisa menghancurkan proxy. |

### Risiko Umum yang Harus Diperiksa Auditor
- Storage collision antara proxy dan implementation.
- Fungsi upgrade tanpa akses kontrol.
- Implementation baru dengan storage layout berbeda.
- Fungsi `initialize` bisa dipanggil ulang (re-initialization attack).
- Penggunaan `delegatecall` ke kontrak yang tidak terpercaya.

---

## Kesimpulan

- **Proxy pattern + `delegatecall`** adalah standar industri untuk upgradeable contract.
- `delegatecall` menjalankan kode dari kontrak lain **dalam konteks storage pemanggil**.
- **Storage layout** harus identik antara proxy dan implementation.
- EIP-1967 menyediakan slot khusus untuk menyimpan alamat implementation, menghindari collision.
- Pemahaman mendalam tentang mekanisme ini **wajib** bagi auditor untuk menemukan celah kritis.

> *"For auditors, being very well versed in these mechanics is a prerequisite for a successful audit."*
