## Apa Itu ERC20 Token?

**ERC20** adalah standar token yang berfungsi di blockchain Ethereum (dan jaringan kompatibel EVM lainnya). Standar ini mendefinisikan sekumpulan aturan (fungsi wajib) yang harus dimiliki sebuah token agar dapat berinteraksi secara mulus dengan token lain dan kontrak pintar di ekosistem.

- ERC20 bersifat **fungible** (dapat dipertukarkan satu sama lain dengan nilai yang sama).
- ERC20 juga merupakan **smart contract** – sehingga dapat memiliki logika bisnis kompleks di dalamnya.

### Contoh Token ERC20 Terkenal

| Token | Catatan |
|-------|---------|
| Tether (USDT) | Stablecoin |
| Chainlink (LINK) | Sebenarnya ERC-677 (kompatibel dengan ERC20, dengan fungsi tambahan) |
| Uniswap (UNI) | Token tata kelola |
| DAI | Stablecoin desentralisasi |

> **Catatan:** ERC-677 adalah standar yang lebih tinggi, tetap kompatibel dengan ERC20 tetapi menambahkan fungsi seperti `transferAndCall`.

## Mengapa Peduli dengan ERC20 Token?

ERC20 memiliki banyak kegunaan penting dalam ekosistem Web3:

| Penggunaan | Penjelasan |
|------------|------------|
| **Governance token** | Memungkinkan pemegang token untuk memilih dalam DApp (tata kelola terdesentralisasi). |
| **Mengamankan jaringan** | Digunakan sebagai insentif dalam mekanisme konsensus (misal staking). |
| **Mewakili aset statis** | Seperti saham, komoditas, atau mata uang fiat yang di-tokenisasi. |
| **Utility token** | Memberikan akses ke layanan atau fitur tertentu dalam platform. |
| **Mata uang aplikasi** | Pembayaran dalam ekosistem DApp. |

## Cara Membuat Token ERC20

Untuk membuat token ERC20, Anda perlu menulis smart contract yang **mematuhi standar ERC20**. Fungsi minimal yang harus ada:

### Fungsi Wajib (Core Functions)

| Fungsi | Deskripsi |
|--------|------------|
| `totalSupply()` | Jumlah total token yang beredar. |
| `balanceOf(address account)` | Saldo token milik suatu alamat. |
| `transfer(address recipient, uint256 amount)` | Mengirim token dari pengirim ke penerima. |
| `approve(address spender, uint256 amount)` | Mengizinkan `spender` menarik token dari pengirim. |
| `transferFrom(address sender, address recipient, uint256 amount)` | Pindah token atas nama pengirim (setelah approval). |
| `allowance(address owner, address spender)` | Cek jumlah token yang masih diizinkan untuk ditarik `spender`. |

### Fungsi Opsional (Informasi)

- `name()` – Nama token (misal "MyToken")
- `symbol()` – Simbol token (misal "MTK")
- `decimals()` – Jumlah desimal (biasanya 18)

### Contoh Kode Sederhana (Menggunakan OpenZeppelin)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract MyToken is ERC20 {
    constructor() ERC20("MyTokenName", "MTN") {
        _mint(msg.sender, 1_000_000 * 10 ** decimals());
    }
}
```

## Standar Token Lain yang Perlu Diketahui

| Standar | Deskripsi | Kompatibilitas dengan ERC20 |
|---------|-----------|------------------------------|
| **ERC-677** | Menambahkan fungsi `transferAndCall` untuk notifikasi penerima. | ✅ Kompatibel penuh |
| **ERC-777** | Standar lebih canggih dengan hook `tokensReceived` dan `tokensToSend`. | ✅ Kompatibel penuh |
| **ERC-20** | Standar dasar. | — |
| **ERC-721 (NFT)** | Token non-fungible, setiap token unik. | ❌ Tidak kompatibel |
| **ERC-1155** | Multi-token standard (fungible + non-fungible dalam satu kontrak). | ❌ Tidak kompatibel langsung |

## Kesimpulan

- ERC20 adalah fondasi penting di ekosistem Ethereum.
- Membuat token sendiri relatif mudah dengan library seperti OpenZeppelin.
- Pahami fungsi wajib dan opsional sebelum mengimplementasikan.
- Pelajari juga standar turunan (ERC-677, ERC-777) untuk kebutuhan lebih lanjut.
