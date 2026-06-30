# Ringkasan Materi: Exploit – Failure to Initialize (Contoh di Remix)

## Pendahuluan

Setelah memahami konsep *failure to initialize* secara teoretis, kita sekarang melihat contoh konkret di **Remix** untuk mendemonstrasikan bagaimana kerentanan ini dapat dieksploitasi. Contoh ini menggunakan kontrak sederhana dari repositori `sc-exploits-minimized`.

> **Link Remix:** [FailureToInitialize.sol – sc-exploits-minimized](https://remix.ethereum.org/#url=https://github.com/Cyfrin/sc-exploits-minimized/blob/main/src/failure-to-initialize/FailureToInitialize.sol&lang=en&optimize=false&runs=200&evmVersion=null&version=soljson-v0.8.20+commit.a1b79de6.js)

---

## Kontrak Rentan: `FailureToInitialize.sol`

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

import "@openzeppelin/contracts/proxy/utils/Initializable.sol";

contract FailureToInitialize is Initializable {
    uint256 public myValue;
    bool public initialized;

    function initialize(uint256 _startingValue) public initializer {
        myValue = _startingValue;
        initialized = true;
    }

    // Seharusnya ada pemeriksaan di sini untuk memastikan kontrak sudah diinisialisasi!
    function increment() public {
        myValue++;
    }
}
```

### Komponen Penting

| Elemen | Fungsi |
|--------|--------|
| `Initializable` | Library OpenZeppelin untuk kontrak upgradeable – mencegah inisialisasi ulang melalui modifier `initializer`. |
| `initialize(uint256 _startingValue)` | Fungsi inisialisasi yang seharusnya dipanggil **sekali** setelah deployment. |
| `increment()` | Fungsi yang mengubah state `myValue` **tanpa memeriksa apakah kontrak sudah diinisialisasi**. |
| `myValue` | State variable yang seharusnya diatur di awal oleh `initialize`. |
| `initialized` | Boolean yang menandakan apakah inisialisasi sudah dilakukan. |

---

## Kerentanan

1. Kontrak dideploy tanpa memanggil `initialize` terlebih dahulu.
2. Pengguna mulai menggunakan kontrak (memanggil `increment`) – `myValue` akan bertambah dari 0.
3. **Penyerang** kemudian memanggil `initialize` dengan nilai `_startingValue` yang mereka inginkan.
4. Karena `initializer` modifier hanya mencegah inisialisasi **ulang** (bukan mencegah panggilan setelah kontrak digunakan), panggilan ini berhasil.
5. `myValue` di-overwrite – nilai yang telah diubah oleh pengguna sebelumnya hilang.
6. Kontrak sekarang dalam state yang **rusak** – nilai `myValue` tidak sesuai dengan harapan.

> **Catatan:** Modifier `initializer` memastikan `initialize` hanya bisa dipanggil **sekali**. Namun, ia **tidak mencegah** pemanggilan `initialize` setelah kontrak sudah digunakan – selama belum dipanggil sebelumnya, siapa pun dapat memanggilnya kapan saja.

---

## Demonstrasi di Remix

### Langkah 1: Deploy Kontrak

![failure-to-initialize-remix1](/security-section-6/21-failure-to-initialize-remix/failure-to-initialize-remix1.png)

- Kontrak dideploy dengan `myValue` default = 0.
- Fungsi `initialize` belum dipanggil.

### Langkah 2: Gunakan Kontrak (Tanpa Inisialisasi)

- Pengguna memanggil `increment()` beberapa kali.
- `myValue` menjadi, misal, 5 (bertambah dari 0).

### Langkah 3: Penyerang Memanggil `initialize`

![failure-to-initialize-remix2](/security-section-6/21-failure-to-initialize-remix/failure-to-initialize-remix2.png)

- Penyerang memanggil `initialize(100)`.
- `myValue` di-overwrite menjadi 100, dan `initialized = true`.

### Langkah 4: Kontrak Rusak

![failure-to-initialize-remix3](/security-section-6/21-failure-to-initialize-remix/failure-to-initialize-remix3.png)

- Sekarang `initialize` tidak bisa dipanggil lagi (karena `initializer` modifier).
- Nilai `myValue` yang benar (yang seharusnya diatur oleh protokol) sudah tidak bisa diperbaiki.
- Protokol dalam keadaan **tidak dapat dipulihkan** (broken) – setidaknya untuk state variable ini.

---

## Dampak dalam Protokol Nyata

Bayangkan skenario ini terjadi pada:

- **Oracle harga** – penyerang dapat mengatur harga yang salah.
- **Fee** – penyerang dapat mengatur biaya transaksi yang menguntungkan mereka.
- **Admin address** – penyerang dapat menjadi owner protokol.
- **Token pendukung** – penyerang dapat menambahkan token jahat.

> *"You could imagine a situation like this impacting the management of something very important – like billions of dollars."*

---

## Mitigasi

### 1. Panggil `initialize` Segera Setelah Deployment

Sertakan pemanggilan `initialize` dalam **script deployment** yang sama dengan deployment kontrak.

```solidity
// Deploy script
FailureToInitialize contract = new FailureToInitialize();
contract.initialize(100); // ← langsung diinisialisasi
```

### 2. Tambahkan Pemeriksaan `initialized` di Fungsi Kritis

```solidity
function increment() public {
    require(initialized, "Contract not initialized");
    myValue++;
}
```

### 3. Gunakan Modifier `onlyInitialized` untuk Fungsi-Fungsi Penting

```solidity
modifier onlyInitialized() {
    require(initialized, "Not initialized");
    _;
}

function increment() public onlyInitialized {
    myValue++;
}
```

---

## Studi Kasus Nyata

- **Compound v2** – kerentanan initializer yang dapat di-front-run.
- **Harvest Finance** – serangan serupa pada tahap awal protokol.
- **Oasis** – meskipun tidak persis sama, kasus ini menunjukkan dampak dari sentralisasi dan kurangnya inisialisasi yang aman.

---

## Kesimpulan

- **Failure to initialize** adalah kerentanan yang mudah diabaikan tetapi dapat berdampak sangat serius.
- Contoh Remix menunjukkan bahwa kontrak dapat digunakan **sebelum diinisialisasi**, dan kemudian diinisialisasi oleh penyerang untuk merusak state.
- Mitigasi terbaik: **panggil initializer dalam deployment script** dan tambahkan **pemeriksaan inisialisasi** pada fungsi-fungsi kritis.