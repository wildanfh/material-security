## OpenZeppelin Contracts

[OpenZeppelin Contracts](https://github.com/OpenZeppelin/openzeppelin-contracts) adalah library populer yang menyediakan kontrak-kontrak yang sudah jadi (pre-written) untuk berbagai standar Ethereum seperti ERC20, ERC721, dan lain-lain. Library ini sangat membantu karena Anda tidak perlu menulis ulang kode standar yang sudah teruji keamanannya.

### Instalasi OpenZeppelin dengan Foundry

Gunakan perintah berikut di direktori proyek Foundry Anda:

```bash
forge install OpenZeppelin/openzeppelin-contracts
```

## Konfigurasi Remapping di `foundry.toml`

Agar compiler dapat menemukan library yang telah diinstal, Anda perlu menambahkan konfigurasi remapping ke file `foundry.toml`:

```toml
remappings = ['@openzeppelin/contracts=lib/openzeppelin-contracts/contracts']
```

Penjelasan:
- `@openzeppelin/contracts` adalah alias (remapping) yang akan digunakan di dalam kode.
- Path di sebelah kanan menunjuk ke lokasi sebenarnya library di folder `lib/`.

## Mengimpor dan Menggunakan ERC20

Setelah remapping selesai, Anda dapat mengimpor kontrak ERC20 dari OpenZeppelin dan membuat token sendiri:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

import {ERC20} from '@openzeppelin/contracts/token/ERC20/ERC20.sol';

contract MyToken is ERC20 {
    constructor() ERC20("MyTokenName", "MTN") {}
}
```

- Kontrak `MyToken` mewarisi semua fungsi standar ERC20 seperti `transfer`, `balanceOf`, `approve`, dll.
- Konstruktor memanggil konstruktor `ERC20` dengan parameter nama token (`"MyTokenName"`) dan simbol (`"MTN"`).
